
## 代码分析

```c
         */

        if (strcmp(param->key, "source") == 0) {
                if (param->type != fs_value_is_string)
                        return invalf(fc, "VFS: Non-string source");
                if (fc->source)
                        return invalf(fc, "VFS: Multiple sources");
                fc->source = param->string;
                param->string = NULL;
                return 0;
        }
        return invalf(fc, "%s: Unknown parameter '%s'",
                      fc->fs_type->name, param->key);
}

int cgroup1_parse_param(struct fs_context *fc, struct fs_parameter *param)
{
    struct cgroup_fs_context *ctx = cgroup_fc2context(fc);
    struct cgroup_subsys *ss;
    struct fs_parse_result result;
    int opt, i;

    opt = fs_parse(fc, cgroup1_fs_parameters, param, &result);
    if (opt == -ENOPARAM) {
        if (strcmp(param->key, "source") == 0) {
            if (fc->source)
                return invalf(fc, "Multiple sources not supported");
            fc->source = param->string;
            param->string = NULL;
            return 0;
        }
        for_each_subsys(ss, i) {
            if (strcmp(param->key, ss->legacy_name))
                continue;
            if (!cgroup_ssid_enabled(i) || cgroup1_ssid_disabled(i))
                return invalfc(fc, "Disabled controller '%s'",
                               param->key);
            ctx->subsys_mask |= (1 key);
    }
    if (opt none = true;
            break;
        case Opt_all:
            ctx->all_ss = true;
            break;
        case Opt_noprefix:
            ctx->flags |= CGRP_ROOT_NOPREFIX;
            break;
        case Opt_clone_children:
            ctx->cpuset_clone_children = true;
            break;
        case Opt_cpuset_v2_mode:
            ctx->flags |= CGRP_ROOT_CPUSET_V2_MODE;
            break;
        case Opt_xattr:
            ctx->flags |= CGRP_ROOT_XATTR;
            break;
        case Opt_release_agent:
            /* Specifying two release agents is forbidden */
            if (ctx->release_agent)
                return invalfc(fc, "release_agent respecified");
            ctx->release_agent = param->string;
            param->string = NULL;
            break;
        case Opt_name:
            /* blocked by boot param? */
            if (cgroup_no_v1_named)
                return -ENOENT;
            /* Can't specify an empty name */
            if (!param->size)
                return invalfc(fc, "Empty name");
            if (param->size > MAX_CGROUP_ROOT_NAMELEN - 1)
                return invalfc(fc, "Name too long");
            /* Must match [\w.-]+ */
            for (i = 0; i size; i++) {
                char c = param->string[i];
                if (isalnum(c))
                    continue;
                if ((c == '.') || (c == '-') || (c == '_'))
                    continue;
                return invalfc(fc, "Invalid name");
            }
            /* Specifying two names is forbidden */
            if (ctx->name)
                return invalfc(fc, "name respecified");
            ctx->name = param->string;
            param->string = NULL;
            break;
    }
    return 0;
}
```

通过 PATCH 文件可以看出来（实则因为菜实在分析不来）

```diff
diff --git a/kernel/cgroup/cgroup-v1.c b/kernel/cgroup/cgroup-v1.c
index ee93b6e895874..527917c0b30be 100644
--- a/kernel/cgroup/cgroup-v1.c
+++ b/kernel/cgroup/cgroup-v1.c
@@ -912,6 +912,8 @@ int cgroup1_parse_param(struct fs_context *fc, struct fs_parameter *param)
         opt = fs_parse(fc, cgroup1_fs_parameters, param, &result);
         if (opt == -ENOPARAM) {
                 if (strcmp(param->key, "source") == 0) {
+                        if (param->type != fs_value_is_string)
+                                return invalf(fc, "Non-string source");
                         if (fc->source)
                                 return invalf(fc, "Multiple sources not supported");
                         fc->source = param->string;
```

如果 `key` 为 `source`，那么 `param->type` 必须被指定为 `string` 类型而不能是文件描述符，此时因为外面的 `cmd=FSCONFIG_SET_FD`，因此获取了文件结构在联合体当中。

