
# 管道创建和获取

通过管道获取指令，执行后门功能。

![](https://attach.52pojie.cn/forum/202501/27/103255jam1m1mf716612gl.png)

## 执行后门功能

系统管理类后门功能对应的控制码如下。

![](https://attach.52pojie.cn/forum/202501/27/110652iat73rjxaqh6726a.png)

### 系统管理类后门功能

此外，还存在 `CMD` 后门、网络相关、`SQL` 管理等后门功能。

![](https://attach.52pojie.cn/forum/202501/27/110655g5nyatuagbnyjdta.png)

### 其他后门功能

部分后门功能的具体操作如下：

#### 剪贴板管理

将剪贴板数据写入文件 `NvSmart.x64.hlp` 中。

![](https://attach.52pojie.cn/forum/202501/27/103321er46yybryznvl2gr.png)

#### 键盘记录

记录数据。

![](https://attach.52pojie.cn/forum/202501/27/103327jor1261omb34ymry.png)

#### 插件加载

遍历 `C:\ProgramData\Microsoft\CryptSvcser` 目录下的 `0~127.plg` 文件，并将其反射加载。

![](https://attach.52pojie.cn/forum/202501/27/103334x6rbi5xzwrkiqi25.png)

# 二、附录

## C&C

![](https://attach.52pojie.cn/forum/202501/27/103339ddydza1q112qn05v.png)

## HASH

![](https://attach.52pojie.cn/forum/202501/27/103345hs97p3ay6c13alaa.png)
