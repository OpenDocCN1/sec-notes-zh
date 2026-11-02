
0014d040: 65 69 70 65 69 0A 68 65 6C 6C 6F 2C 20 70 65 69  eipei.hello, pei
0014d050: 70 65 69 0A 68 65 6C 6C 6F 2C 20 70 65 69 70 65  pei.hello, peipe
0014d060: 69 0A 68 65 6C 6C 6F 2C 20 70 65 69 70 65 69 0A  i.hello, peipei.
0014d070: 68 65 6C 6C 6F 2C 20 70 65 69 70 65 69 0A 68 65  hello, peipei.he
0014d080: 6C 6C 6F 2C 20 70 65 69 70 65 69 0A 68 65 6C 6C  llo, peipei.hell
0014d090: 6F 2C 20 70 65 69 70 65 69 0A 68 65 6C 6C 6F 2C  o, peipei.hello,
0014d0a0: 20 70 65 69 70 65 69 0A 68 65 6C 6C 6F 2C 20 70   peipei.hello, p
0014d0b0: 65 69 70 65 69 0A 00 00 00 00 00 00 00 00 00 00  eipei...........
0014d0c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
*
0014d1f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................

这里有这么多是因为我运行了很多次调试某个错误。

## 0x01 读取文件

紧接着我们来实现文件读取功能,同样仿照`file_write`的写法，其中仅仅是写和读的区别罢了

``` 
/* 从文件file当中读取count个字节写入buf,返回读出的字节数，若到文件尾则返回-1 */
int32_t file_read(struct file* file, void* buf, uint32_t count){
  uint8_t* buf_dst = (uint8_t*)buf;
  uint32_t size = count, size_left = size;
  /* 若要读取的字节数超过了文件可读取的剩余量，就使用剩余量作为待读取的字节数 */
  if((file->fd_pos + count) > file->fd_inode->i_size){
    size = file->fd_inode->i_size - file->fd_pos;
    size_left = size;
    if(size == 0){      //若为文件尾，则返回-1
      return -1;
    }
  }
  uint8_t* io_buf = sys_malloc(BLOCK_SIZE);
  if(io_buf == NULL){
    printk("file_read: sys_malloc for io_buf failed\n");
    return -1;
  }
  uint32_t* all_blocks = (uint32_t*)sys_malloc(BLOCK_SIZE + 48);    //同sys_write类似
  if(all_blocks == NULL){
    printk("file_read: sys_malloc for io_buf failed\n");
    return -1;
  }

  uint32_t block_read_start_idx = file->fd_pos / BLOCK_SIZE;    //数据快所在的起始扇区地址
  uint32_t block_read_end_idx = (file->fd_pos + size) / BLOCK_SIZE;     //数据所在块的终止扇区地址
  uint32_t read_blocks = block_read_end_idx - block_read_start_idx;     //如果增量为0,表示数据在同一扇区
  ASSERT(block_read_start_idx fd_inode->i_sectors[block_idx];
    }else{      //这说明是在一级间接块当中分配数据
      indirect_block_table = file->fd_inode->i_sectors[12];
      ide_read(cur_part->my_disk, indirect_block_table, all_blocks + 12, 1);
    }
  }else{        //若要读取多个块，下面会有三种情况
    /* 第一种情况：起始块和终止块属于直接块 */
    if(block_read_end_idx fd_inode->i_sectors[block_idx];
        block_idx++;
      }
    }else if(block_read_start_idx = 12){
        /* 第二种情况：起始块和终止快一个是直接块一个是间接块 */
        /* 我们先将直接块写入all_blocks */
        block_idx = block_read_start_idx;
        while(block_idx fd_inode->i_sectors[block_idx];
          block_idx++;
        }
        ASSERT(file->fd_inode->i_sectors[12] != 0);     //确保已经分配了一级间接块
        /* 这里咱们再将间接块写入all_blocks */
        indirect_block_table = file->fd_inode->i_sectors[12];
        ide_read(cur_part->my_disk, indirect_block_table, all_blocks + 12, 1);  //写入间接块的地址们
    }else{
      /* 第三种情况，数据均在间接块当中 */
      ASSERT(file->fd_inode->i_sectors[12] != 0);   //确保一级间接块已经分配
      indirect_block_table = file->fd_inode->i_sectors[12];
      ide_read(cur_part->my_disk, indirect_block_table, all_blocks + 12, 1);
    }
  }
  /* 用到的块地址已经全部收集到all_blocks当中，下面开始读取 */
  uint32_t sec_idx, sec_lba, sec_off_bytes, sec_left_bytes, chunk_size;
  uint32_t bytes_read = 0;
  while(bytes_read fd_pos / BLOCK_SIZE;
    sec_lba = all_blocks[sec_idx];
    sec_off_bytes = file->fd_pos % BLOCK_SIZE;
    sec_left_bytes = BLOCK_SIZE - sec_off_bytes;
    chunk_size = size_left my_disk, sec_lba, io_buf, 1);
    memcpy(buf_dst, io_buf + sec_off_bytes, chunk_size);
    buf_dst += chunk_size;
    file->fd_pos += chunk_size;
    bytes_read += chunk_size;
    size_left -= chunk_size;
  }
  sys_free(all_blocks);
  sys_free(io_buf);
  return bytes_read;
}
```

### 2. 实现 `sys_read`

这里我们来实现`sys_read`，来对`file_read`进行封装，我们一样还是在`fs.c`中完成,他非常简单

