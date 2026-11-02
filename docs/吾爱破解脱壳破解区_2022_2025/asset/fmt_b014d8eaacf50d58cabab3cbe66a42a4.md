
接着是 `new Func(b.c)`

```
//b.c
internal sh c()
{
    return this.a.a(this.b);
}
//this.a.a(this.b);
private sh a(si A_0)
{
    string text = A_0.a(this.b.at8()); //还是验证 Prd 里面 是否包含 WU5D
    string text2 = A_0.e(); //注册码 后半段
    //如果 Evl 为true 就是 Evaluation 模式 否则 Licensed，反正咱们没填
    LicenseMode licenseMode = ((A_0.a() == null) ? LicenseMode.Unlicensed : (A_0.a().j() ? LicenseMode.Evaluation : LicenseMode.Licensed));
    bool flag;
    int? num = A_0.a(out flag); //过期时间 没填
    sf.a a = new sf.a();
    a.a(text);
    a.b(text2);
    a.a(licenseMode);
    a.a(flag);
    a.a(num);
    a.a(A_0.d() ?? new string[0]);
    sf.a a2 = a;
    if (A_0.a().a() != null)
    {
        a2.a(A_0.a().a()); //Anl.dsr 反正没填
    }
    return a2;
}
```

终于又到之前 `e` 那个回调了。。。

```
//上面的 this.e = A_3 那个回调
private void a(bool A_0, Func A_1)
{
    this.c = A_1();

    if (!A_0) //因为 A_0 是 true 所以 不走这些
    {
        if (this.c.y0() == LicenseMode.Unlicensed)
        {
            this.c = null;
            return;
        }
        if (string.IsNullOrEmpty(this.c.y3()))
        {
            this.c = null;
            this.f = true;
            return;
        }
        if (this.c.y1())
        {
            this.e = new bool?(true);
            return;
        }
        this.c = null;
        return;
    }
    else
    {
        if (this.c.y2() != null) //过期时间 实际咱们没有 所以 跳过
        {
            this.d = DateTime.Today.AddDays((double)this.c.y2().Value);
            this.e = null;
            return;
        }
        this.e = new bool?(false); // 看来这个为 false 反而是真解
        return;
    }
}
```

## 4> 开工

分析完了 可以开始动工了

```
internal static string GetLicense(Type t)
{
    string name = "52pojie";
    var temp = new
    {
        S = Convert.ToBase64String(Encoding.UTF8.GetBytes("0")),
        D = new
        {
            Id = name,
            Prd = new[]
            {
                new {N = "WU5D", C = "WU5D"}
            },
            Crt = DateTime.Now.ToString("yyyyMMdd hhmmss", CultureInfo.InvariantCulture)
        }
    };
    string json = JsonConvert.SerializeObject(temp);
    //还记得之前那个盲猜的 encode吗？调用它试试~
    string lic = (string)t.Assembly.GetType("GrapeCity.Documents.Excel.sp").GetMethod("g").Invoke(null, new object?[] { json }); ;
    return $"{name}#A0{lic}";
}
```

### Hook 部分

![](https://attach.52pojie.cn/forum/202304/10/134210gguxme7aed7hm72m.png)

`10.png` (110.04 KB, 下载次数: 1)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNjI3OHwxMDcxOTg1MXwxNzkzNTczODIwfDE2ODA4NnwxNzcxOTMy&nothumb=yes)

2023-4-10 13:42 上传

## 4> 结果

![](https://attach.52pojie.cn/forum/202304/10/134213tw1htj1q6bywwb4q.png)

`11.png` (51.31 KB, 下载次数: 1)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNjI3OXwwYmViMzBjMnwxNzkzNTczODIwfDE2ODA4NnwxNzcxOTMy&nothumb=yes)

2023-4-10 13:42 上传

![](https://attach.52pojie.cn/forum/202304/10/134215v8zscksff7lkmz1d.png)

`12.png` (59.48 KB, 下载次数: 1)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNjI4MHxkZjBhZmU0OHwxNzkzNTczODIwfDE2ODA4NnwxNzcxOTMy&nothumb=yes)

2023-4-10 13:42 上传

完美收工~

## 4> 注意事项

仅限于学习交流，请勿用于商业或非法用途

## 4> 附录

有兴趣的同学可以解析内部注册码 能少走一些弯路 (＾Ｕ＾)ノ~ＹＯ
