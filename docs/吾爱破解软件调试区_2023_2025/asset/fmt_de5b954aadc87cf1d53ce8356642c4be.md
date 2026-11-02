
![](https://attach.52pojie.cn/forum/202404/19/101524lu5h7t7jcu4dfeuf.png)

image-20240417192445712.png (73.03 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY5MDMwOHw1YjgwZDAzM3wxNzkzNTcwMDcxfDE2ODA4NnwxOTE1MzUw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDMwOHw1YjgwZDAzM3wxNzkzNTcwMDcxfDE2ODA4NnwxOTE1MzUw&nothumb=yes)

2024-4-19 10:15 上传

这里就只用知道这是个安全描述符（结构体指针），点击`SECURITY_ATTRIBUTES`（安全属性，也是个结构体指针），由于我的MSDN有点问题，就直接手写他的结构体

```
typedef struct _SECURITY_ATTRIBUTES {
    DWORD nLength;
    LPVOID lpSecurityDescriptor;
    BOOL bInheritHandle;
} SECURITY_ATTRIBUTES, *PSECURITY_ATTRIBUTES;
```

4> `nLength`

存储当前结构体大小的值

4> `lpSecurityDescriptor`

指向安全描述符的指针，意思为不同用户被赋予了何种权限。在我们写代码的时候通常不用去特别关注他，他默认采用的安全设置就跟你的父进程是一样的

4> `bInheritHandle`

确定在创建新进程时是否继承返回的句柄。如果此字段设置为非零值，则新进程将继承句柄。如果为 0，则新进程不会继承句柄。

介绍完后，所以如果不继承的话那就

```
CreateEvent(NULL,FALSE,FALSE,NULL)
```

（先不管其他参数的意思）

如果要继承，那就要创建`SECURITY_ATTRIBUTES`结构体

```
SECURITY_ATTRIBUTES sa;
ZeroMemory(&sa, sizeof(SECURITY_ATTRIBUTES));//初始化操作
sa.nLength = sizeof(SECURITY_ATTRIBUTES);//结构体大小
sa.bInheritHandle = TRUE;//可以继承
CreateEvent(sa,FALSE,FALSE,NULL);
```

这样就能继承了，因此，为了彰显句柄表是否能够继承，那就应该还要在添加一个成员，就是能否被继承的标志

![](https://attach.52pojie.cn/forum/202404/19/101543ptw242memhbbmx2f.png)

image-20240417213135725.png (41.56 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5MDMwOXw5Y2ExYjIxOHwxNzkzNTcwMDcxfDE2ODA4NnwxOTE1MzUw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDMwOXw5Y2ExYjIxOHwxNzkzNTcwMDcxfDE2ODA4NnwxOTE1MzUw&nothumb=yes)

2024-4-19 10:15 上传

句柄表的0代表不继承，1代表继承

如果是父子进程的话，那么子进程是有机会直接继承过去的

比如说，在创建进程时

```
if (!CreateProcess(
                ChildProcessName,        //对象名称的完整路径
                CommandLine,                //命令行参数
                NULL,                                //不继承进程句柄
                NULL,                                //不继承线程句柄
                FALSE,                                //不继承句柄
                0,                                        //没有创建标志
                NULL,                                //使用父进程环境变量
                NULL,                                //使用父进程目录作为当前目录，可以自己设置目录
                &si,                                //STARTUPINFO结构体
                &pi)                                //PROCESS_INFORMATION结构体
FALSE,                                //不继承句柄
```

这里我写的是`FALSE`，子进程是IE，父进程的句柄表跟子进程没有关系，也就是说子进程会有一个独立的句柄表，哪怕父进程里有可以的继承的，也没有任何关系

如果改成`TRUE`，那子进程就会得到父进程中允许继承的句柄表

![](https://attach.52pojie.cn/forum/202404/19/101559v1l2cqf1a8f3z4qq.png)

image-20240417215144489.png (40.12 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY5MDMxMHw0N2NlNWUzMHwxNzkzNTcwMDcxfDE2ODA4NnwxOTE1MzUw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDMxMHw0N2NlNWUzMHwxNzkzNTcwMDcxfDE2ODA4NnwxOTE1MzUw&nothumb=yes)

2024-4-19 10:15 上传

这样子进程就可以直接使用父进程的句柄了

```
NULL,                                //不继承进程句柄
NULL,                                //不继承线程句柄
```

再看进程与线程

```
[in, optional]           LPSECURITY_ATTRIBUTES lpProcessAttributes, // SD
[in, optional]           LPSECURITY_ATTRIBUTES lpThreadAttributes,  // SD
```

如果那个需要继承，就把那个定义出来，写成`TRUE`

3> 总结

- 每个进程对应一个内核对象

- 有两种方式来共享内核

这里的知识有点绕，需要多多消化，查阅资料
