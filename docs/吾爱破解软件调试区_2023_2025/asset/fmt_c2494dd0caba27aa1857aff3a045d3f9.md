
# 文件系统头文件定义

```c
#define __FS_FILE_H

#include "stdint.h"

#include "inode.c"

/* 文件结构 */
struct file {
  uint32_t  fd_pos;     //记录当前文件操作的偏移地址，以0为起始，最大为文件大小-1
  uint32_t fd_flag;
  struct inode* fd_inode;
};

/* 标准输入输出描述符 */
enum std_fd{
  stdin_no,         //0 标准输入
  stdout_no,        //1 标准输出
  stderr_no         //2 标准错误
};

/* 位图类型 */
enum bitmap_type{
  INODE_BITMAP,     //inode位图
  BLOCK_BITMAP,     //块位图
};

#define MAX_FILE_OPEN 32    //系统可以打开的最大文件数
#endif
```

其中的`fd_flag`就是文件操作标识，类系`O_READONLY`

下面我们来构建文件表，创建文件`fs/file.c`

## 文件表实现

```c
#include "file.h"
#include "fs.h"
#include "stdint.h"
#include "thread.h"
#include "global.h"
#include "string.h"
#include "inode.h"
#include "debug.h"
#include "memory.h"
#include "interrupt.h"
#include "stdio-kernel.h"
#include "super_block.h"

/* 文件表 */
struct file file_table[MAX_FILE_OPEN];

/* 从文件表file_table中获取一个空闲位，并返回下标，失败就返回-1 */
int32_t get_free_slot_in_global(void){
  uint32_t fd_idx = 3;
  while(fd_idx fd_table[local_fd_idx] == -1){  //-1表示可使用，我们在最开始init的时候就置-1了
      cur->fd_table[local_fd_idx] = global_fd_idx;
      break;
    }
    local_fd_idx++;
  }
  if(local_fd_idx == MAX_FILES_OPEN_PER_PROC){
    printk("exceed max open files_per_proc\n");
    return -1;
  }
  return local_fd_idx;
}

/* 分配一个inode，返回i结点号 */
int32_t inode_bitmap_alloc(struct partition part){
  int32_t bit_idx = bitmap_scan(&part->inode_bitmap, 1);
  if(bit_idx == -1){
    return -1;
  }
  bitmap_set(&part->inode_bitmap, bit_idx, 1);
  return bit_idx;
}

/* 分配1个扇区，返回扇区地址 */
int32_t block_bitmap_alloc(struct partition part){
  int32_t bit_idx = bitmap_scan(&part->block_bitmap, 1);
  if(bit_idx == -1){
    return -1;
  }
  bitmap_scan(&part->block_bitmap, bit_idx, 1);
  return (part->sb->data_start_lba + bit_idx);
}

/* 将内存中的bitmap第bit_idx位所在的512字节同步到硬盘 */
void bitmap_sync(struct partition, uint32_t bit_idx, uint8_t btmp){
  uint32_t off_sec = bit_idx / 4096;    //该位相对与位图扇区的偏移量
  uint32_t off_size = off_sec * BLOCK_SIZE;     //偏移的字节数
  uint32_t sec_lba;                 //扇区地址
  uint8_t* bitmap_off;              //位图偏移

  /* 需要被同步到硬盘的位图只有inode_bitmap和block_bitmap */
  switch(btmp){
    case INODE_BITMAP:
      sec_lba = part->sb->inode_bitmap_lba + off_sec;
      bitmap_off = part->inode_bitmap.bits + off_size;
      break;
    case BLOCK_BITMAP:
      sec_lba = part->sb->block_bitmap_lba + off_sec;
      bitmap_off = part->block_bitmap.bits + off_size;
      break;
  }
  ide_write(part->my_disk, sec_lba, bitmap_off, 1);
}
```

这里面我们构造了一些对于普通文件的操作，注释也十分详细就不多说了

## 目录相关

