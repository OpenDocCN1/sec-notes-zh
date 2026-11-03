
# Lua 文件解密分析

## 关键代码片段

```cpp
luaL_error(a1, "error loading module %s from file %s :\n\t%s", v75, v79, v78);
}
goto LABEL_99;
}
if ( (v138 & 1) != 0 )
    cocos2d::CCLog((cocos2d *)"can not get file data of %s", v139);
else
    cocos2d::CCLog((cocos2d *)"can not get file data of %s", (const char *)&v138 + 1);
LABEL_100:
std::string::~string(&v138);
return 1LL;
}
```

## 分析流程

通过分析，我们知道，他先是读入`lua`文件，然后判断文件头，根据不同的文件头走不同的流程。我们的文件头是`FFSZ`，走的应该是`des`解密的流程；`des`解密之后还有一个`zlib`的解压缩。ok，按照这个思路开始写代码。

一开始写的时候，我去掉文件头直接`des`解密，但得出来的结果一直不符合预期。所以我直接跳过了12个字节重新解密，别问我为什么跳过12个字节，我也是猜的。然后解密出的结果，第九个字节刚好是`789c`，标准的`zlib`头。这里再进行`zlib`解压，好家伙，直接出明文了。解密代码如下：

```python
import pyDes,base64,zlib
from pyDes import des, ECB
def des_descrypt(s,KEY): 
    secret_key = KEY
    iv = secret_key
    k = des(secret_key, ECB,IV=iv, pad=None)
    de = k.decrypt(s)
    return de 
 
f=open('data.lua','rb').read()
f=f[12:]
ret = des_descrypt(f,b'FP_F_ENC') 

new_data = zlib.decompress(ret[8:])

print(new_data)

open('data_decrypt.lua','wb').write(new_data)
```

## 验证结果

看一下解密出来的文件，完美！

![](https://attach.52pojie.cn/forum/202302/14/181242vfu5zq3uxw6xu83w.png)

`image.png`（107.93 KB，下载次数：1）

2023-2-14 18:12 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU5MDY1MHwwNDI5OTA2NnwxNzkzNTY5Njk2fDE2ODA4NnwxNzQ2MDI4&nothumb=yes)

## 总结

好了，至此此文件的解密完成。总结一下，平常我们遇到的都是`xxtea`的`lua`加密，但是这个`lua`的`so`里面虽然也有`xxtea`，但是用了`des`加密并且进行了压缩，所以遇到`lua`的游戏的时候不要慌，冷静分析，也是可以搞定的

## 常见压缩算法头特征

最后附一下常见各种压缩算法的头特征：

```text
gzip 1f 8b 08
lzma 6c 00
zlib 78 9c
```
