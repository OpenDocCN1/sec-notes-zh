
# [原创] 采用USRP SDR和URH进行无线开关发射的信号解调和协议数据包解密

- 原帖: <https://www.52pojie.cn/forum.php?mod=viewthread&tid=1787578>
- 抓取时间: 2026-11-01T22:52:28

## 研究目标与产品

我们今天的研究目标是一个INSMA 433Mhz 无线射频开关。通过特定的工具研究其通讯协议，给出采用URH工具研究和SDR进行无线信号的手段。产品在网站宣传的图如下：

![](https://attach.52pojie.cn/forum/202305/19/183858ju363f8n68688c6n.png)

`00_product.png` (448.23 KB, 下载次数: 0)  
[forum.php?mod=attachment&aid=MjYxNjEzNXxiMmRlMjc5ZXwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjEzNXxiMmRlMjc5ZXwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes)  
2023-5-19 18:38 上传

![](https://attach.52pojie.cn/forum/202305/19/183901p3d0ikx918rf96d3.png)

`01_product.png` (266.42 KB, 下载次数: 0)  
[forum.php?mod=attachment&aid=MjYxNjEzNnw5YzFhY2EwM3wxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjEzNnw5YzFhY2EwM3wxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes)  
2023-5-19 18:39 上传

## 采样设置

采用USRP B210和URH工具进行采样。如下图所示，运行URH，选择`Record Signal`菜单，设置采用的中心频率和`subdev`参数等，运行后保存4份采样数据。

![](https://attach.52pojie.cn/forum/202305/19/183934ay33swsn98qxgnx3.png)

`02_record_signal.png` (29.11 KB, 下载次数: 0)  
[forum.php?mod=attachment&aid=MjYxNjEzN3w3OGRiYTRmZXwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjEzN3w3OGRiYTRmZXwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes)  
2023-5-19 18:39 上传

![](https://attach.52pojie.cn/forum/202305/19/183937a2lh0ls220u27lpq.png)

`03_RECORD.png` (37.32 KB, 下载次数: 0)  
[forum.php?mod=attachment&aid=MjYxNjEzOHxjYTZiZTE1M3wxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjEzOHxjYTZiZTE1M3wxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes)  
2023-5-19 18:39 上传

## 打开采样文件

采用`open`菜单，全选4份采样文件，打开文件。如下图所示，设置调制方式为ASK。

![](https://attach.52pojie.cn/forum/202305/19/184013gloonot08rrostol.png)

`04_open_files.png` (94.58 KB, 下载次数: 0)  
[forum.php?mod=attachment&aid=MjYxNjEzOXw5Njg0MDZiN3wxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjEzOXw5Njg0MDZiN3wxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes)  
2023-5-19 18:40 上传

## ASK调制与关键参数

对于遥控器发射的信号，进行一个简单的科普说明。对于ASK调制方式，bit 1代表了开，bit 0代表了关。从发射器的角度，关的状态其实就是功率电平的低状态。为了对于这个无线模式进行探索，我们需要知道如下的几个参数：1、对于遥控器的每个按钮，是整个数据包的那些bit来标识的？2、每个Bit的宽度是多少（与之相对应的概念就是符号率）？3、无线传输的中心频率是多少？对于第3个问题，产品说明书已经告知。对于第2个问题，URH工具可以提供细节如下图所示：以第一个采用文件为例，按住CTRL键+鼠标滚轮，放大波形，直至看到正弦波的样子。然后选中下面对应的一个bit 1。对应的波形也会被蓝色底色标记。这个蓝色底色的宽度就是对应的Bit宽度。

![](https://attach.52pojie.cn/forum/202305/19/184053jfjjptjjpj88vlr1.png)

`05_detail_signal.png` (131.73 KB, 下载次数: 0)  
[forum.php?mod=attachment&aid=MjYxNjE0MHxmZDBiNGQ5MXwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjE0MHxmZDBiNGQ5MXwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes)  
2023-5-19 18:40 上传

![](https://attach.52pojie.cn/forum/202305/19/184057bzrs7oxgi6ggg6ho.png)

`05_detail_signal_2.png` (87.2 KB, 下载次数: 0)  
[forum.php?mod=attachment&aid=MjYxNjE0MXwyYWFjNTdiNXwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjE0MXwyYWFjNTdiNXwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes)  
2023-5-19 18:40 上传

## 测量Bit宽度与Demodulate视图

为了更清晰的展示，可以采用URH的`Demodulate`视图，这个视图给出了类似数字电平的矩形图来表示bit位。如下图所示。

![](https://attach.52pojie.cn/forum/202305/19/184128hxyntn23y273yt8n.png)

`06_bitwidth.png` (90.85 KB, 下载次数: 0)  
[forum.php?mod=attachment&aid=MjYxNjE0MnxmMzYxNjNjNHwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjE0MnxmMzYxNjNjNHwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes)  
2023-5-19 18:41 上传

我们选择一个相对较窄的矩形波，拖动蓝色矩形的宽度，使得其正好覆盖整个矩形。如下图所示，可以看出来此时他的宽度是481.0us。我们可以认为是电平bit1的宽度大概至少要达到481us。为了更清晰的展示出协议数据包的情况，二进制显然是不太合适的，如下图所示选择`Hex`十六进制格式：

![](https://attach.52pojie.cn/forum/202305/19/184226ocmh9zarm9a1a532.png)

`07_hexview.png` (91.25 KB, 下载次数: 1)  
[forum.php?mod=attachment&aid=MjYxNjE0M3xiODRmMzIzNXwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjE0M3xiODRmMzIzNXwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes)  
2023-5-19 18:42 上传

## 转换为Hex视图与自动检测参数

把四组数据都转换为`Hex view`，虽然可读性有些强，但是数据还是比较乱的。这是因为URH对于解调参数的预估可能受到了一些因素的影响，我们可以先尝试点击`Auto DetectParameters`按钮，经过点击之后，我们分别浏览4组数据如下：

![](https://attach.52pojie.cn/forum/202305/19/184333ia5jzadejgv9rwwi.png)

`08_hexview2.png` (94.35 KB, 下载次数: 0)  
[forum.php?mod=attachment&aid=MjYxNjE0NHxlM2FmZGYxNHwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjE0NHxlM2FmZGYxNHwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes)  
2023-5-19 18:43 上传

![](https://attach.52pojie.cn/forum/202305/19/184337bu55ggwjzr5qgu88.png)

`09_all_hex_view.png` (89 KB, 下载次数: 0)  
[forum.php?mod=attachment&aid=MjYxNjE0NXw4Y2I3MjIzOHwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjE0NXw4Y2I3MjIzOHwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes)  
2023-5-19 18:43 上传

![](https://attach.52pojie.cn/forum/202305/19/184340fbua1y1lm3ay9j9k.png)

`10_autodetect.png` (91.5 KB, 下载次数: 0)  
[forum.php?mod=attachment&aid=MjYxNjE0Nnw4ZjYyNmJmY3wxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjE0Nnw4ZjYyNmJmY3wxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes)  
2023-5-19 18:43 上传

## 分析按钮D的数据包

我们可以看到最后一组的数据一致性最好，也就是按下按钮D时的那组数据。我们可以几乎肯定的是，按下按钮D时的协议数据包为`ee8e8e8e8e8e8e8eee8888888`。可以看到这个协议数据包数字只有两个`8`和`e`。那么对应到解调波形上是怎样的呢？我们可以看到`e`代表了一个宽脉冲，`8`代表了一个短脉冲。

![](https://attach.52pojie.cn/forum/202305/19/184505nlup64xsxxhur4xh.png)

`11_view_data.png` (91.91 KB, 下载次数: 0)  
[forum.php?mod=attachment&aid=MjYxNjE0N3w4YTYzZjljM3wxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjE0N3w4YTYzZjljM3wxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes)  
2023-5-19 18:45 上传

![](https://attach.52pojie.cn/forum/202305/19/184508q28ojyro7ouuiw2g.png)

`12_pulse_width.png` (89.88 KB, 下载次数: 0)  
[forum.php?mod=attachment&aid=MjYxNjE0OHw1MWEwZDMwYnwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjE0OHw1MWEwZDMwYnwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes)  
2023-5-19 18:45 上传

## 手动微调解调参数

这个遥控器通过这种脉冲图样的调制信息来进行发射信息。那么A、B、C按钮我们可以先假设也都是这样的一个图样。那么为什么URH解调出来的信息会有些问题呢？我个人感觉还是因为干扰或者相应的参数不合适导致的，URH自动评估出的参数是存在一些问题的。那么我们微调一下左边的参数，进行手动微调。把`Noise`统一调节为0.0014，`samples/symbol`为500，`Errortolerance`为6。并且把图形进行适当的缩小。如下图所示。

![](https://attach.52pojie.cn/forum/202305/19/184534xlx8mlexh9hm68c8.png)

`13_adjust.png` (95.72 KB, 下载次数: 1)  
[forum.php?mod=attachment&aid=MjYxNjE0OXxjNDc2NmE2OXwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjE0OXxjNDc2NmE2OXwxNzkzNTczNTQ2fDE2ODA4NnwxNzg3NTc4&nothumb=yes)  
2023-5-19 18:45 上传

## 按钮A数据

此时我们把`button A`对应数据拷贝出来：

```
1d1d1d1d222223ba [Pause: 14476samples]
ee8e8e8e8e8e9d1d222223ba [Pause: 14725 samples]
ee8e9d1d3a3a3a3a222223ba [Pause: 14490 samples]
ee8e8e8e8e8e9d3a222223ba [Pause: 14745 samples]
ee8e8e8e8e8e8e9d111111dd [Pause: 14751 samples]
ee8e8e8e8e8e8e8e888893ba [Pause: 14754 samples]
ee8e8e8e8e8e8e8e888888ee8 [Pause: 14753samples]
ee8e8e8e8e8e8e8e889111dd [Pause: 14702 samples]
ee8e8e8e8e8e8e8e888888ee8 [Pause: 14704samples]
ee8e8e8e8e8e8e8e888888ee8 [Pause: 14756samples]
ee8e8e8e8e8e8e8e888888ee8 [Pause: 14772samples]
ee8e8e8e8e8e8e8e888913ba [Pause: 14769 samples]
ee8e8e8e8e8e8e8e888888ee8 [Pause: 14777samples]
ee8e8e8e8e8e8e8e888888ee8 [Pause: 14775samples]
ee8e8e8e8e8e8e8e911113ba [Pause: 14781 samples]
ee8e8e8e8e8e8e8e888891dd [Pause: 14586 samples]
```
