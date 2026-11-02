
```
#include "fs.h"
#include "ide.h"
#include "stdint.h"
#include "global.h"
#include "super_block.h"
#include "inode.h"
#include "dir.h"
#include "string.h"
#include "memory.h"

/* 格式化分区，也就是初始化分区的元信息，创建文件系统 */
static void partition_format(struct disk* hd, struct partition* part){
  /* blocks_bitmap_init(为了方便实现，一个块就是一扇区) */
  uint32_t boot_sector_sects = 1;       //启动扇区数
  uint32_t super_block_sects = 1;       //超级块扇区数
  uint32_t inode_bitmap_sects = DIV_ROUND_UP(MAX_FILES_PER_PART, BITS_PER_SECTOR); //表示inode位图所占用的扇区数，这里表示占一个扇区就行了
  uint32_t inode_table_sects = DIV_ROUND_UP((sizeof(struct inode) * MAX_FILES_PER_PART), SECTOR_SIZE); //inode表所占扇区
  uint32_t used_sects = boot_sector_sects + super_block_sects + inode_bitmap_sects + inode_table_sects; //总共已经被使用的扇区
  uint32_t free_sects = part->sec_cnt - used_sects;                         //本分区剩余的扇区
  /*************************** 简单处理块位图占据的扇区数  *******************************/
  uint32_t block_bitmap_sects;
  block_bitmap_sects = DIV_ROUND_UP(free_sects, BITS_PER_SECTOR);   //获取空闲扇区位图所占的扇区数
  /* block_bitmap_bit_len是位图中位的长度，也是可用块的数量 */
  uint32_t block_bitmap_bit_len = free_sects - block_bitmap_sects;  //这里得出剩余空闲块扇区数
  block_bitmap_sects = DIV_ROUND_UP(block_bitmap_bit_len, BITS_PER_SECTOR);     //然后位图的扇区数就在对于位长度/每扇区的位数 向上取整来获得空闲位图扇区数
  /***************************************************************************************/

  /* 超级块初始化 */
  struct super_block sb;
  sb.magic = 0x20001109;
  sb.sec_cnt = part->sec_cnt;
  sb.inode_cnt = MAX_FILES_PER_PART;
  sb.part_lba_base = part->start_lba;
  sb.block_bitmap_lba = sb.part_lba_base + 2; //第0块是引导块，第1块是超级块，接下来就是这个空闲块位图了
  sb.block_bitmap_sects = block_bitmap_sects;

  sb.inode_bitmap_lba = sb.block_bitmap_lba + sb.block_bitmap_sects; //紧接着空闲块位图就存放咱们的i结点位图
  sb.inode_bitmap_sects = inode_bitmap_sects;

  sb.inode_table_lba = sb.inode_bitmap_lba + sb.inode_bitmap_sects; //然后就是我们的inode数组了
  sb.inode_table_sects = inode_table_sects;

  sb.data_start_lba = sb.inode_table_lba + sb.inode_table_sects;    //最后就是咱们的数据区了
  sb.root_inode_no = 0;
  sb.dir_entry_size = sizeof(struct dir_entry);

  printk("%s info:\n", part->name);
  printk("  magic:0x%x\n  part_lba_base:0x%x\n  all_sectors:0x%x\n  inode_cnt:0x%x\n  inode_bitmap_lba:0x%x\n\
        inode_bitmap_sectors:0x%x\n  inode_table_lba:0x%x\n  inode_table_sectors:0x%x\n  data_start_lba:0x%x\n",\
        sb.magic, sb.part_lba_base, sb.sec_cnt, sb.inode_cnt, sb.inode_bitmap_lba, sb.inode_bitmap_sects, sb.inode_table_lba, sb.inode_table_sects,\
        sb.data_start_lba);

  struct disk* hd = part->my_disk;
  /***************************************
   * 1.将超级块写入本分区的1扇区
   * *************************************/
  ide_write(hd, part->start_lba + 1, &sb, 1);
  printk("  super_block_lba:0x%x\n", part->start_lba + 1);

  /* 找出数据量最大的元信息，用其尺寸做存储缓冲区 */
  /* 这里是因为空闲块位图和inode数组都十分大，所以使用栈的话难以维系，所以需要申请堆来进行赋值 */
  uint32_t buf_size = (sb.block_bitmap_sects >= sb.inode_bitmap_sects ? sb.block_bitmap_sects : sb.inode_bitmap_sects);
  buf_size = (buf_size >= sb.inode_table_sects ? buf_size : sb.inode_table_sects) * SECTOR_SIZE;

  uint8_t* buf = (uint8_t*)sys_malloc(buf_size); //申请的内存由内存管理系统清0后返回,返回一个指向1字节地址的指针

  /******************************************
   * 2.将块位图初始化并且写入sb.block_bitmap_lba
   * ****************************************/
  /* 初始化块位图 */
  buf[0] |= 0x01;   //第0个块预留给根目录，位图中先占位
  uint32_t block_bitmap_last_byte = block_bitmap_bit_len / 8;
  uint8_t block_bitmap_last_bit = block_bitmap_bit_len % 8;
  uint32_t last_size = SECTOR_SIZE - (block_bitmap_last_byte % SECTOR_SIZE); //last_size是块位图最后一扇区中剩余没用的部分

  /* 先将位图最后一字节到其所在的扇区的结束全置为1，这里置1是为了防止错误访问 */
  memset(&buf[block_bitmap_last_byte], 0xff, last_size);

  /* 再将上一步中覆盖的最后一字节内的有效位置0 */
  uint8_t bit_idx = 0;
  while(bit_idx i_size = sb.dir_entry_size * 2;    //大小就是.和..两个目录项大小
  i->i_no = 0;      //这里是inode编号
  i->i_sectors[0] = sb.data_start_lba;  //由于上面的memset，i_sectors数组的其他元素都初始化为0
  ide_write(hd, sb.inode_table_lba, buf, sb.inode_table_sects);

  /******************************************
   * 5.将根目录写入sb.data_start_lba
   * ****************************************/
  /* 写入根目录的两个目录项.和.. */
  memset(buf, 0, buf_size);
  struct dir_entry* p_de = (struct dir_entry*)buf;
  /* 初始化当前目录“.” */
  memcpy(p_de->filename, ".", 1);
  p_de->i_no = 0;
  p_de->f_type = FT_DIRECTORY;
  p_de++;

  /* 初始化当前目录的父目录".." */
  memcpy(p_de->filename, "..", 1);
  p_de->i_no = 0;   //根目录的父目录还是其自己
  p_de->f_type = FT_DIRECTORY;

  /* sb.data_start_lba已经分配给了根目录，里面是根目录的目录项 */
  ide_write(hd, sb.data_start_lba, buf, 1);

  printk("  root_dir_lba:0x%x\n", sb.data_start_lba);
  printk("%s format done\n", part->name);
  sys_free(buf);
}
```

