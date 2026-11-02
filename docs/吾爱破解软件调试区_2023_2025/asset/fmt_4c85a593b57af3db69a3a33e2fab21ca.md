
```c
      uint32_t bytes_written  = file_write(wr_file, buf, count);
      return bytes_written;
    }else{
      console_put_str("sys_write: not allowed to write file without flag O_RDWR or O_WRONLY\n");
      return -1;
    }
  }
}

int32_t sys_close(int32_t fd) {
  int32_t ret = -1;   // 返回值默认为-1,即失败
   if (fd > 2) {
      uint32_t global_fd = fd_local2global(fd);
      if(is_pipe(fd)){
        /* 如果此管道上的描述符都被关闭，释放管道的环形缓冲区 */
        if(--file_table[global_fd].fd_pos == 0){
          mfree_page(PF_KERNEL, file_table[global_fd].fd_inode, 1);
          file_table[global_fd].fd_inode = NULL;
        }
        ret = 0;
      }else{
        ret = file_close(&file_table[global_fd]);
      }
      running_thread()->fd_table[fd] = -1; // 使该文件描述符位可用
    }
  return ret;
}
```

由于我们的管道也算文件，因此我们需要修改其中的 `read`、`write` 以及 `close`，然后还有就是我们这个算是匿名管道，是由父子进程共享的，因此在 `fork` 也要增加管道的打开数，我们修改 `fork.c` 如下：

```c
/* 更新inode打开数 */
static void update_inode_open_cnts(struct task_struct* thread) {
  int32_t local_fd = 3, global_fd = 0;
  while (local_fd fd_table[local_fd];
    ASSERT(global_fd i_open_cnts++;
      }
    }
    local_fd++;
  }
}
```

由于有了管道，我们还需要到 `wait_exit.c` 当中修改一下：

```c
  /* 关闭进程打开的文件 */
  uint8_t local_fd = 3;
  while(local_fd fd_table[local_fd] != -1){
      if(is_pipe(local_fd)){
        uint32_t global_fd = fd_local2global(local_fd);
        if(--file_table[global_fd].fd_pos == 0){
          mfree_page(PF_KERNEL, file_table[global_fd].fd_inode, 1);
          file_table[global_fd].fd_inode = NULL;
        }
      }else{
        sys_close(local_fd);
      }
    }
    local_fd++;
  }
```

我们在收回子进程资源的时候管道这里记得判断一下，就是如此，然后 `pipe` 的系统调用我们自行加上就可以了。

### 4. 利用管道实现通信

我们现在来编写用户进程来验证一下父子之间的通信，如下：

