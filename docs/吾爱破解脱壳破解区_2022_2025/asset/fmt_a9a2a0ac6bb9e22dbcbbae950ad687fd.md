
```python
return matches

parttern = "41 ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? 9C ?? ?? ?? ?? ?? ?? ?? 9D 41 FF ?? ?? 41 ??"

#parttern="41 51 4C 8D 0D ?? ?? ?? ?? 4D 8D 89 ?? ?? ?? ?? 41 FF E1 ?? 41 59"
#parttern="41 ?? 4C 8D ?? ?? ?? ?? ?? 4D 8D ?? ?? ?? ?? ?? 41 FF ?? ?? 41 ??"
#parttern="51 48 8D ?? ?? ?? ?? ?? 48 8D ?? ?? ?? ?? ?? FF ?? E8 59"
#parttern="52 48 8D ?? ?? ?? ?? ?? 48 8D ?? ?? ?? ?? ?? FF ?? E8 5A"
#parttern="?? 48 B8 ?? ?? ?? ?? ?? ?? ?? ?? 9C ?? ?? ?? ?? ?? ?? ?? 9D FF ?? ?? ??"
#parttern="41 ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? 9C ?? ?? ?? ?? ?? ?? ?? 9D 41 FF ?? ?? 41 ??"
#parttern="52 48 8D ?? ?? ?? ?? ?? 48 8D ?? ?? ?? ?? ?? FF ?? E8 5A"
#parttern="41 ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? 9C ?? ?? ?? ?? ?? ?? ?? 9D 41 FF ?? ?? 41 ??"
#parttern="41 ?? 4C 8D ?? ?? ?? ?? ?? 4D 8D ?? ?? ?? ?? ?? 41 FF ?? ?? 41 ??"
#parttern="41 51 4C 8D 0D ?? ?? ?? ?? 4D 8D 89 ?? ?? ?? ?? 41 FF E1 ?? 41 59"
#parttern="51 48 B9 ?? ?? ?? ?? ?? ?? ?? ?? 9C 48 81  ?? ?? ?? ?? ?? 9D FF E1 E8 59"
#parttern="52 48 8D ?? ?? ?? ?? ?? 48 8D ?? ?? ?? ?? ?? FF E2 E9 5A"
#parttern="E9 01 00 00 00 ??"
#parttern="50 48 8D ?? ?? ?? ?? ?? 48 8D ?? ?? ?? ?? ?? FF E0 ?? 58"
#parttern="51 48 8D ?? ?? ?? ?? ?? 48 8D ?? ?? ?? ?? ?? FF E1 ?? 59"
#parttern="?? 48 ?? ?? ?? ?? ?? ?? ?? ?? ?? 9C 48 ?? ?? ?? ?? ?? ?? 9D FF ?? ?? ??"

patch_addr_list1 = find_pattern(parttern)

def patch_flower(addr):
    fill_nop(addr,(len(parttern)+2)//3)

for addr in patch_addr_list1:
    print("[+]", hex(addr))
    patch_flower(addr)
```

每个特征码运行一遍大部分的函数都能进行反编译了（特征码是边分析边总结的，所以可能存在重复的）

## 静态分析

顺着消息回调函数找到关键调用

