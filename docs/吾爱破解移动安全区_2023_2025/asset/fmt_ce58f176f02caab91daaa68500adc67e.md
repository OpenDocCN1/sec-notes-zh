
# [Android 原创] 某棋牌游戏 lua 逆向破解修改（一）

原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=1780626](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1780626)

抓取时间：2026-11-01T21:44:44

## 背景

最近朋友在玩几个棋牌游戏，我花了点时间研究了一下，虽然破解失败，但是过程感觉还是有收获，分享给大家

这篇文章主要分享 如何找 `key`

破解思路后面有空再分享

## cocos2dlua

棋牌游戏的逻辑一般是使用 lua 编写的，lua 脚本加载一般是用 cocos2d 加载的，我的破解方向集中在这里，如果你要破解的棋牌游戏不是这个技术栈，那就不能照搬

lua 脚本常见情况如下：

- `luac`（编辑器打开脚本，前面几个字节是 lua）
- `cocos2dlua`（编辑器打开脚本，前面几个字节，不是 lua，多打开几个会发现是固定的）

我这里遇到的就是 `cocos2dlua`（看一下 lib 文件有没有叫类似名字的 so）

`cocos2dlua` 加密原理给大家简单介绍一下

- 写好 lua 脚本
- 使用 `key` 进行加密（加密算法叫 `xxtea`）
- 加密完成之后在最前面拼接 `sign`

这个 `sign` 就是刚刚提到的固定字符串

解密步骤如下：

- 读取脚本，切掉前缀，也就是 `sign`
- 使用找到的 `key` 进行解密

那么解密难点就是找 `key`

`key` 一般来说都在 `libcocos2dlua.so` 里面，有以下几种玩法：

- 不修改 cocos2d 代码，明文存储
- 修改 cocos2d 代码，明文存储
- 修改 cocos2d 代码，代码计算得到

### 不修改 cocos2d 代码，明文存储

这种最简单了，编辑器打开 lua 脚本，直接搜前缀，`key` 就在附近

为什么在附近？因为源码要求 `sign` 和 `key` 作为参数一起传

```cpp
bool AppDelegate::applicationDidFinishLaunching()
{
    // set default FPS
    Director::getInstance()->setAnimationInterval(1.0 / 60.0f);

    // register lua module
    auto engine = LuaEngine::getInstance();
    ScriptEngineManager::getInstance()->setScriptEngine(engine);
    lua_State* L = engine->getLuaStack()->getLuaState();
    lua_module_register(L);

    register_all_packages();

    LuaStack* stack = engine->getLuaStack();
    stack->setXXTEAKeyAndSign("2dxLua", strlen("2dxLua"), "XXTEA", strlen("XXTEA"));
}
```

上面的 `2dxlua` 就是 `sign`，`XXTEA` 就是 `key`（加密算法也叫 `xxtea`）

