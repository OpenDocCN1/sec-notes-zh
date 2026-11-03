
# [Android 脱壳] 鲨鱼记账 oppo加固脱修+解锁会员

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2049364
- 抓取时间: 2026-11-01T15:42:23.852Z

下载后提取安装包 是oppo加固

![](https://attach.52pojie.cn/forum/202507/29/074853gpx6c5p2baqr8zzq.jpg)

## 教程开始

先[脱壳](https://www.52pojie.cn/forum-5-1.html)，然后将 `dex` 修复，回填

![](https://attach.52pojie.cn/forum/202507/29/074943akznnnxyv7spffvy.jpg)

### 删除加固特征

`aeests` 里的 `classes` 图片长按，打开方式，编辑文本，将内容清空

![](https://attach.52pojie.cn/forum/202507/29/075030ldutct7ztaa65wr6.jpg)

![](https://attach.52pojie.cn/forum/202507/29/075039kow95o6hwfwhw3iw.jpg)

进入 `lib` 文件夹

删除特征

### 修改入口

![](https://attach.52pojie.cn/forum/202507/29/075158fnnmzol8odz3nn8t.jpg)

mt 有 bug，用 np

![](https://attach.52pojie.cn/forum/202507/29/075247sxovuxvk7oolx3ki.jpg)

![](https://attach.52pojie.cn/forum/202507/29/075256pgmgwlbbsqibz13y.jpg)

### 普通去签

![](https://attach.52pojie.cn/forum/202507/29/075323g3v818o1us9oy8qy.jpg)

成功

![](https://attach.52pojie.cn/forum/202507/29/075348zr0th14yirhfvrbv.jpg)

### 修改会员

`dex` 过滤会员

![](https://attach.52pojie.cn/forum/202507/29/075409abt4d1569bdv1zt6.jpg)

跳转搜索

![](https://attach.52pojie.cn/forum/202507/29/075431u33i2mbz3umb56d2.jpg)

高亮部分方法名全部赋值 `true`

方法名 `a` 赋值到期时间

![](https://attach.52pojie.cn/forum/202507/29/075451iq2giy4ml24q206z.jpg)

![](https://attach.52pojie.cn/forum/202507/29/075507k5jzigefoorehqhh.jpg)

![](https://attach.52pojie.cn/forum/202507/29/075516cq02pnvpihlpkp22.jpg)

![](https://attach.52pojie.cn/forum/202507/29/075526el3hhhoqi9d211d9.jpg)

![](https://attach.52pojie.cn/forum/202507/29/075536jk77sze9g22g0ueg.jpg)

![](https://attach.52pojie.cn/forum/202507/29/075604pntdlry6ztt8vjjn.jpg)

修改成功

### AI 代码解释

- VIP功能可用时（`p0()` 返回 `true`）：
  - 检查用户是否为会员（`i.w()`），若为会员则显示有效期（`i.a()`）或过期状态（`i.b()`）
  - 非会员时提示“开通会员，关闭广告”
- VIP功能不可用时（`p0()` 返回 `false`）：
  - 若用户已开通会员但未激活（`q0()` 为 `true`），调用 `a0()` 返回特定字符串。
  - 否则根据会员状态显示不同信息：
    - 已过期会员：显示过期日期（`i.f0()`）
    - 有效会员：显示剩余天数（`i.g0()`）
    - 非会员：提示“开通VIP，畅享高级功能”

### 脱修包及修改包链接

https://www.123865.com/s/uMoTjv-c59V

自己动手丰衣足食

![](https://static.52pojie.cn/static/image/smiley/default/4.gif)
