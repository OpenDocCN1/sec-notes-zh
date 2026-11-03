
## 2. 生成字符串型操作码

![](https://attach.52pojie.cn/forum/202506/22/220250laxmfaghrf937zm0.png)

image.png（75.77 KB，下载次数：1）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDYxN3xkODFkZTNmOXwxNzkzNTcwODY2fDE2ODA4NnwyMDQwNzg5&nothumb=yes)

操作码 304  
2025-6-22 22:02 上传

操作码 304 生成字符串（调试的时候忘了截图！！）

可以看出来操作数是基于操作码计算得到的。

## 3. 函数调用型操作码

![](https://attach.52pojie.cn/forum/202506/22/220549hwl1l2vw261dq4bn.png)

image.png（84.5 KB，下载次数：1）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDYyN3wxOWMyYjQzOXwxNzkzNTcwODY2fDE2ODA4NnwyMDQwNzg5&nothumb=yes)

操作码 302  
2025-6-22 22:05 上传

操作码 302 进行函数调用。

## 4. 属性访问型操作码

![](https://attach.52pojie.cn/forum/202506/22/220805h9u1zzcu7qq9ea9p.png)

image.png（16.57 KB，下载次数：1）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDYyOHw0MThmYjk3OXwxNzkzNTcwODY2fDE2ODA4NnwyMDQwNzg5&nothumb=yes)

操作码 495  
2025-6-22 22:08 上传

操作码 495 访问浏览器对象下的属性。

另外：

- **操作码 122**：生成 `eval` 函数调用。
- **操作码 149、220、446**：处理数组操作（增大数组 `register_280`）。
- **操作码 265、495**：对象属性赋值。
- **操作码 319**：生成数组。
- **操作码 477**：创建新对象。

## 五、编写反编译器

分析完了，开始写反编译器，流程如下：

1. 初始化节点

   ![](https://attach.52pojie.cn/forum/202506/22/185343bmicf8iy6mcmg6zf.png)

   image.png（28.55 KB，下载次数：1）

   [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDU4NHw1YjY5N2Q0ZnwxNzkzNTcwODY2fDE2ODA4NnwyMDQwNzg5&nothumb=yes)

   2025-6-22 18:53 上传

2. 创建空 `AST`

   ![](https://attach.52pojie.cn/forum/202506/22/185426lxs7kyj1lzmxskjx.png)

   image.png（4.71 KB，下载次数：1）

   [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDU4NXwyMDJkYWM3MnwxNzkzNTcwODY2fDE2ODA4NnwyMDQwNzg5&nothumb=yes)

   2025-6-22 18:54 上传

3. 模拟虚拟机执行环境。
4. 加载 `base64` 解码后的指令。

   ![](https://attach.52pojie.cn/forum/202506/22/185538ovy93f3ih0tqyzxf.png)

   image.png（28.86 KB，下载次数：0）

   [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDU4N3w3NTIwMzdlNHwxNzkzNTcwODY2fDE2ODA4NnwyMDQwNzg5&nothumb=yes)

   2025-6-22 18:55 上传

5. 指令处理循环：读取操作码、处理不同的操作码、解码操作码语义。

   ![](https://attach.52pojie.cn/forum/202506/22/221129byb84e8ydpqbnt4t.png)

   image.png（77.73 KB，下载次数：0）

   [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDYyOXwyMjQ1MDBjOHwxNzkzNTcwODY2fDE2ODA4NnwyMDQwNzg5&nothumb=yes)

   解码操作码语义  
   2025-6-22 22:11 上传

6. 类型推断生成 `AST`：生成不同类型节点，根据值类型生成对应 `AST` 节点。

   ![](https://attach.52pojie.cn/forum/202506/22/185711cu6abuqux8gumz5q.png)

   image.png（41.56 KB，下载次数：0）

   [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDU5MHw0ZDIxZTIwM3wxNzkzNTcwODY2fDE2ODA4NnwyMDQwNzg5&nothumb=yes)

   2025-6-22 18:57 上传

7. `AST` 生成：根据操作类型创建对应的 `AST` 节点，将节点添加到程序体。

   ![](https://attach.52pojie.cn/forum/202506/22/185633qerzpo2lsgvsp2n7.png)

   image.png（23.3 KB，下载次数：1）

   [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDU4OXwwNzY3NDFiY3wxNzkzNTcwODY2fDE2ODA4NnwyMDQwNzg5&nothumb=yes)

   2025-6-22 18:56 上传

## 六、输出虚拟化之前的 JS 代码

![](https://attach.52pojie.cn/forum/202506/22/185256avedtdjyvhd0p3kh.png)

image.png（123.73 KB，下载次数：2）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDU4M3w2MjdiNzZhOXwxNzkzNTcwODY2fDE2ODA4NnwyMDQwNzg5&nothumb=yes)

2025-6-22 18:52 上传

一个变种白盒 `AES`，一个变异的 `base64`。

### web 端协议参数生成

![](https://attach.52pojie.cn/forum/202506/22/221714emv3yrh2ooyd3oli.png)

image.png（325.81 KB，下载次数：0）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDYzMHwwMGQ1N2I2YXwxNzkzNTcwODY2fDE2ODA4NnwyMDQwNzg5&nothumb=yes)

协议  
2025-6-22 22:17 上传

### app 端 webview 参数生成

![](https://attach.52pojie.cn/forum/202506/22/221923el9zxmzd2dmu9r0m.png)

image.png（198.22 KB，下载次数：1）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDYzMXw0NWU2NjEzN3wxNzkzNTcwODY2fDE2ODA4NnwyMDQwNzg5&nothumb=yes)

app  
2025-6-22 22:19 上传
