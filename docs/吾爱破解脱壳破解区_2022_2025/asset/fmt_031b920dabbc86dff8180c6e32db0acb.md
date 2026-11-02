
# [.NET逆向] [.NET] 某宝石盒子 系列组件 许可证分析

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=1786261](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1786261)
- 抓取时间：2026-11-01T22:52:37

## 某宝石盒子 系列组件 许可证分析

### 前情提要

手头的事儿搞得差不多了，项目从 Win8.1 迁移到 UWP，Webview 升 Webview2 踩了好多坑...
之后 UWP 好像还得再迁... QAQ...
于是放松放松，再来水个帖子...
唉... 虽然感觉这个好像也达不到精华的程度 囧...

### 嘉宾介绍

还是某个和 Office 相关的系列组件，因为 Nuget 上就这类多

```
官网
aHR0cHM6Ly93d3cuZ2VtYm94c29mdHdhcmUuY29tL2J1bmRsZQ==
示例库介绍
aHR0cHM6Ly93d3cuZ2VtYm94c29mdHdhcmUuY29tL2RvY3VtZW50L2V4YW1wbGVzL2dldHRpbmctc3RhcnRlZC84MDE=
示例库文档
aHR0cHM6Ly93d3cuZ2VtYm94c29mdHdhcmUuY29tL2RvY3VtZW50L2RvY3MvZXZhbHVhdGlvbi1hbmQtbGljZW5zaW5nLmh0bWw=
```

### 准备工作

先来个能触发鉴权的 Demo。

```
GemBox.Document.ComponentInfo.SetLicense("FREE-LIMITED-KEY");
var document = new GemBox.Document.DocumentModel();
var section = new GemBox.Document.Section(document);
document.Sections.Add(section);
for (int j = 0; j 
```

会触发 超过免费页面额度的错误。

### 开工

先通过引用直接定位到 `dll`，看一眼目标函数 `SetLicense`

```
// Token: 0x06009BD1 RID: 39889 RVA: 0x002F256C File Offset: 0x002F076C
public static void SetLicense(string serialKey)
{
        try
        {
                ComponentInfo.\u0003.\u000F(Regex.Replace(serialKey, \u0003\u0018\u001A\u0003.\u0005\u0002(2011419674), string.Empty));
        }
        catch (\u0005\u0005\u0002\u0003 u0005_u0005_u0002_u)
        {
                throw new \u0002\u0005\u000F\u0002(u0005_u0005_u0002_u.Message);
        }
}
```

拿 `de4dot` 去个混淆看看。

发现字符串并没有自动解密，看来要手动解密。

解密函数是

```
// Token: 0x060025E7 RID: 9703 RVA: 0x000C5800 File Offset: 0x000C3A00
[MethodImpl(MethodImplOptions.NoInlining)]
internal static string \u0005\u0002(int \u0002)
{
        string text;
        if (\u0003\u0018\u001A\u0003.\u0002.TryGetValue(\u0002, out text))
        {
                return text;
        }
        return \u0003\u0018\u001A\u0003.\u0008\u0002(\u0002, true);
}
```

结果

