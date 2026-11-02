
# [漏洞分析] 【Linux kernel 漏洞复现】CVE-2022-0185

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1842879
- 抓取时间: 2026-11-01T21:55:40

## CVE-2022-0185

算是第一次进行Linux的漏洞分析，其中还是有许多不足，复现还有几种手法没实现完全

![](http://imgsrc.baidu.com/forum/pic/item/77c6a7efce1b9d16e4308055b5deb48f8c54640a.jpg)

## 一、背景

![](http://imgsrc.baidu.com/forum/pic/item/9f2f070828381f304cb1979def014c086e06f05e.jpg)

由[www.willsroot.io/2022/01/cve-2022-0185.html](https://www.willsroot.io/2022/01/cve-2022-0185.html)大师提出，以此赢得了31337刀的奖金(但看评论好像本来是50K，最后起了点矛盾降价惹)以及成功在google kctf中实现容器逃逸

整个漏洞利用了VFS中`fsconfig`某个参数实现的函数过程当中的一个整形溢出来实现越界写，其中利用手法也是十分精彩，原作者博客是利用到他曾经在 corctf2021上出的一道 [www.willsroot.io/2021/08/corctf-2021-fire-of-salvation-writeup.html](https://www.willsroot.io/2021/08/corctf-2021-fire-of-salvation-writeup.html)，此外作者好朋友D3V17也在当年出了同样一道题目，但是难度会在前面的题更大，[syst3mfailure.io/wall-of-perdition/](https://syst3mfailure.io/wall-of-perdition/),其中便是使用`msg_msg`搭配`userfaultfd`来实现任意写或者读，而为了应用到本次漏洞，考虑到高版本`userfaultfd`无法为普通用户使用，所以让 FUSE来代替`userfaultfd`，一般前者不会在CTF赛事当中出现，他需要在有着完备的环境条件才可使用，就比如说真机，而 BitesByWill师傅也顺便写了一个简单的`fuse`库来辅助利用orz

## 二、文件系统基础知识

### 1.VFS基本概念

早就听说过虚拟文件系统的大名，网上的解释如下：

> VFS（Virtual Filesystem Switch）称为虚拟文件系统或虚拟文件系统转换，是一个内核软件层，在具体的文件系统之上抽象的一层，用来处理与Posix文件系统相关的所有调用，表现为能够给各种文件系统提供一个通用的接口，使上层的应用程序能够使用通用的接口访问不同文件系统，同时也为不同文件系统的通信提供了媒介

从概念上来看其实十分的简单易懂,也就是类似开发中的面向接口编程了,他为不同的文件系统定义了一个普遍的接口,所以不同文件系统只需要按照这个规定好的接口来构造即可,具体实现部分可以按照自己的思路来

在我们的oS当中,整体布局可以用下面这个图来表示,CSDN找的,侵删.不得不说这个图画的十分的好,我就自己不献丑了

![](https://img-blog.csdn.net/20180318233241252?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3UwMTA0ODc1Njg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

平常我们所使用的系统调用也是第一先经过VFS,通过他提供的接口来进行系统调用的操作,当然具体实现还是看底层真正存在的文件系统类型了,但是在用户层程序员的眼中看来并无太大区别,由于不同文件系统的多样性和用户系统调用的复杂性,VFS文件系统这一抽象层的存在是十分具有存在意义的.

### 2.VFS抽象接口

上述示例中提到VFS也有自己的文件模型，用来支持操作系统的系统调用。下面是VFS抽象模型支持的所有Linux系统调用：

- 文件系统相关：`mount`, `umount`, `umount2`, `sysfs`, `statfs`, `fstatfs`, `fstatfs64`, `ustat`
- 目录相关：`chroot`，`pivot_root`，`chdir`，`fchdir`，`getcwd`，`mkdir`，`rmdir`，`getdents`，`getdents64`，`readdir`，`link`，`unlink`，`rename`，`lookup_dcookie`
- 链接相关：`readlink`，`symlink`
- 文件相关：`chown`， `fchown`，`lchown`，`chown16`，`fchown16`，`lchown16`，`hmod`，`fchmod`，`utime`，`stat`，`fstat`，`lstat`，`acess`，`oldstat`，`oldfstat`，`oldlstat`，`stat64`，`lstat64`，`lstat64`，`open`，`close`，`creat`，`umask`，`dup`，`dup2`，`fcntl`， `fcntl64`，`select`，`poll`，`truncate`，`ftruncate`，`truncate64`，`ftruncate64`，`lseek`，`llseek`，`read`，`write`，`readv`，`writev`，`sendfile`，`sendfile64`，`readahead`

### 3.VFS Common File Model

就是说文件系统的抽象化模型,在我们硬盘之中,扇区普遍设置为512字节,而我们内存与硬盘的交互是十分缓慢的,所以一般数据交换并不以扇区(sectors)为单位,而是以块(block)为单位,块一般为4KB大小

硬盘中文件系统就是用来存放文件信息的,其中不仅有文件类容,还需要存放一些关于文件的信息,例如说文件归属,文件权限等,因此在文件系统中会存放`inode`节点,在之前自己实现的OS当中是存放了文件数据的扇区号等等用来方便访问.我们最初的`open`系统调用实际上就是将`inode`节点从硬盘转到内存了而已.

文件系统中定义了四个较为重要的对象,他们合在一起便构建了我们的统一文件模型,接下来分别介绍他们

#### 1.`Superblock`

超级块,Unix的特色,里面存放了一系列我们需要使用的元信息,相当于是一个统筹全局的资料库

![](http://imgsrc.baidu.com/forum/pic/item/d058ccbf6c81800a8f226cc8f43533fa838b4735.jpg)

他的数据结构体被存放于`include/linux/fs.h`中,大家有兴趣可以详细查看

```
struct super_block {
        struct list_head        s_list;                /* Keep this first */                 //链接各超级块
        dev_t                        s_dev;                /* search index; _not_ kdev_t */
        unsigned char                s_blocksize_bits;
        unsigned long                s_blocksize;
        loff_t                        s_maxbytes;        /* Max file size */
        struct file_system_type        *s_type;
        const struct super_operations        *s_op;                                 //操作文件系统的函数指针
        const struct dquot_operations        *dq_op;
        const struct quotactl_ops        *s_qcop;
        const struct export_operations *s_export_op;
        unsigned long                s_flags;
        unsigned long                s_iflags;        /* internal SB_I_* flags */
        unsigned long                s_magic;
        struct dentry                *s_root;                                 //根目录入口点
        struct rw_semaphore        s_umount;
        int                        s_count;
        atomic_t                s_active;

        ...

} __randomize_layout;
```

一般我们的超级块都会存放在该文件系统的头部,由于linux一般可以挂在多个文件系统,所以这个超级块的`s_list`字段就是用来链接下一个超级快

#### 2.`inode`

他标识了一个存在于硬盘中的文件，他在内存当中是以VFS `inode`存在，在硬盘中则可能有些许不同，可能包含了存放在硬盘中不需要的一些字段，可能有某种不以`inode`进行管理的文件系统，说的就是你们，FAT和Reiserfs，VFS的处理方式就是将其中的特定信息赋值给内存中的`inode`

```
/*

 * Keep mostly read-only and often accessed (especially for

 * the RCU path lookup and 'stat' data) fields at the beginning

 * of the 'struct inode'

 */

struct inode {

        umode_t                        i_mode;

        unsigned short                i_opflags;

        kuid_t                        i_uid;

        kgid_t                        i_gid;

        unsigned int                i_flags;

        ...

} __randomize_layout;
```

#### 3.`dentry`

目录项缓存，Linux使用它来快速访问此前的查找操作的结果。在VFS连同文件系统实现读取的一个目录项（目录或文件）的数据之后，则创建一个`dentry`实例，以缓存找到的数据，这样下次我们寻找该文件/目录则会首先从该缓存找起，而不是再次通过文件名来到查找相应目录项了

```
struct dentry {
        /* RCU lookup touched fields */
        unsigned int d_flags;                /* protected by d_lock */
        seqcount_spinlock_t d_seq;        /* per dentry seqlock */
        struct hlist_bl_node d_hash;        /* lookup hash list */
        struct dentry *d_parent;        /* 父目录 */
        struct qstr d_name;
        struct inode *d_inode;                /* 该文件名所属inode，如果不存在则为NULL */
        unsigned char d_iname[DNAME_INLINE_LEN];        /* small names */

        /* Ref lookup also touches following */
        struct lockref d_lockref;        /* per-dentry lock and refcount */
        const struct dentry_operations *d_op;
        struct super_block *d_sb;        /* The root of the dentry tree */
        unsigned long d_time;                /* used by d_revalidate */
        void *d_fsdata;                        /* fs-specific data */

        union {
                struct list_head d_lru;                /* LRU list */
                wait_queue_head_t *d_wait;        /* in-lookup ones only */
        };
        struct list_head d_child;        /* child of parent list */
        struct list_head d_subdirs;        /* our children */
        /*
         * d_alias and d_rcu can share memory
         */
        union {
                struct hlist_node d_alias;        /* inode alias list */
                struct hlist_bl_node d_in_lookup_hash;        /* only for in-lookup ones */
                 struct rcu_head d_rcu;
        } d_u;
} __randomize_layout;
```

#### 4.传统`mount`系统调用

我们平时都会使用`mount`系统调用来挂在某一文件系统，我们可以查看一下Linux手册

```
SYNOPSIS

       mount [-h|-V]

       mount [-l] [-t fstype]

       mount -a [-fFnrsvw] [-t fstype] [-O optlist]

       mount [-fnrsvw] [-o options] device|mountpoint

       mount [-fnrsvw] [-t fstype] [-o options] device mountpoint

       mount --bind|--rbind|--move olddir newdir

       mount --make-[shared|slave|private|unbindable|rshared|rslave|rprivate|runbindable] mountpoint
```

我们通常会采用下面命令来进行挂载

```
mount -t xfs /dev/sdb1 -o /mnt/temp
```

具体含义就是将`/dev/sdb1`设备代表的文件系统以xfs文件系统的格式来挂载到`/mnt/temp`目录下，这样就加入到了我们`/`根目录的文件树当中，这样以来我们才可以正常访问其中内容，除了Linux实际上windows也存在挂载操作，但为了简化我们用户的使用就隐藏了这一点

除了命令行使用的`mount`，我们来关注一下代码当中使用的`mount`系统调用，以下查看手册

```
man 2 mount
```
