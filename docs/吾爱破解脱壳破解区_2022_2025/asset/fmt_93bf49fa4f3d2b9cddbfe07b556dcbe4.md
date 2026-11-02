
# [.NET逆向] .Net 爆破之 MD Token 预爆 + 运气【插件类完结篇】

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1779798
- 抓取时间：2026-11-01T22:53:19

## .Net 爆破篇 之 MD Token 预爆 + 运气【插件类完结】

### 前情提要

我近期发的东西，尽可能每个帖子都多少能有些解决问题的思路，并且应该能从中学到点不一样的东西。而且大家看了不少 Hook 和逆向算法，还有反射运用，是时候换点口味了。

那么这期就来看看回归初心，直接爆破吧。有时候看不懂其实也没啥，运气好就行。

比如这篇就是传统爆破文，知识点是 Token 定位。

### 题外话

我个人而言对 Excel 插件和 VBA 插件之类的使用上没什么兴趣，平常工作中用不上。

虽然近期发了一些帖子，但没有针对这个行业的意思，只是他们都刚好能拿来拓展一些知识点和提高定位问题的能力。

这期应该也是“国内”的最后一期，以一个不是很完美的爆破教程收尾。

至于问为什么看着我像是都在 Excel 插件，因为最近刚好看的是这些，而且这些我之前没接触过。

说到国外的库，可以参考：

