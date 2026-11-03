
# [Android 原创] `SoFixerMinus`，对`SoFixer`的小小修补

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=2045934>
- 抓取时间：2026-11-01T15:42:57.322Z
- 项目地址：<https://github.com/wp00000008/SoFixerMinus>

## 1. 对`SoFixer`的修正

(1) `rel/rela`判断错误，可能将`Rela`当作`Rel`处理，导致反编译时有些函数被切割。如下：

![](https://attach.52pojie.cn/forum/202507/15/195726na1nffoaajc4rrk1.png)

(2) 其它……

## 2. 编译方法

### 64位编译方法

```
gcc *.cpp
```

### 32位编译方法

（32位未测试）

注释`config.h`中的`__SO64__`

```
gcc *.cpp
```
