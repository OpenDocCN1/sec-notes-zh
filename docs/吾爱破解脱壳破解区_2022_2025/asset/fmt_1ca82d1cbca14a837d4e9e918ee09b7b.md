
```
// createVM1(3488, length=1, paramsCount=2, param=[R67,R43]);

    R139 = R95.map((ele) => {

        return R43[0][ele];

    });

    R8 = R139.join("");

    R153 = [16, 1, 32, 12, 19, 27, 8, 5];

    R186 = R153.map((ele) => {

        return R43[0][ele];

    });

    R162 = R186.join("");

    R87 = [89, 39, 179, 150, 218, 82, 58, 252, 177, 52, 186, 123, 120, 64, 242, 133, 143, 161, 121, 179];

    R48 = [];

    R181 = 0;

    while (R181 > 2] + R116[R48[R61 * 3] & 3 > 4] + R116[R48[R61 * 3 + 1] & 15 > 6] + R116[R48[R61 * 3 + 2] & 63];

        R61++;

    }

    R80 += R116[R48[18] >> 2] + R116[R48[18] & 3 > 4] + R116[R48[19] & 15
```

可以看到`sign`由三部分构成

### 四、下步改进方向

本方法只是去除了最简单的花指令，然后生成控制流图，最后手动优化得到js代码，属于半自动半手动

下一步将继续学习中间代码优化相关知识，进行常量折叠、常量传播等优化，优化完毕后将控制流图转换成对应的js代码

### 参考资料

[//jixun.uk/posts/2024/qqmusic-zzc-sign/](https://jixun.uk/posts/2024/qqmusic-zzc-sign/)

[//www.resourch.com/archives/129.html](https://www.resourch.com/archives/129.html)