[https://www.52pojie.cn/thread-1768897-1-1.html](https://www.52pojie.cn/thread-1768897-1-1.html)

里面提到都是咱已经研究过的，以我目前阅读理解能力 + 工具能完成破解的组件。（感觉搞 .Net 相关的应该都能搞定这些）

有兴趣的可以留言，以后可以拿出来讲解一下。

好了，回归正题！

### 本期主题

爆破！不用多解释。

MD Token 可以理解成 .Net 的元数据标识。

这里提及是可以拿它在 .Net 爆破中做一个预演，预爆。

### 本期嘉宾

反正某人说版本已经很多了，不差这一个，那就安排最新版练手。

```
aHR0cHM6Ly92YmF5eWRzLmNvbS8=
```

原版地址：[https://pan.baidu.com/s/1jCh4BsE8D2rSxB34yoyyYA&pwd=aaav](https://pan.baidu.com/s/1jCh4BsE8D2rSxB34yoyyYA&pwd=aaav)

备份地址：[https://pan.baidu.com/s/1cKMLDc3zXIFUAnMr3ZODPg?pwd=52pj](https://pan.baidu.com/s/1cKMLDc3zXIFUAnMr3ZODPg?pwd=52pj)

### 准备工作

下载后打开查看。功能正常，然后点击授权信息，会有免费版的提示。

![1.png](https://attach.52pojie.cn/forum/202304/28/181320s0caheqq5000t0eh.png)

1.png（43.5 KB，下载次数：2）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTEyMnwzYzkyOThiY3wxNzkzNTczNTk3fDE2ODA4NnwxNzc5Nzk4&nothumb=yes)

2023-4-28 18:13 上传

我们先 将 DLL 备份一份。然后拖到 `dnspy` 看是否能正常加载。

为了验证是否能进行爆破，我们需要先做一个操作，就是直接不改任何代码，但是对 DLL 进行一个保存操作，确保能正常执行。我们通过 文件-保存模块：

![2.png](https://attach.52pojie.cn/forum/202304/28/181331a60pk76qsickjmum.png)

2.png（100.2 KB，下载次数：3）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTEyM3xhZGJkZTcwOXwxNzkzNTczNTk3fDE2ODA4NnwxNzc5Nzk4&nothumb=yes)

2023-4-28 18:13 上传

切到 MD 写入选项：

![3.png](https://attach.52pojie.cn/forum/202304/28/181333ljjkjnj0sbck1wys.png)

3.png（111.27 KB，下载次数：4）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTEyNHxlNjdmMTI2NHwxNzkzNTczNTk3fDE2ODA4NnwxNzc5Nzk4&nothumb=yes)

2023-4-28 18:13 上传

把红框位置的几项全部勾上，然后确定开始写 DLL。写完后打开 Excel-VBA 编辑器，看插件还能否正常加载。

如果它还在，恭喜你，这个插件是可以尝试爆破的。

当然，如果出现失败的情况别急，你可以尝试将文件的最后修改时间和创建时间什么的，改得和之前备份的文件一样。毕竟有时候鸡贼的作者可能会检测一下文件的变动时间，来判断你是不是对 DLL 进行了一个修改（比如 E 灵试用版）。

如果直接就不能保存，保存失败，那么抱歉，这个 DLL 因为壳的原因不能进行爆破。

既然确认了这个插件是能被“修改”的，那么我们开工。

### 开始分析

我们先搜一下关键词，比如注册、激活、免费。发现一个结果都没有，那么肯定做了字符串保护。

那么我们用 `de4dot` 进行一个反混淆的处理，一般都能自动处理，但是这次讲一下怎么手动指定。

![4.png](https://attach.52pojie.cn/forum/202304/28/181336s9zyhc69cz5cmf99.png)

4.png（106.86 KB，下载次数：4）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTEyNXwxMTQwMmExZnwxNzkzNTczNTk3fDE2ODA4NnwxNzc5Nzk4&nothumb=yes)

2023-4-28 18:13 上传

上图红框的就是常见的字符串加密之后的样子。我们导航过去之后能看到：

![5.png](https://attach.52pojie.cn/forum/202304/28/181338vjjdlljdejd459v5.png)

5.png（18.89 KB，下载次数：2）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTEyNnw2YWUxNTBiOXwxNzkzNTczNTk3fDE2ODA4NnwxNzc5Nzk4&nothumb=yes)

2023-4-28 18:13 上传

函数上方有一个 Token，这个 `dnspy` 里面被称为 `MDToken`，一般函数、字段什么的都有这个 Token。

如果保存 DLL 或者用 `de4dot` 去混淆脱壳时附加了 `--preserve-tokens` 指令，那么脱壳后的 DLL 对应函数会保持脱壳前相同的 `MDToken`。

是不是有了一个大胆的想法？咱们先去反混淆后的 DLL 里面定位关键点，然后通过 Token 在未混淆的原始 DLL 里面进行爆破操作。尽可能降低对 DLL 造成的改动，避免直接使用脱壳后的 DLL 而产生一些奇怪的问题。

那么开始吧。

我这里用的是 `de4dot reactor 6.7`。

这里提一嘴，有的 `de4dot` 就算加了保持 Token 的指令，也可能出现脱壳后前后 Token 不一致的情况。这个自行选择好一点的版本吧。

`de4dot` 加上两个指令：

```
// 脱壳后保持 Token
--preserve-tokens

// 字符串解密
--strtyp delegate --strtok 字符串解密函数的 Token，图中是 0x06000991
```

```
// 完整命令
de4dot "D:\Excel880\VBAxxxxxxPro\VBACodexxxxPro.dll" --preserve-tokens --strtyp delegate --strtok 0x06000991
```

然后在脱壳后的 DLL 里面搜一下关键词：注册、激活、免费、卸载等。

![6.png](https://attach.52pojie.cn/forum/202304/28/181340yx44bxjj4fx347gz.png)

6.png（42.17 KB，下载次数：3）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTEyN3xkMDJjNDIzMXwxNzkzNTczNTk3fDE2ODA4NnwxNzc5Nzk4&nothumb=yes)

2023-4-28 18:13 上传

![7.png](https://attach.52pojie.cn/forum/202304/28/181342pe60kdxkqxkepbcc.png)

7.png（40.99 KB，下载次数：4）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTEyOHxhODMxNTJmZHwxNzkzNTczNTk3fDE2ODA4NnwxNzc5Nzk4&nothumb=yes)

2023-4-28 18:13 上传

可以看到，好像明面上的逻辑都和 `Class113` 有关。大概率它是一个授权类。

再看一眼那个枚举：

![8.png](https://attach.52pojie.cn/forum/202304/28/181345cwz0e81c11806jjn.png)

8.png（16.13 KB，下载次数：3）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTEyOXxjMzU0NWU0OHwxNzkzNTczNTk3fDE2ODA4NnwxNzc5Nzk4&nothumb=yes)

2023-4-28 18:13 上传

明显是个常见的鉴权枚举。那么爆破思路就有了。

```
// Class113
// Token: 0x06000764 RID: 1892 RVA: 0x000062C2 File Offset: 0x000044C2
internal bool method_9()
{
        return this.bool_0; //一个疑似过期判断的，强制改成 false 不过期
}

// Class113
// Token: 0x06000761 RID: 1889 RVA: 0x000062B1 File Offset: 0x000044B1
internal Class49.Enum1 method_6()
{
        return this.enum1_0;//枚举这里爆破掉 强制返回一个非免费枚举
}
```

再多瞅一眼这个授权类：

![9.png](https://attach.52pojie.cn/forum/202304/28/181347w7lg774aao7n5id8.png)

9.png（149.05 KB，下载次数：3）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTEzMHxlYjUxNmQyYnwxNzkzNTczNTk3fDE2ODA4NnwxNzc5Nzk4&nothumb=yes)

2023-4-28 18:13 上传

还返回两个日期。

其中还看到了 `vm`，得都用上 `vm` 了，要不 `Shift+Delete` 吧？或者咱们试试呗，至少能改的那个咱们给他改了呗……