```c
#include "stdio.h"
#include "syscall.h"
#include "string.h"
int main(int argc, char** argv) {
   int32_t fd[2] = {-1};
   pipe(fd);
   int32_t pid = fork();
   if(pid) {          // 父进程
      close(fd[0]);  // 关闭输入
      write(fd[1], "Hi, my son, I love you!", 24);
      printf("\nI`m father, my pid is %d\n", getpid());
      return 8;
   } else {
      close(fd[1]);  // 关闭输出
      char buf[32] = {0};
      read(fd[0], buf, 24);
      printf("\nI`m child, my pid is %d\n", getpid());
      printf("I`m child, my father said to me: \"%s\"\n", buf);
      return 9;
   }
}
```

这里就是父进程往管道里面写入数据，然后子进程读出，然后我们按照原先的方法写入文件系统。

然后我们运行虚拟机来查看结果：

![](https://attach.52pojie.cn/forum/202302/20/160417b2wqzz2w2zqm5qwf.png)

Screenshot 2023-02-20 141125.png (33.22 KB, 下载次数: 0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU5MTc5OHwxYjNhZTAwYnwxNzkzNTcwMjQ5fDE2ODA4NnwxNzQ4NDEz&nothumb=yes)

2023-2-20 16:04 上传

可以看到正如我们所料，完全是一致的，这里父进程首先返回，然后子进程就变成了孤儿进程，接下来就会过继给 `init` 进程领养，这里比较重要的一点是子进程确实从管道获取了父进程发给他的消息。

### 5. 在 shell 中支持管道

我们平时 Linux 操作中肯定少不了 `cat|grep` 操作，这个竖线表示的就是管道，也就是通过这个管道我们实现数据过滤功能，这里之所以可以这样使用是因为我们增加了输入输出重定向的功能，大伙应该还记得刚刚我们修改 `read` 和 `write` 系统调用的时候在标准情况下还是加上了管道的判断，这就是为了现在这一步，接下来我们来完善 `shell/pipe.c`：

```c
/* 将文件描述符old_local_fd重定位为new_local_fd */
void sys_fd_redirect(uint32_t old_local_fd, uint32_t new_local_fd){
  struct task_struct* cur = running_thread();
  /* 针对恢复标准描述符 */
  if(new_local_fd fd_table[old_local_fd] = new_local_fd;
  }else{
    uint32_t new_global_fd = cur->fd_table[new_local_fd];
    cur->fd_table[old_local_fd] = new_global_fd;
  }
}
```

这里就是修改本进程的表而已，然后我们在 `shell.c` 当中新增代码,这里注意上面的我们还需要添加个系统调用。下面就是 shell 的新增代码，如下：

```c
/* 简单的shell */
void my_shell(void){
  cwd_cache[0] = '/';
  while(1){
    print_prompt();
    memset(cmd_line, 0, MAX_PATH_LEN);
    memset(final_path, 0, MAX_PATH_LEN);
    readline(cmd_line, MAX_PATH_LEN);
    if(cmd_line[0] == 0){ //如果只输入了一个回车
      continue;
    }
    /* 针对管道的处理 */
    char* pipe_symbol = strchr(cmd_line, '|');  //获取第一个管道地址
    if(pipe_symbol){
      /* 支持多重管道操作，如cmd1|cmd2|..|cmdn,
       * 这里cmd1和cmdn的标准输出输入需要单独处理*/
      /* 1 生成管道 */
      int32_t fd[2] = {-1};     //fd[0]输入，fd[1]输出
      pipe(fd);
      /* 将标准输出重定向到fd[1],使后面的输出信息重定向到内核环形缓冲区 */
      fd_redirect(1, fd[1]);
      /* 2 第一个命令 */
      char* each_cmd = cmd_line;
      pipe_symbol = strchr(each_cmd, '|');
      *pipe_symbol = 0;
      /* 执行第一个命令，命令的输出会写入环形缓冲区 */
      argc = -1;
      argc = cmd_parse(each_cmd, argv, ' ');
      cmd_execute(argc, argv);  //由于子进程会拷贝父进程信息，所以这里一起重定向了
      /* 跨过'|',处理下一个命令 */
      each_cmd = pipe_symbol + 1;
      /* 将标准输入重定向到fd[0],使之指向内核环形缓冲区 */
      fd_redirct(0, fd[0]);
      /* 3 中间的命令，输入和输出都是指向环形缓冲区 */
      while((pipe_symbol = strchr(each_cmd, '|'))){
        *pipe_symbol = 0;
        argc = -1;
        argc = cmd_parse(each_cmd, argv, ' ');
        cmd_execute(argc, argv);
        each_cmd = pipe_symbol + 1;
      }
      /* 4 处理管道中最后一个命令 */
      /* 这里将标准输出恢复屏幕 */
      fd_redirect(1, 1);
      /* 执行最后一个命令 */
      argc = -1;
      argc = cmd_parse(each_cmd, argv, ' ');
      cmd_execute(argc, argv);

      /* 5 将标准输入恢复为键盘 */
      fd_redirect(0, 0);
      /* 6 关闭管道 */
      close(fd[0]);
      close(fd[1]);
    }else{  //一般的无管道操作
      argc = -1;
      argc = cmd_parse(cmd_line, argv, ' ');
      if(argc == -1){
        printf("num of argument exceed %d\n", MAX_ARG_NR);
        continue;
      }
      cmd_execute(argc, argv);
    }
  }
  panic("my_shell: should not be here");
}
```

这是将命令的选择和执行单独封装为 `cmd_execute()`，然后根据管道符号 `|` 依次进行重定位。接下来我们修改一下 `cat` 命令来方便测试管道 shell：

```c
#include "syscall.h"
#include "stdio.h"
#include "string.h"

