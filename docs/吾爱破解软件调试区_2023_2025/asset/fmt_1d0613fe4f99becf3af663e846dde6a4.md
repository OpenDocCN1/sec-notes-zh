
```
struct partition* cur_part;     //默认情况下操作的是哪个分区

/* 在分区链表中找到名为part_name的分区，并将其指针赋值给cur_part */
static bool mount_partition(struct list_elem* pelem, int arg){
  char* part_name = (char*)arg;
  struct partition* part = elem2entry(struct partition, part_tag, pelem);
  if(!strcmp(part_name, part->name)){
    cur_part = part;
    struct disk* hd = cur_part->my_disk;

    /* sb_buf用来存储从硬盘上读入的超级块 */
    struct super_block* sb_buf = (struct super_block*)sys_malloc(SECTOR_SIZE);

    /* 在内存中创建分区cur_part的超级块 */
    cur_part->sb = (struct super_block*)sys_malloc(sizeof(struct super_block));
    if(cur_part->sb == NULL){
      PANIC("alloc memory failed");
    }

    /* 读入超级块 */
    memset(sb_buf, 0, SECTOR_SIZE);
    ide_read(hd, cur_part->start_lba + 1, sb_buf, 1);

    /* 把缓冲区中的超级快复制到当前分区的sb中 */
    memcpy(cur_part->sb, sb_buf, sizeof(struct super_block));

    /**************** 将硬盘上的块位图读入到内存 ***************/
    cur_part->block_bitmap.bits = (uint8_t*)sys_malloc(sb_buf->block_bitmap_sects * SECTOR_SIZE);
    if(cur_part->block_bitmap.bits == NULL){
      PANIC("alloc memory failed!");
    }
    cur_part->block_bitmap.btmp_bytes_len = sb_buf->block_bitmap_sects * SECTOR_SIZE;
    /* 从硬盘上面读入块位图到分区的block_bitmap.bits*/
    ide_read(hd, sb_buf->block_bitmap_lba, cur_part->block_bitmap.bits, sb_buf->block_bitmap_sects);
    /***********************************************************/

    /**************** 将硬盘上的inode位图读入到内存 ***************/
    cur_part->inode_bitmap.bits = (uint8_t*)sys_malloc(sb_buf->inode_bitmap_sects * SECTOR_SIZE);
    if(cur_part->inode_bitmap.bits == NULL){
      PANIC("alloc memory failed!");
    }
    cur_part->inode_bitmap.btmp_bytes_len = sb_buf->inode_bitmap_sects * SECTOR_SIZE;
    /* 从硬盘上面读入块位图到分区的inode_bitmap.bits*/
    ide_read(hd, sb_buf->inode_bitmap_lba, cur_part->inode_bitmap.bits, sb_buf->inode_bitmap_sects);
    /***********************************************************/

    list_init(&cur_part->open_inodes);
    printk("mount %s done!\n", part->name);
    /* 此处返回true是为了迎合主调函数list_traversal的实现，与函数本身无关，只有返回true该函数才会停止遍历 */
    return true;
  }
  return false;         //使得list_traversal继续遍历
}
```

上述代码也就是在内存中创建一个超级块以及位图等，十分简单，接下来我们就到`filesys_init`函数当中注册默认装载分区是`sdb1`，然后来看看效果

```
  /* 确定默认操作的分区 */
  char default_part[8] = "sdb1";
  /* 挂载分区 */
  list_traversal(&partition_list, mount_partition, (int)default_part);
```

![](http://imgsrc.baidu.com/forum/pic/item/79f0f736afc37931dc5e6defaec4b74542a911c6.jpg)

可以看到也是十分正常的装载了`sdb1`

## 0x02 文件描述符

### 1. 文件描述符简介

我们的这个文件描述符与`inode`并不是同一个东西，`inode`是操作系统为自己的文件系统准备的数据结构，他用于文件存储的管理，与用户关系不大，而文件描述符才是与用户息息相关的。

咱们读写文件的时候，`inode`给咱们用户提供的信息仅仅是有哪些数据块，可是我们对文件的操作指针都是任意的，就拿读来说，我们读取文件都是通过多次读一小部分来进行的，在读的过程中我们必须维护一个文件偏移指针，所以为了保证下一次读的正确性，我们必须要时刻记录这个偏移指针的值，我们就将类似的倾向于用户的信息保存到文件描述符当中

在`linux`中，我们读写函数文件时都是操作文件描述符来完成的，而对于一个进程来说，我们打开了某个文件就是拥有了一个文件描述符，这个描述符对应了一个`inode`结点，然后通过`inode`节点来获取文件，每个进程中的`PCB`就保存着一个数组，用他来指向每个文件结构，也就是文件描述符，这里我给个图让大家方便理解：

![](http://imgsrc.baidu.com/forum/pic/item/8326cffc1e178a8266e29466b303738da877e81f.jpg)

我们来简单梳理一下找到文件的过程：

- 某进程把文件描述符作为参数提交给文件系统
- 文件系统用此文件描述符在该进程的`PCB`当中的文件描述符数组中索引对应的元素
- 从该元素当中获取文件的`inode`，最终找到`inode`的数据快

这里我们首先需要创建文件描述符，基本工作如下：

- 在全局的`inode`队列中新建一个`inode`，然后返回该`inode`的地址
- 在全局文件表中找到一个空位，在这里填上文件结构，使其中的`fd_inode`指向上一步中返回的`inode`地址，然后返回本文件在文件表中的下标地址
- 在`PCB`中的文件描述符数组中找到一个空位，使其值指向上一布中返回的文件结构下标，并返回本文件描述符在文件描述符数组中的下标值

### 2. 文件描述符实现

首先就是在我们的`PCB`当中添加进去元素，也就是修改`thread/thread.h`

```
int32_t fd_table[MAX_FILES_OPEN_PER_PROC];

这里是简单的在task_struct中添加了描述符数组，但是最大只支持8个。

注意这个数组也需要被初始化，所以我们在thread.c中记得修改
```

```
  /* 预留标准输入输出 */
  pthread->fd_table[0] = 0;
  pthread->fd_table[1] = 1;
  pthread->fd_table[2] = 2;
  /* 其余全置-1 */
  uint8_t fd_idx = 3;
  while(fd_idx fd_table[fd_idx] = -1;
    fd_idx++;
  }
  pthread->stack_magic = 0xdeadbeef;    //自定义魔数
```

我们在这里先预留了三个标准输入输出错误，我们等到讲到的时候会解释。到此我们的文件描述符算是创建完毕。

## 0x03 总结

今天的部分很少，就是简单的对于一些基本数据进行解释以及初始化，在下一节我们将会进行真正的对于我们需要的文件进行编程.

本次我的所有源码已在`github`上成功上传，分支名定为`FileSys_0`，欢迎各位指教

> 

[//github.com/PeiandHao/Simple_OS_For_Studing" target="_blank" rel="noopener noreferrer nofollow"](https://github.com/PeiandHao/Simple_OS_For_Studing:rel="nofollow noopener" href="https)