源码在 [`AppDelegate.cpp#L74`](https://github.com/cocos2d/cocos2d-x/blob/90f6542cf7fb081335f04e474b880d7ce8c445a1/templates/lua-template-default/frameworks/runtime-src/Classes/AppDelegate.cpp#L74)

### 修改 cocos2d 代码，明文存储

不修改源码，直接记事本打开就能找到 `key`，所以一些开发者为了提升安全性，修改了 cocos2d 代码，让 `sign` 和 `key` 远离（比如写一个函数获取 `key`），增加破解难度

这里列举几个可能的变形点，依次在 ida 里找就可能找到

以下均为 function

- `applicationDidFinishLaunching`：入口点，默认 `sign` 和 `key` 在这里
- `setXXTEAKeyAndSign`：如函数名所示，设置 `sign` 和 `key`
- `xxtea_decrypt`：算法解密点，因为需要传入 `key` 进行解密，按 `x` 看调用的地方，有 `key` 的踪迹

下面举一个例子

010 打开 lua 脚本

![001.png](https://attach.52pojie.cn/forum/202305/02/011345csgijo7brzcz7ws8.png)

001.png (53.4 KB, 下载次数: 2) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTY1OXw3NmQ3ODNhZXwxNzkzNTY5NDgyfDE2ODA4NnwxNzgwNjI2&nothumb=yes) 2023-5-2 01:13 上传

看到了一个前缀

ida 分析 `libcocos2dlua.so`

搜索 `applicationDidFinishLaunching`，没有看到明文

![002.png](https://attach.52pojie.cn/forum/202305/02/012200diswik6nqizokk3f.png)

002.png (132.04 KB, 下载次数: 3) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTY2MHwyOTRlZmVmNnwxNzkzNTY5NDgyfDE2ODA4NnwxNzgwNjI2&nothumb=yes) 2023-5-2 01:22 上传

搜索 `setXXTEAKeyAndSign`，也没有看到明文

![003.png](https://attach.52pojie.cn/forum/202305/02/012202hzlroltadroedn7y.png)

003.png (68.59 KB, 下载次数: 4) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTY2MXw4YjVjYWYzNnwxNzkzNTY5NDgyfDE2ODA4NnwxNzgwNjI2&nothumb=yes) 2023-5-2 01:22 上传

搜索 `xxtea_decrypt`，出现两个，有戏，因为一般是一个

![004.png](https://attach.52pojie.cn/forum/202305/02/012204gx58j65xsrzryyxr.png)

004.png (9.3 KB, 下载次数: 4) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTY2MnxmNzdkOTg1ZHwxNzkzNTY5NDgyfDE2ODA4NnwxNzgwNjI2&nothumb=yes) 2023-5-2 01:22 上传

依次按 `x` 看调用的地方，第一个比较正经，是正常调用关系

第二个是自定义的函数，进去看看

![005.png](https://attach.52pojie.cn/forum/202305/02/012206peymk3snc2314c1g.png)

005.png (50.65 KB, 下载次数: 2) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTY2M3wwYWVkZDJiN3wxNzkzNTY5NDgyfDE2ODA4NnwxNzgwNjI2&nothumb=yes) 2023-5-2 01:22 上传

经过分析源码，得知 `key` 是图中的 `v21`（这里就不贴明文了，避免被搜索到，哈哈）

![006.png](https://attach.52pojie.cn/forum/202305/02/012208mhuw222x16h9jh92.png)

006.png (108.81 KB, 下载次数: 3) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTY2NHw2ZTlhZjVkNXwxNzkzNTY5NDgyfDE2ODA4NnwxNzgwNjI2&nothumb=yes) 2023-5-2 01:22 上传

到这里，`key` 就找到了

### 修改 cocos2d 代码，代码计算得到

另外一种变形就是 `key` 不是明文存的，是计算得到，下面举一个例子说明

步骤类似

010 打开

![007.png](https://attach.52pojie.cn/forum/202305/02/012515r88e9yg5h88znb45.png)

007.png (48.85 KB, 下载次数: 3) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTY2NXxlMDM4OGFhOXwxNzkzNTY5NDgyfDE2ODA4NnwxNzgwNjI2&nothumb=yes) 2023-5-2 01:25 上传

搜索 `applicationDidFinishLaunching`，看到明文了，但是有好几个字符串，到底是哪一个呢？

![008.png](https://attach.52pojie.cn/forum/202305/02/013301oipar2ipzuzia970.png)

008.png (96.61 KB, 下载次数: 2) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTY2NnxjNTkzOGE4NXwxNzkzNTY5NDgyfDE2ODA4NnwxNzgwNjI2&nothumb=yes) 2023-5-2 01:33 上传

上面出现好几个字符串的那一行，实际调用的是函数 `setXXTEAKeyAndSign`

搜索 `setXXTEAKeyAndSign`

![009.png](https://attach.52pojie.cn/forum/202305/02/013303tfb1zdtdoiqnzok4.png)

009.png (93.99 KB, 下载次数: 1) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTY2N3xkYzJmNjE5OHwxNzkzNTY5NDgyfDE2ODA4NnwxNzgwNjI2&nothumb=yes) 2023-5-2 01:33 上传

经过阅读代码，慢慢扣偏移，最后得知 `key` 是 `v21`（上面的例子也是 `v21`，这个只是巧合，不用在意）

当然了，这里也不用慢慢扣代码，可以写代码梭哈，输入 4 个字符串，前两个用 `0x2019%s%s` 组合起来了，后面两个也是一样的组合方式，加起来 6 个可能，依次遍历，看看到底是哪个就行

这里是梭哈的 golang 代码

![010.png](https://attach.52pojie.cn/forum/202305/02/013305zfg0aazc0fz9jztp.png)

010.png (51.53 KB, 下载次数: 3) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTY2OHxiODMzNjBmYnwxNzkzNTY5NDgyfDE2ODA4NnwxNzgwNjI2&nothumb=yes) 2023-5-2 01:33 上传

## 解密

上面是找 `key` 的方法，找到之后就要解密了

推荐自己写，我找了一些成品，只能说差强人意

推荐用 golang 写，无需乱七八糟的依赖，写完就运行

对 golang 不熟悉，也可以用 python，但是要依赖特定版本 visual studio，有多恶心，我就不多说了

以下是 golang 代码

```go
package main

import (
        "flag"
        "fmt"
        "io/ioutil"
        "os"
        "path/filepath"
        "time"

        "github.com/xxtea/xxtea-go/xxtea"
)

type CMD struct {
        encrypt_file string
        decrypt_file string
}

var prefix string = "xxxx"
var lua_key string = "xxxx"

func main() {
        cmd, r := initFlag()
        if r {
                return
        }

        if cmd.decrypt_file != "" {
                err := decrypt(cmd.decrypt_file)
                if err != nil {
                        fmt.Println(err)
                }
                return
        }

        if cmd.encrypt_file != "" {
                err := encrypt(cmd.encrypt_file)
                if err != nil {
                        fmt.Println(err)
                }
                return
        }

        fmt.Println("error, empty input")
}

func encrypt(path string) error {
        by, err := os.ReadFile(path)
        if err != nil {
                fmt.Println("read fail", err)
                return err
        }

        encrypt_data := xxtea.Encrypt(by, []byte(lua_key))

        // 拼接前缀
        data := append([]byte(prefix), encrypt_data...)

        // fileInfo, err := os.Stat(path)
        // if err != nil {
        //         return err
        // }

        name := filepath.Base(path)
        x1 := name[:len(name)-len(filepath.Ext(name))]
```
