
- [//blog.csdn.net/beyond702/article/details/51744082" target="_blank" rel="noopener noreferrer nofollow"](<https://blog.csdn.net/beyond702/article/details/51744082:rel="nofollow noopener" href="https>)
- [//juejin.cn/post/7072897327509274631#heading-7" target="_blank" rel="noopener noreferrer nofollow"](<https://juejin.cn/post/7072897327509274631#heading-7:rel="nofollow noopener" href="https>)
- [//blog.csdn.net/chzphoenix/article/details/80569567" target="_blank" rel="noopener noreferrer nofollow"](<https://blog.csdn.net/chzphoenix/article/details/80569567:rel="nofollow noopener" href="https>)

如果可以全部看完，那你对`resources`的文件结构和打包流程、资源管理及资源的编译的有了一定的了解。

## `res`文件夹

`res`：资源文件目录，二进制格式。实际上，APK文件下的`res`文件夹并不是二进制格式，而是经过编译后的二进制资源文件。在Android应用程序开发中，资源文件通常是以XML格式存储的，如布局文件、字符串资源、颜色资源等。在编译时，Android编译器会将这些XML资源文件编译成二进制格式的资源文件，以提高应用程序的运行效率和安全性。虽然`res`文件夹下的二进制资源文件不能直接编辑和修改，但是开发者仍然可以通过Android提供的资源管理工具，如`aapt`、`apktool`等，来反编译和编辑这些资源文件的。

在`res`文件夹中，主要包含以下子文件夹和文件：

| `res`子目录 | 存储的资源类型 |
| --- | --- |
| `animator/` |  |