![1.png](https://attach.52pojie.cn/forum/202305/16/173725et01dbgtt5gwdzwn.png)

`1.png` (35.83 KB, 下载次数: 0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNTIyNXwzOGQ5ZTA4OXwxNzkzNTczNTU0fDE2ODA4NnwxNzg2MjYx&nothumb=yes)

2023-5-16 17:37 上传

看来这里面可能有循环调用。

我们可以用反射自己手动解密

```
void decodeString(int i)
{
    Type type = AccessTools.GetTypesFromAssembly(typeof(GemBox.Document.ComponentInfo).Assembly).First(t => t.Name == "\u0003\u0018\u001A\u0003");
    string invoke = (string) AccessTools.Method(type,"\u0005\u0002", new[] {typeof(int)}).Invoke(null, new object[] {i});
    Console.WriteLine($"{i}：{invoke}");
}
new[]
{
    2011419674, // \s+
}.ToList().ForEach(decodeString);
```

也可以试着看看别的环境里的库能不能解，比如这个库默认使用的是 `netstandard2.0`，旁边还有个 `net35` 和 `net6.0-windows7.0` 的环境。

咱们解下 `net35` 吧~

发现成功... 有时候换条路其实也行的，当然，硬啃就像上面自己慢慢调试了...

咱们继续看去混淆后的函数。

![10.png](https://attach.52pojie.cn/forum/202305/16/175344c5osobzqjqolww5p.png)

`10.png` (21.77 KB, 下载次数: 0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNTIzNnwzMjUyYzVjMHwxNzkzNTczNTU0fDE2ODA4NnwxNzg2MjYx&nothumb=yes)

2023-5-16 17:53 上传

PS. 这里可以使用 `dnspy` 的功能修改字段名，方便阅读代码。

比如我们可以将 `this.string_0` 这里名称改成 `_license`，反正自己看看怎么舒服怎么来。

接着老样子，查引用，查读取

![2.png](https://attach.52pojie.cn/forum/202305/16/173732llqelewspplllpvf.png)

`2.png` (97.55 KB, 下载次数: 0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNTIyNnwzOGQ5ZTA4OXwxNzkzNTczNTU0fDE2ODA4NnwxNzg2MjYx&nothumb=yes)

2023-5-16 17:37 上传

```
// Class333
// Token: 0x060009A8 RID: 2472 RVA: 0x000DC678 File Offset: 0x000DA878
// 上层入参
//string_1 DN
//string_2 2023May09
//
public bool method_1(string string_1, string string_2, Action action_0, ref int int_1, ref int int_2, ref int int_3)
{
        if (string.IsNullOrEmpty(this._license))
        {
                return false;
        }
        if (this.method_2()) // 这里 是检查是否是 某种旧格式的 LicenseKey
        {
                throw new Exception65("The serial key is not valid. You are probably using serial key for an older version of the product. Please renew your license to continue using this version, or use an older version.");
        }
        char c = this._license[this._license.Length - 1]; // license 最后一位 是 A 或者 B
        if (c == 'A') // 所以 可用 License 有两种 A或B 结尾
        {
                return this.method_3(string_1, string_2, action_0, ref int_1, ref int_2, ref int_3);
        }
        if (c == 'B')
        {
                return this.method_4(string_1, string_2, action_0, ref int_1, ref int_2, ref int_3);
        }
        if (c != 'Y') //这里为什么是个Y？ 因为 默认的 免费Key FREE-LIMITED-KEY 结尾是Y...
        {
                this.method_7();
                return false;
        }
        return this.method_13(action_0, ref int_1, ref int_2, ref int_3);
}
// Class333
// Token: 0x060009A9 RID: 2473 RVA: 0x0003987C File Offset: 0x00037A7C
[MethodImpl((MethodImplOptions)256)]
private bool method_2()
{
        //private static readonly Regex regex_0 = new Regex("[a-zA-Z0-9]{4}-[a-zA-Z0-9]{4}-[a-zA-Z0-9]{4}-[a-zA-Z0-9]{4}");
        return Class333.regex_0.IsMatch(this._license);
}
```

然后来看 `method_3` 和 `method_4` 的逻辑

![3.png](https://attach.52pojie.cn/forum/202305/16/173735iw4x6z2ors4338o7.png)

`3.png` (35.37 KB, 下载次数: 0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNTIyN3xjNGJhZjI2ZnwxNzkzNTczNTU0fDE2ODA4NnwxNzg2MjYx&nothumb=yes)

2023-5-16 17:37 上传

```
private void method_10(string string_1) // 入参是否过期 所以 我们 
{
        DateTime dateTime = DateTime.ParseExact(string_1, "yyyyMMMdd", CultureInfo.InvariantCulture);
        if (DateTime.Now > dateTime)
        {
                throw new Exception65("The serial key has expired.");
        }
}
```

所以推测 `method_4`（`B`）是限时许可。着重看 `method_3`（`A`）即可。

![4.png](https://attach.52pojie.cn/forum/202305/16/173738e98h98zk9wswl4kt.png)

`4.png` (26.58 KB, 下载次数: 0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNTIyOHw1M2MxNTkwY3wxNzkzNTczNTU0fDE2ODA4NnwxNzg2MjYx&nothumb=yes)

2023-5-16 17:37 上传

```
private bool method_3(string string_1, string string_2, Action action_0, ref int int_1, ref int int_2, ref int int_3)
{
        if (this._license.Length
```

![5.png](https://attach.52pojie.cn/forum/202305/16/173740f9y9303i30iyi36y.png)

`5.png` (12.54 KB, 下载次数: 0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNTIyOXwwYjMwNmVkMHwxNzkzNTczNTU0fDE2ODA4NnwxNzg2MjYx&nothumb=yes)

2023-5-16 17:37 上传

![6.png](https://attach.52pojie.cn/forum/202305/16/173743f043tlr00jy54y3j.png)

`6.png` (19.46 KB, 下载次数: 0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNTIzMHw1NGEzN2U4ZXwxNzkzNTczNTU0fDE2ODA4NnwxNzg2MjYx&nothumb=yes)

2023-5-16 17:37 上传

PS. 根据黑名单可以看出，有一些特殊的前缀 `SN`、`DN`、`PN`、`AN`、`EN`，可以直观看出许可证的前18位格式

最后就剩一个