这个格式化的函数我在注释中已经很详细的说明了，所以这里也不过多讲，这里需要结合上面我给出的超级块的结构图进行理解，我们目前仅仅构造了根目录以及其中的两个目录项 `.` 和 `..`，大家熟练使用 Linux 的肯定都知道。

然后我们在其中再添加一个初始化函数就行了，在`fs/fs.c`当中：

```
/* 在磁盘上搜索文件系统，若没有则格式化分区创建文件系统 */

void filesys_init(){

  uint8_t channel_no = 0, dev_no, part_idx = 0;

  /* sb.buf 用来存储从硬盘上读入的超级块 */

  struct super_block* sb_buf = (struct super_block*)sys_malloc(SECTOR_SIZE);

  if(sb_buf == NULL){

    PANIC("alloc memory failed!");

  }

  printk("searching filesystem ...\n");

  while(channel_no prim_parts;

      while(part_idx logic_parts;

        }

        /* channels数组是全局变量，默认值为0,disk属于嵌套结构，

        * partition是disk的嵌套结构，所以partition中成员也默认为0

        * 下面处理存在的分区 */

        if(part->sec_cnt != 0){   //如果分区存在

          memset(sb_buf, 0, SECTOR_SIZE);

          /* 读出分区的超级块，根据魔数判断是否存在文件系统 */

          ide_read(hd, part->start_lba + 1, sb_buf, 1);   //这里start_lba + 1 是超级块所在的扇区

          if(sb_buf->magic == 0x20001109){

            printk("%s has filesystem\n", part->name);

          }else{

              printk("formatting %s's partition %s.......\n", hd->name, part->name);

              partition_format(part);

          }

        }

        part_idx++;

        part++;   //下一分区

      }

      dev_no++;   //下一磁盘

    }

    channel_no++;   //下一通道

  }

  sys_free(sb_buf);

}
```

上面代码所做的工作就是简单的搜索每个分区是否存在文件系统，如果不存在就进行初始化，下面我们第一次编译运行来看看情况。

![](http://imgsrc.baidu.com/forum/pic/item/fcfaaf51f3deb48ffc4c4918b51f3a292cf578cb.jpg)

可以看到咱们对于`hd80M.img`的各个分区已经进行了初始化，理论上我们对其进行初始化过后，硬盘上就应该永远存留着我们已经格式化好的文件系统了，我们再次运行试试看。

![](http://imgsrc.baidu.com/forum/pic/item/6c224f4a20a4462310d54da4dd22720e0df3d7dd.jpg)

发现确实没有再次进行初始化，因为初始化程序检测到每个分区已经存在了。

### 4> 3.挂载分区

咱们最初使用电脑的时候操作系统一般都是 Windows，他的分区十分简单的已经给咱们分出来了，那就是 C、D 等盘，但是 Linux 不一样，Linux 内核所在的分区是默认分区，从系统启动之后就以该分区为默认分区，如果要想使用其他分区的话需要使用`mount`命令手动将新分区挂载到默认分区的某个目录之下，这就使得我们的目录结构同树一样不断向下分支，当我们不想使用这个新分区的时候，就使用`umount`进行卸载。

但是目前的问题就是咱们的操作系统内核并不在文件系统上面，他处在一个`hd60M.img`的简单裸盘上面，我们根本没对其进行操作，更加谈不上分区，所以这里我们就采用一个简单的方法：

所谓的挂载，追求的就是我们能访问到磁盘上的文件，所以我们将文件系统分区的元信息加载到内存供咱们能访问就足够了，下面我们直接进行代码实现：