``` 
/* 从文件描述符fd指向的文件中读取count个字节到buf，若成功则返回读出的字节数，失败则返回-1 */
int32_t sys_read(int32_t fd, void* buf, uint32_t count){
  if(fd 
```

紧接着我们立刻到`main`函数当中测试

``` 
int main(void){
  put_str("I am Kernel\n");
  init_all();
  intr_enable();
  uint32_t fd = sys_open("/file1", O_CREAT);
  sys_close(fd);

  fd = sys_open("/file1", O_RDWR);
  printf("open /file1, fd:%d\n", fd);
  char buf[64] = {0};
  int read_bytes = sys_read(fd, buf, 18);
  printf("1_ read %d bytes:\n%s\n", read_bytes, buf);

  memset(buf, 0, 64);
  read_bytes = sys_read(fd, buf, 6);
  printf("2_ read %d bytes:\n%s\n", read_bytes, buf);

  memset(buf, 0, 64);
  read_bytes = sys_read(fd, buf, 6);
  printf("3_ read %d bytes:\n%s\n", read_bytes, buf);

  printf("______ close file1 and ropen ______ \n");
  sys_close(fd);
  fd = sys_open("/file1", O_RDWR);
  memset(buf, 0, 64);
  read_bytes = sys_read(fd, buf, 24);
  printf("4_ read %d bytes:\n%s\n", read_bytes, buf);
  sys_close(fd);
 while(1);//{
    //console_put_str("Main ");
  //};
  return 0;
}
```

直接运行查看结果

![](http://imgsrc.baidu.com/forum/pic/item/c75c10385343fbf245f9b4e4f57eca8064388f24.jpg)

可以发现十分成功！这里有个点注意那就是文件偏移指针关闭的时候会重置为0,所以第二次打开后会从头开始读取

## 0x02 实现文件读写指针定位功能

我们读写文件那肯定不能每次都从头读从尾写，所以我们迫切需要一种能控制文件偏移指针函数，这个函数原型就是`lseek`， 我们先来看看`lseek`的用法：

`lseek`原型是"`off_t lseek(int fd, off_t offset, int whence)`", `fd`是文件描述符，`offset`是便宜量，`whence`是`offset`的参照物，该函数的共能就是设置文件读写指针`fd_pos`为参照物+便宜量的值，我们的`whence`有以下三种取值：

- `SEEK_SET`，文件开始
- `SEEK_CUR`, 当前读写位置
- `SEEK_END`，文件尺寸大小，其实也就是文件最后一个字节的下一个字节，由于咱们的`fd_pos`始终指向下一个可读写的位置，它是以0为起始的便宜量，因此文件末尾是指文件大小

因此我们首先在`fs.h`中定义`whence`结构

``` 
/* 文件读写偏移量 */
enum whence{
  SEEK_SET = 1,
  SEEK_CUR,
  SEEK_END
};
```

然后我们到`fs.c`中实现函数`sys_lseek`

``` 
/* 重置文件的偏移指针，成功时则返回偏移量，失败的时候返回-1 */
int32_t sys_lseek(int32_t fd, int32_t offset, uint8_t whence){
  if(fd  0 && whence fd_inode->i_size;
  switch(whence){
    case SEEK_SET:
      new_pos = offset;
      break;
    case SEEK_CUR:
      new_pos = (int32_t)pf->fd_pos + offset;
      break;
    case SEEK_END:
      new_pos = file_size + offset;
  }
  if(new_pos  (file_size - 1)){
    return -1;
  }
  pf->fd_pos = new_pos;
  return pf->fd_pos;
}
```

紧接着立刻到`main`函数中实现

``` 
int main(void){
  put_str("I am Kernel\n");
  init_all();
  intr_enable();
  uint32_t fd = sys_open("/file1", O_CREAT);
  sys_close(fd);
  fd = sys_open("/file1", O_RDWR);
  printf("open /file1, fd:%d\n", fd);
  char buf[64] = {0};
  int read_bytes = sys_read(fd, buf, 18);
  printf("1_ read %d bytes:\n%s\n", read_bytes, buf);
  memset(buf, 0, 64);
  read_bytes = sys_read(fd, buf, 6);
  printf("2_ read %d bytes:\n%s\n", read_bytes, buf);
  memset(buf, 0, 64);
  read_bytes = sys_read(fd, buf, 6);
  printf("3_ read %d bytes:\n%s\n", read_bytes, buf);
  printf("______ SEEK_SET 0 ______ \n");
  sys_lseek(fd, 0, SEEK_SET)
  memset(buf, 0, 64);
  read_bytes = sys_read(fd, buf, 24);
  printf("4_ read %d bytes:\n%s\n", read_bytes, buf);
  sys_close(fd);
 while(1);//{
    //console_put_str("Main ");
  //};
  return 0;
}
```

这里我们在连续读30字节后又从头开始读24字节

![](http://imgsrc.baidu.com/forum/pic/item/58ee3d6d55fbb2fb805243d50a4a20a44723dcf8.jpg)

这里我们并没有关闭文件，但是指针确实指向了开头，我们通过查看输出就知道了。

## 0x03 实现文件删除

### 1. 回收 inode

这里我们删除文件涉及比较底层的`inode`，`block`等，相关函数是`unlink`，这些删除其实就是创建的逆过程，代码实现如下，当然先从底层开始，在`inode.c`中实现
