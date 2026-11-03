
```javascript
O = function (e, t, a, i) {
    var f = wuye4(5);
    e = e.replace(/(^\s*)|(\s*$)/g, ""),
        t = t.replace(/(^\s*)|(\s*$)/g, "");
    var h = "";
    if (!i) {
        wuye4.i(wuye4(34).a)(131);
        var w = wuye4.i(wuye4(34).b)("010001", "", '00b5eeb166e069920e80bebd1fea4829d3d1f3216f2aabe79b6c47a3c18dcee5fd22c2e7ac519cab59198ece036dcf289ea8201e2a0b9ded307f8fb704136eaeb670286f5ad44e691005ba9ea5af04ada5367cd724b5a26fdb5120cc95b6431604bd219c6b7d83a6f8f24b43918ea988a76f93c333aa5a20991493d4eb1117e7b1');
        h = wuye4.i(wuye4(34).c)(w, t)
    }
    return h
}
```

### 2.`TGT`接口

请求`1.tickets`接口后会响应`TGT-`开头的值，接着请求`2.TGT`接口会响应一个`ST-`开头的值

![](https://attach.52pojie.cn/forum/202405/01/134428oz1bbb21db22dq1d.png)

image-20240428170250195.png (177.43 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5MzIzOXwzMTc2YzQ2ZXwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzIzOXwzMTc2YzQ2ZXwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes)

2024-5-1 13:44 上传

### 3.`ST`接口

把上面`ST-`开头的值传到这里，请求成功后，就算登陆成功了。

![](https://attach.52pojie.cn/forum/202405/01/134430x33nnopp524o6423.png)

image-20240428170359035.png (162.87 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5MzI0MHw3MzkyZGI0ZnwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzI0MHw3MzkyZGI0ZnwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes)

2024-5-1 13:44 上传

## 二、效果图

![](https://attach.52pojie.cn/forum/202405/01/134448s7fffi1hx16nzfh1.png)

image-20240428181150988.png (120.79 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY5MzI0OHxlYWRkYThlZHwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzI0OHxlYWRkYThlZHwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes)

2024-5-1 13:44 上传

![](https://attach.52pojie.cn/forum/202405/01/134450bw6o67whk2m4hrew.png)

image-20240428181214545.png (120.58 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY5MzI0OXwxNWJiZWU3ZnwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzI0OXwxNWJiZWU3ZnwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes)

2024-5-1 13:44 上传

## 三、扩展（实现查询个人成绩）

点查询个人成绩,进入到教务系统。

![](https://attach.52pojie.cn/forum/202405/01/134452r7c5b8ccb4scz3ce.png)

image-20240501122449805.png (188.17 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY5MzI1MHxjNmY3NzgxNnwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzI1MHxjNmY3NzgxNnwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes)

2024-5-1 13:44 上传

登录教务系统，把下面的`service`替换成上面的`url`，然后拿到`ST`开头的值。

![](https://attach.52pojie.cn/forum/202405/01/134455pt3zvcc8ez6ec44c.png)

image-20240501123817731.png (103.55 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY5MzI1MXw1ZTdiYzU0Y3wxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzI1MXw1ZTdiYzU0Y3wxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes)

2024-5-1 13:44 上传

然后请求这个接口，最后会重定向到学生成绩查询页面。

![](https://attach.52pojie.cn/forum/202405/01/134457c84xhhwuuz03zh8u.png)

image-20240501124113186.png (115.24 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY5MzI1MnwwZGFkMTc2MXwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzI1MnwwZGFkMTc2MXwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes)

2024-5-1 13:44 上传

最后调用查询成绩接口。

![](https://attach.52pojie.cn/forum/202405/01/134459fkkkevh4afwhwvxy.png)

image-20240501124320922.png (62.3 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY5MzI1M3w1NDUyMzg1MXwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzI1M3w1NDUyMzg1MXwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes)

2024-5-1 13:44 上传

效果如下

![](https://attach.52pojie.cn/forum/202405/01/134528xypgaprp0sssg5sm.png)

image-20240501132328218.png (71.87 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY5MzI1NXxjYzI0MWI1N3wxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzI1NXxjYzI0MWI1N3wxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes)

2024-5-1 13:45 上传

## 四、把成绩推送到微信

使用 [//www.pushplus.plus/](https://www.pushplus.plus/)

![](https://attach.52pojie.cn/forum/202405/01/134424zcopblz3qpjr4krz.jpg)

628df2a54c8f4afa82f70ac8bdfbb29.jpg (145.17 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5MzIzN3w2ZmE1NjVjZXwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzIzN3w2ZmE1NjVjZXwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4&nothumb=yes)

2024-5-1 13:44 上传

## 五、代码实现

![](https://static.52pojie.cn/static/image/filetype/zip.gif)

[href="forum.php?mod=attachment&aid=MjY5MzIzMnw0YWU4YjJkZnwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzIzMnw0YWU4YjJkZnwxNzkzNTcyMjYyfDE2ODA4NnwxOTE5NjM4)

(6.74 KB, 下载次数: 25)

2024-5-1 13:39 上传

点击文件名下载附件
下载积分: 吾爱币 -1 CB
