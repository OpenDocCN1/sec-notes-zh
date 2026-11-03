
```
Membership["getMaxLoginNum"].implementation = function () {

    console.log(`Membership.getMaxLoginNum is called`);

    let result = this["getMaxLoginNum"]();

    console.log(`Membership.getMaxLoginNum result=${result}`);

    return 10;

};

Membership["getChannel"].implementation = function () {

    console.log(`Membership.getChannel is called`);

    let result = this["getChannel"]();

    console.log(`Membership.getChannel result=${result}`);

    return result;

};

Membership["isBeta"].implementation = function () {

    console.log(`Membership.isBeta is called`);

    let result = this["isBeta"]();

    console.log(`Membership.isBeta result=${result}`);

    return result;

};

});

}

setTimeout(main, 1000);
```

### 11. hook修改完成后，发现我们获得了永久VIP。

![](https://attach.52pojie.cn/forum/202501/23/115136f9txftkkq5ta80p3.png)

Snipaste_2025-01-23_10-18-33.png (160.27 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1MTI0MHxhMWJjYjhhMnwxNzkzNTc3NDY4fDE2ODA4NnwyMDAxOTg1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MTI0MHxhMWJjYjhhMnwxNzkzNTc3NDY4fDE2ODA4NnwyMDAxOTg1&nothumb=yes)

2025-1-23 11:51 上传

### 12. 接下来我们还是去做持久化，还是使用`xposed`写插件，关键代码如下：

[Java] 纯文本查看 复制代码