我们刚刚已经实现了`inode`以及普通文件的操作，现在我们需要再添加一些对于目录的操作，这里也就是简单的读出目录项等然后复制就行，具体代码为`fs/dir.h`

```c
#include "dir.h"
#include "super_block.h"
#include "ide.h"
#include "inode.h"
#include "stdint.h"
#include "global.h"
struct dir root_dir;    //根目录

/* 打开根目录 */
void open_root_dir(struct partition* part){
  root_dir.inode = inode_open(part, part->sb->root_inode_no);
  root_dir.dir_pos = 0;
}

/* 在分区part上打开i结点作为inode_no的目录并返回目录指针 */
struct dir* dir_open(struct partition* part, uint32_t inode_no){
  struct dir* pdir = (struct dir*)sys_malloc(sizeof(struct dir));
  pdir->inode = inode_open(part, inode_no);
  pdir->dir_pos = 0;
  return pdir;
}

/* 在part分区内的pdir目录内寻找名为name的文件或者目录，找到后返回true并将其目录项存入dir_e，否则返回false */
bool search_dir_entry(struct partition* part, struct dir* pdir, const char* name, struct dir_entry* dir_e){
  uint32_t block_cnt = 140;     //12个直接块，加上一级间接块，有128个，所以总共140个
  /* 12个直接块和128个间接块，共560字节 */
  uint32_t* all_blocks = (uint32_t*)sys_malloc(48 + 512);
  if(all_blocks == NULL){
    printk("search_dir_entry: sys_malloc for all_blocks failed");
    return false;
  }
  uint32_t block_idx = 0;
  while(block_idx inode->i_sectors[block_idx];
    block_idx++;
  }
  block_idx = 0;
  if(pdir->inode->i_sectors[12] != 0){  //如果含有1级块，则我们会直接将那一块进行读出
    ide_read(part->my_disk, pdir->inode->i_sectors[12], all_blocks + 12, 1);
  }
  /* 至此，all_blocks已经存储了全部的文件扇区地址 */
  uint8_t* buf = (uint8_t*)sys_malloc(SECTOR_SIZE);
  struct dir_entry* p_de = (struct dir_entry*)buf;      //p_de为指向目录项的指针
  uint32_t dir_entry_size = part->sb->dir_entry_size;
  uint32_t dir_entry_cnt = SECTOR_SIZE/dir_entry_size;  //1扇区可以容纳的目录项个数
  /* 开始在所有块中查找 */
  while(block_idx my_disk, all_block[block_idx], buf, 1);
    uint32_t dir_entry_idx = 0;
    /* 遍历扇区中的所有目录项 */
    while(dir_entry_idx filename, name)){
        memcpy(dir_e, p_de, dir_entry_size);
        sys_free(buf);
        sys_free(all_blocks);
        return true;
      }
      dir_entry_idx++;
      p_de++;
    }
    block_idx++;
    p_de = (struct dir_entry*)buf;  //这里我们的p_de要重新指向新的块，所以要更新指针为开头
    memset(buf, 0, SECTOR_SIZE);
  }
  sys_free(buf);
  sys_free(all_blocks);
  return false;
}

/* 关闭目录 */
void dir_close(struct dir* dir){
  /****************** 根目录无法关闭 *******************
   * 1 根目录自打开后就不应该关闭，否则还需要再次open_root_dir
   * 2 root_dir所在的内存是低端1MB之内，并非在堆中，free会出问题*/
  if(dir == &root_dir){
    /* 不做任何处理，直接返回 */
    return;
  }
  inode_close(dir->inode);
  sys_free(dir);
}

/* 在内存中初始化目录项p_de */
void create_dir_entry(char* filename, uint32_t inode_no, uint8_t file_type, struct dir_entry* p_de){
  ASSERT(strlen(filename) filename, filename, strlen(filename));
  p_de->i_no = inode_no;
  p_de->f_type = file_type;
}
```

上面实现了一些基本操作，我们这里还需要一点工作，那就是增加一个函数`sync_dir_entry()`，这个函数比较复杂，他的功能是在指定目录下添加目录项，如下：