![](https://attach.52pojie.cn/forum/202504/01/173601hd29h4cdcz9zy4d3.png)

17.png (74.97 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc2Nzk4NnwyNjBkMDE2Y3wxNzkzNTcxMDc3fDE2ODA4NnwyMDIwMTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2Nzk4NnwyNjBkMDE2Y3wxNzkzNTcxMDc3fDE2ODA4NnwyMDIwMTg4&nothumb=yes)
2025-4-1 17:36 上传

由于通信的时候`magic==0x154004`是确定的，另外一个分支是测试使用的，因此完全可以不用分析，只分析`1AA0`函数即可。

本场比赛的第一个需要注意的点（不能算坑，只是踩了）：

![](https://attach.52pojie.cn/forum/202504/01/173603nb4qj76pzs5qj8k9.png)

18.png (91.03 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc2Nzk4N3wxZDEwMWE4N3wxNzkzNTcxMDc3fDE2ODA4NnwyMDIwMTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2Nzk4N3wxZDEwMWE4N3wxNzkzNTcxMDc3fDE2ODA4NnwyMDIwMTg4&nothumb=yes)
2025-4-1 17:36 上传

可以发现`R3`传过来的数据是每两个字节为一组，每个字节零扩展成`unsigned int`类型作为TEA加密的明文传入。

TEA加密看似是标版，实则解密之后会发现不对，这里可以先dump`140004060`的数据，尝试进行TEA解密。其实很好判断解密是否成功，解密之后的数据异或`sxx`之后，应当得到一个`@`开头的全ASCII字符，标准解密失败之后有次不小心交叉引用`TEAEnc`函数的时候发现了问题所在。

![](https://attach.52pojie.cn/forum/202504/01/173605sfoobbkudzz1qqfo.png)

20.png (110.85 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc2Nzk4OHw1ZjBiZjFmZnwxNzkzNTcxMDc3fDE2ODA4NnwyMDIwMTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2Nzk4OHw1ZjBiZjFmZnwxNzkzNTcxMDc3fDE2ODA4NnwyMDIwMTg4&nothumb=yes)
2025-4-1 17:36 上传

显然，该函数是被hook了，这里直接考虑动态调试去dump。

## 动态调试

![](https://attach.52pojie.cn/forum/202504/01/173607jvh7ekf4bll4vd1d.png)

21.png (120.06 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc2Nzk4OXxjZmRjMzJhOHwxNzkzNTcxMDc3fDE2ODA4NnwyMDIwMTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2Nzk4OXxjZmRjMzJhOHwxNzkzNTcxMDc3fDE2ODA4NnwyMDIwMTg4&nothumb=yes)
2025-4-1 17:36 上传

找到地址直接去看看hook函数。

![](https://attach.52pojie.cn/forum/202504/01/173610rlj8yd22jg4ewxx1.png)

22.png (170.44 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc2Nzk5MHw3YTY4YmJhMXwxNzkzNTcxMDc3fDE2ODA4NnwyMDIwMTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2Nzk5MHw3YTY4YmJhMXwxNzkzNTcxMDc3fDE2ODA4NnwyMDIwMTg4&nothumb=yes)
2025-4-1 17:36 上传

好在该hook也不复杂，但是直接分析汇编指令显然也不明智，把hook跳板拆开，将有效的指令插入原code中，最后修正偏移即可，这里可以直接选择区域导出十六进制值放`CyberChef`去分析，修跳转偏移也是。

![](https://attach.52pojie.cn/forum/202504/01/173612gohpb95621p5lwsr.png)

23.png (318.58 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc2Nzk5MXwxY2RlYTQzNHwxNzkzNTcxMDc3fDE2ODA4NnwyMDIwMTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2Nzk5MXwxY2RlYTQzNHwxNzkzNTcxMDc3fDE2ODA4NnwyMDIwMTg4&nothumb=yes)
2025-4-1 17:36 上传

修图中框选的指令偏移即可。最后直接用`IDA`反编译，得到最终结果。

![](https://attach.52pojie.cn/forum/202504/01/173614o0ys9b78gkh0a1ag.png)

24.png (110.09 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc2Nzk5Mnw0YWY0NzM2MnwxNzkzNTcxMDc3fDE2ODA4NnwyMDIwMTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2Nzk5Mnw0YWY0NzM2MnwxNzkzNTcxMDc3fDE2ODA4NnwyMDIwMTg4&nothumb=yes)
2025-4-1 17:36 上传

根据伪代码写逆向逻辑即可。

```c
#include
#include
#include

void decrypt_tea(uint32_t* v, uint32_t* k) {
        uint32_t delta = 0x9e3779b9;
    uint32_t v0 = v[0], v1 = v[1], sum = delta*32, i;     /* set up */
    uint32_t k0 = k[0], k1 = k[1], k2 = k[2], k3 = k[3];    /* cache key */
    for (i = 0; i > 11) & 3]; 
            v1 -= result ^ (v0 + ((16 * v0) ^ (v0 >> 5)));
            v0 -= (sum + v1) ^ (k0 + 16 * v1) ^ (k1 + (v1 >> 5));
        sum -= delta;
    }                                                           /* end cycle */
    v[0] = v0; v[1] = v1;
}
unsigned char ida_chars[200] =
{
        ...
};
char enc[100]={0};
int main(){

        uint32_t k[]={
                'A','C',
                'E','6',
        };
        for(int i=0;i
```

手动去掉`@`，然后逆转再`base58`解码，得到答案。

![](https://attach.52pojie.cn/forum/202504/01/173616hspt8a4tthtdt48t.png)

25.png (51.61 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=Mjc2Nzk5M3wzYzZiNjA5NHwxNzkzNTcxMDc3fDE2ODA4NnwyMDIwMTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2Nzk5M3wzYzZiNjA5NHwxNzkzNTcxMDc3fDE2ODA4NnwyMDIwMTg4&nothumb=yes)
2025-4-1 17:36 上传

所以最终正确输入就是`ACE_We1C0me!T0Z0Z5GamESecur1t9*CTf`。
