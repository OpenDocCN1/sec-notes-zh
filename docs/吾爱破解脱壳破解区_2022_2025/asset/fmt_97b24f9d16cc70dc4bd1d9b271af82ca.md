
# [Web逆向] 关于超星学习通网页版字体加密分析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1631357
- 抓取时间: 2026-11-01T23:27:10

分析仅供学习研究，请勿使用于非法用途

最近发现超星针对题目做了字体加密，闲来无事研究了一下。

如图可以看到字体是被加密了一部分

![](https://attach.52pojie.cn/forum/202205/04/152723ss3z3khjh4g3yysz.png)

image.png (138.36 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjUxOTUwNHwzNTA0NzYyOXwxNzkzNTc1NjI4fDE2ODA4NnwxNjMxMzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTUwNHwzNTA0NzYyOXwxNzkzNTc1NjI4fDE2ODA4NnwxNjMxMzU3&nothumb=yes)

2022-5-4 15:27 上传

加密字体上右键审核元素看到 class 包含了一个`font-cxsecret`,我们看下源码

![](https://attach.52pojie.cn/forum/202205/04/153031spbuph2dpz221wk6.png)

image.png (32.58 KB, 下载次数: 6)

[href="forum.php?mod=attachment&aid=MjUxOTUwNXxlYmEyNDZlYXwxNzkzNTc1NjI4fDE2ODA4NnwxNjMxMzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTUwNXxlYmEyNDZlYXwxNzkzNTc1NjI4fDE2ODA4NnwxNjMxMzU3&nothumb=yes)

2022-5-4 15:30 上传

源码中发现字体的`base64`，我们尝试解码还原成`.ttf`格式

使用字体查看工具打开看下信息，可以直接看到这个`ttf`包含哪些字以及他的基本信息

![](https://attach.52pojie.cn/forum/202205/04/153514vifmeui8oyoi3f8i.png)

image.png (67.22 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjUxOTUwN3xmYjdjNzU3YnwxNzkzNTc1NjI4fDE2ODA4NnwxNjMxMzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTUwN3xmYjdjNzU3YnwxNzkzNTc1NjI4fDE2ODA4NnwxNjMxMzU3&nothumb=yes)

2022-5-4 15:35 上传

将`ttf`解析成`xml`()

![](https://attach.52pojie.cn/forum/202205/04/153853xrqmurr33fbzq5iz.png)

image.png (116.22 KB, 下载次数: 5)

[href="forum.php?mod=attachment&aid=MjUxOTUwOHw3NTMzYjQwZXwxNzkzNTc1NjI4fDE2ODA4NnwxNjMxMzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTUwOHw3NTMzYjQwZXwxNzkzNTc1NjI4fDE2ODA4NnwxNjMxMzU3&nothumb=yes)

2022-5-4 15:38 上传

由于之前没有研究过`ttf`所以百度了一手`ttf`文件结构，大家可以看这个([//blog.csdn.net/blueangle17/article/details/23750999" target="_blank"](https://blog.csdn.net/blueangle17/article/details/23750999:rel="nofollow noopener" href="https))

![](https://attach.52pojie.cn/forum/202205/04/154213f0xyppue7v5r5rpp.png)

image.png (230.4 KB, 下载次数: 5)

[href="forum.php?mod=attachment&aid=MjUxOTUwOXw3MTFkMmJkNHwxNzkzNTc1NjI4fDE2ODA4NnwxNjMxMzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTUwOXw3MTFkMmJkNHwxNzkzNTc1NjI4fDE2ODA4NnwxNjMxMzU3&nothumb=yes)

2022-5-4 15:42 上传

如图映射结果是一致的说明不是修改映射关系来加密字体的

如果映射结果没有变化的话那可能是修改了图元数据，我尝试将他的unicode还原成中文发现确实是加密后的字体，基本上可以理清楚超星的加密是修改了此字体图元数据，所以显示成未加密的字。

我们接下来尝试解析并还原加密文字

由于超星每个网页的`ttf`数据都是不同的，并且只有部分字体加密，所以我们需要先找到一份完整的`ttf`文件,根据字体查看工具发现是用的字体是

`SourceHanSansCN-Normal Version 1.000;PS 1;hotconv 1.0.78;makeotf.lib2.5.61930`

直接百度找一手源文件，然后解析成`xml`查看图元信息是否一致

超星加密的`ttf`某个字图元数据

```asm

      
      
        
        
        
        
      

        
        
        
        
      
```

百度下载的字体`ttf`图元数据

```asm

```

我们可以看到 `uni6408`是用的是 官方`uni4E89`的图元数据

查看对应文字 【搈】=>【争】

没毛病，那这样我们基本上已经理清楚了超星的加密方式

然后就是如果还原字体了，我的思路是用原版的`ttf`的图元数据做一个字典，然后反推加密的字体

```python
def yh_mapping():
    keylist={}
    with open('yh.xml') as f_baes:
        xml_base = f_baes.read()
    s_base = xml_base.split("")[:-1]
    for i in range(0, len(s_base)):
        lists_base = []
        contour = re.findall('', s_base[i])
        name=re.findall('name="(.*?)"',s_base[i])[0]
        for j in range(0, len(contour)):
            x = re.findall('x=\"(.*?)\"', contour[j])
            y = re.findall('y=\"(.*?)\"', contour[j])
            on = re.findall('on=\"(.*?)\"', contour[j])
            lists_base.append(x[0] + y[0] + on[0])
        keylist[name]=hashlib.md5("".join(lists_base).encode('utf8')).hexdigest()
    print(keylist)
    with open("map.txt", "w") as f:
        f.write(json.dumps(keylist))
```

这个代码的作用是将原版`ttf`的字体图元数据拼接并且`md5`生成唯一标志,然后与其unicode编码相对应,然后我们就得到了一个原版的编码=>唯一标志的`json`文件

![](https://attach.52pojie.cn/forum/202205/04/160336vd555pyy68585n58.png)

image.png (326.17 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjUxOTUxMHxmOWIwNzI5NHwxNzkzNTc1NjI4fDE2ODA4NnwxNjMxMzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTUxMHxmOWIwNzI5NHwxNzkzNTc1NjI4fDE2ODA4NnwxNjMxMzU3&nothumb=yes)

2022-5-4 16:03 上传

接下来处理超星的`ttf`,将他的图元数据以同样的方法处理,然后根据图元数据生成的唯一标志比较，将原版的unicode对应进去

```python

def font_mapping(b64_code):

    with open("map.txt", "r", encoding="utf-8") as f:

        keys=json.loads(f.read())

    kes_sign_list={}

    for i in keys:

        kes_sign_list[keys[i]]=i

    lists_base_all = []

    keylist={}

    text_font = TTFont(BytesIO(base64.decodebytes(b64_code.encode())))

    text_font.save("font.ttf")

    text_font.saveXML("font.xml")

    with open('font.xml') as f_baes:

        xml_base = f_baes.read()

    s_base = xml_base.split("")[:-1]

    for i in range(0, len(s_base)):

        lists_base = []

        contour = re.findall('', s_base[i])

        name=re.findall('name="(.*?)"',s_base[i])[0]

        for j in range(0, len(contour)):

            x = re.findall('x=\"(.*?)\"', contour[j])

            y = re.findall('y=\"(.*?)\"', contour[j])

            on = re.findall('on=\"(.*?)\"', contour[j])

            lists_base.append(x[0] + y[0] + on[0])

        keylist[name]=hashlib.md5("".join(lists_base).encode('utf8')).hexdigest()

        lists_base_all.append(lists_base)

    keys_list={}

    for i in keylist:

        keys_list[i]=kes_sign_list[keylist[i]].replace("uni","\\u").encode("utf-8").decode('unicode_escape')

    return keys_list
```

函数传入超星的加密`base64`,返回加密`ttf` unicode所对应的解密字体,效果如图所示

![](https://attach.52pojie.cn/forum/202205/04/161050hrnm8zwymg0s55m5.png)

image.png (55.28 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUxOTUxMXwzMzdlMzI4NnwxNzkzNTc1NjI4fDE2ODA4NnwxNjMxMzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTUxMXwzMzdlMzI4NnwxNzkzNTc1NjI4fDE2ODA4NnwxNjMxMzU3&nothumb=yes)

2022-5-4 16:10 上传

最后将加密数据带入计算得出解密后的文本

![](https://attach.52pojie.cn/forum/202205/04/161148i0jsjjo58lsj5zi9.png)

image.png (169.63 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjUxOTUxMnw1NmVlNGE3OXwxNzkzNTc1NjI4fDE2ODA4NnwxNjMxMzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTUxMnw1NmVlNGE3OXwxNzkzNTc1NjI4fDE2ODA4NnwxNjMxMzU3&nothumb=yes)

2022-5-4 16:11 上传

总结: 超星所使用的字体加密 通过修改字体的图元数据使A字的字形变成了B字的字形，虽然他看上去是B字但是复制的时候还是A字

相关代码，文件已放附件，由于原版字体文件放入后超过3m所以有需要得可以自行百度下载

编写较乱，各位看官还请多见谅