int main(int argc, char** argv){
  if(argc > 2){
    printf("cat: only support 1 argument.\neg: cat filename\n");
    exit(-2);
  }
  if(argc == 1){
    char buf[512] = {0};
    read(0, buf, 512);
    printf("%s", buf);
    exit(0);
  }
  int buf_size = 1024;
  char abs_path[512] = {0};
  void* buf = malloc(buf_size);
  if(buf == NULL){
    printf("cat: malloc memory failed\n");
    return -1;
  }
  if(argv[1][0] != '/'){    //如果输入的不是绝对路径
    getcwd(abs_path, 512);
    strcat(abs_path, "/");
    strcat(abs_path, argv[1]);
  }else{
    strcpy(abs_path, argv[1]);
  }
  int fd = open(abs_path, O_RDONLY);
  if(fd == -1){
    printf("cat: open: open %s failed\n",argv[1]);
    return -1;
  }
  int read_bytes = 0;
  while(1){
    read_bytes = read(fd, buf, buf_size);
    if(read_bytes == -1){
      break;
    }
    write(1, buf, read_bytes);
  }
  free(buf);
  close(fd);
  return 66;
}
```

这里只需要添加一个简单的判断即可，因为当只有一个 `cat` 的时候我们的标准输入输出已经被重定向，所以这里我们直接输出直接读标准 `fd` 就行了。这里我们还需要重新用 `compile.sh` 来重新编译。然后写入文件系统。

最后我们实现一个简单的帮助系统调用，实现在 `fs/fs.c` 中，这样能帮助我们更贴近现实的操作系统，如下：

```c
/* 显示系统支持的内部命令 */
void sys_help(void){
  printk("buildin commands:\n\
      ls: show directory of file\n\
      cd: change curent work directory\
      mkdir: create a directory\
      rmdir: remove a empty directory\n\
      rm: remove a regular file\n\
      pwd: show current work directory\n\
      ps: show process information\n\
      clear: clear screen\n\
      shortcut key:\n\
      ctrl+l: clear screen\n\
      ctrl+u: clear input\n\n");
}
```

可以看到这是个简单到不能再简单的系统调用，然后我们立刻来最后一次上机测试：

![](https://attach.52pojie.cn/forum/202302/20/160419kxzvfifis79bxijw.png)

Screenshot 2023-02-20 155115.png (27.06 KB, 下载次数: 1)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU5MTc5OXwyYTI0ZWQ0NHwxNzkzNTcwMjQ5fDE2ODA4NnwxNzQ4NDEz&nothumb=yes)

2023-2-20 16:04 上传

从中可以看到我们使用 `ls -l|cat|./cat|../cat` 来不断测试发现确实照常输出了目录内容。

## 0x04 总结

今天对于管道大家可能会有更深的认识，这里我们整个操作系统已经基本上完善了，这也加深了我对于内核的认识，不知道有没有人跟我一起实现到这里，不过之前有位友友跟我一起实现但是后面不知道是不是有事还是什么没看到动静了，但是整体跟下来对于操作系统确实不会局限与以往的书本了，还是挺有用的。终于整个操作系统实现完毕，这两天加班是真累，等会儿出门吃个烤鸡晚上回来写一篇整体总结，就这样了。

本次我的所有源码已在 github 上成功上传，分支名定为 `Process`，欢迎各位指教。

> [https://github.com/PeiandHao/Simple_OS_For_Studing](https://github.com/PeiandHao/Simple_OS_For_Studing)