```c
struct fs_parameter {
    const char                *key;                /* Parameter name */
    enum fs_value_type        type:8;                /* The type of value here */
    union {
        char                *string;
        void                *blob;
        struct filename        *name;
        struct file        *file;
    };
    size_t        size;
    int        dirfd;
};
```

在判断中可以看到这样一句：

```c
if (strcmp(param->key, "source") == 0) {
    if (fc->source)
        return invalf(fc, "Multiple sources not supported");
    fc->source = param->string;
    param->string = NULL;
    return 0;
}
```

此时将 `string` 保存在 `fc->source` 当中，因为它们共用内存，所以这里的 `string` 实际上是 `struct file` 结构体指针。

最后要 `free` 掉这个 `fs_context` 结构时，就意外地造成了这里的文件结构的 uaf，最后这个系统调用完成会触发 `fscontext_release`。

```c
void put_fs_context(struct fs_context *fc)
{
    struct super_block *sb;

    if (fc->root) {
        sb = fc->root->d_sb;
        dput(fc->root);
        fc->root = NULL;
        deactivate_super(sb);
    }

    if (fc->need_free && fc->ops && fc->ops->free)
        fc->ops->free(fc);

    security_free_mnt_opts(&fc->security);
    put_net(fc->net_ns);
    put_user_ns(fc->user_ns);
    put_cred(fc->cred);
    put_fc_log(fc);
    put_filesystem(fc->fs_type);
    kfree(fc->source);//这里意外地 free 掉了文件结构
    kfree(fc);
}

static int fscontext_release(struct inode *inode, struct file *file)
{
    struct fs_context *fc = file->private_data;

    if (fc) {
        file->private_data = NULL;
        put_fs_context(fc);
    }
    return 0;
}
```

### 代码调试与触发 UAF

第一步打断点 `__do_sys_fsconfig`，然后跟到图示这个位置，可以发现获取到了文件结构了。

![](https://xia0ji233.pro/2024/03/13/CVE-2021-4145/3.png)

随后跟到这个位置

![](https://xia0ji233.pro/2024/03/13/CVE-2021-4145/4.png)

这里会有调用刚刚的 `cgroup1_parse_param`，当然也可以直接下断点 `continue` 过去。

![](https://xia0ji233.pro/2024/03/13/CVE-2021-4145/5.png)

当然这里可以看到 `source` 直接被取走了，保存到了 `fc` 结构当中。

随后下断在 `fscontext_release`，然后 `continue` 过去，走到 `kfree` 这和位置可以发现 `source` 被释放。

![](https://xia0ji233.pro/2024/03/13/CVE-2021-4145/6.png)

这里也能看到作者原意是想在这里释放 `source` 字符串，但是这里释放了 `file` 文件结构指针，调试的时候可以和之前对一下，发现地址是一致的，因此这里造成了 uaf。

### 延长竞争时间

这里采用 `writev` 写入大量数据使得文件拿锁的时间加长。为了调试 `exp`，可以用 `add-symbol-file` 命令去添加符号，这里可以选择断 `write_cmd` 的 `writev` 函数，因为这里会因为写入数据量过大而长期持有锁，`writev` 就会尝试持续获得锁。

![](https://xia0ji233.pro/2024/03/13/CVE-2021-4145/7.png)

随后经过系统调用来到 `do_writev` 函数

![](https://xia0ji233.pro/2024/03/13/CVE-2021-4145/8.png)

不过这里多线程比较难调，也不放调试具体过程了，感觉原理还是比较浅显易懂的。

### 总结

我们可以总结出以下的利用思路：

- `fsconfig` 系统调用的代码存在类型混淆漏洞，间接导致了可以使得某文件描述符结构被 uaf
- 通过写入大量数据延长竞争时间，通过建立一个链接的方式绕过 `open` 时赋予的标记位，使得两个线程可以卡在权限校验之后。
- 第三个线程在第二个线程卡住的时间申请 `/etc/passwd` 文件的结构，替换线程 2 正在写入的文件，完成漏洞利用。

分析的还有很多不足之处，如果有讲的不好的地方恳请师傅多多包涵并帮助指正。
