
```powershell
$source = @"
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Windows.Forms;

public class Screenshot
{
    public static List CaptureScreens()
    {
        var results = new List();
        var allScreens = Screen.AllScreens;

        foreach (Screen screen in allScreens)
        {
            try
            {
                Rectangle bounds = screen.Bounds;

                using (Bitmap bitmap = new Bitmap(bounds.Width, bounds.Height))
                {
                    using (Graphics graphics = Graphics.FromImage(bitmap))
                    {
                        graphics.CopyFromScreen(new Point(bounds.Left, bounds.Top), Point.Empty, bounds.Size);
                    }

                    results.Add((Bitmap)bitmap.Clone());
                }
            }
            catch (Exception)
            {
            }
        }

        return results;
    }
}
"@

Add-Type -TypeDefinition $source -ReferencedAssemblies System.Drawing, System.Windows.Forms

$screenshots = [Screenshot]::CaptureScreens()

for ($i = 0; $i -lt $screenshots.Count; $i++){
    $screenshot = $screenshots[$i]
    $screenshot.Save("./Display ($($i+1)).png")
    $screenshot.Dispose()
}
```

如下图所示：

![](https://attach.52pojie.cn/forum/202502/14/164531f0dlldttxb11so1e.jpg)

`IMG_1227.jpg` (48.86 KB, 下载次数: 2)

<https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1NDk2MXxhMTRmYmIyMXwxNzkzNTc2NzM5fDE2ODA4NnwyMDA2MzU0&nothumb=yes>

2025-2-14 16:45 上传

`UploadGoFile`下的`upload_file(file_path)`函数会将待上传的文件上传到`https://store1.gofile.io/contents/uploadfile`，然后获取到文件分享链接，之后程序再将文件分享链接上传发送给黑客，相关代码如下图所示：

![](https://attach.52pojie.cn/forum/202502/14/164544ltvw5srmhsvburri.png)

`code6.png` (76.08 KB, 下载次数: 1)

<https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1NDk2M3w2ZWRmNzM1NnwxNzkzNTc2NzM5fDE2ODA4NnwyMDA2MzU0&nothumb=yes>

2025-2-14 16:45 上传

`StealCommonFiles`下的函数会根据文件内容、文件大小、文件拓展名等文件特征选择性窃取文件，压缩为zip并上传发送给黑客（将国家、IP地址和文件所在驱动器号作为zip压缩包文件名），相关代码、关键词、文件拓展名，如下图所示：

![](https://attach.52pojie.cn/forum/202502/14/164555i6hlgmgg5gq8p7gm.png)

`code7.png` (287.33 KB, 下载次数: 2)

<https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1NDk2NHw2YWRjMDhlYXwxNzkzNTc2NzM5fDE2ODA4NnwyMDA2MzU0&nothumb=yes>

2025-2-14 16:45 上传

程序在启动时会直接执行`Maincookie` Main class 下的函数和`StealCommonFiles().StealFiles()`，如下图所示：

![](https://attach.52pojie.cn/forum/202502/14/164608nm7m9wpwm9z9wsmj.png)

`code8.png` (51.57 KB, 下载次数: 1)

<https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1NDk2NXw3ZDc2NGUwYXwxNzkzNTc2NzM5fDE2ODA4NnwyMDA2MzU0&nothumb=yes>

2025-2-14 16:46 上传

## 2. 附录

### （一）恶意行为者资产

GitHub账户：<https://github.com/nvslks>

相关GitHub仓库：

- <https://github.com/nvslks/g>（本文分析样本）
- <https://github.com/nvslks/h>
- <https://github.com/nvslks/f>
- <https://github.com/nvslks/b>
- <https://github.com/nvslks/a>
- <https://github.com/nvslks/c>
- <https://github.com/nvslks/e>
- <https://github.com/nvslks/d>

截至本文撰写时仍然可访问。

### （二）本文分析样本Hash (SHA-256)

- `Install_DeepSeek.exe` - `c9d815df845f5e13e8ef1142cb2dec8f97a449d639be2b38bdd75beaacff70b8`
- `Bind.exe` - `9d09a10bfa2aeb89aba5e20e88fb4fc1f56392d859d0592db66221a9f00000c4`
- `g.bat` - `25696f0af2570b4ab6013ff7dcb6995dcfaff31323ad05b268a680a2411ccf0a`
- `1.bat` - `1003d1b243a01a0a518491d0af6c67e33b8d904762b5de167b39acec05ca46f4`
- `python.py` - `f6d47d223401627eb4978b7582368854d271d6446b3137dac5ce99cb46efd886`
