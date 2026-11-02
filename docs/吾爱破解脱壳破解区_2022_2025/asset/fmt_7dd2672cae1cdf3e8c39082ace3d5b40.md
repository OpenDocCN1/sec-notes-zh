
# 逆向分析记录

## 定位 `v16` 初值

```
((void (__cdecl *)(char *, int *))sub_401470)(v16, &v13);

return v13 == 0125624;
```

可以看到 `v16` 在这里得到初值：`v16[v5] = byte_416050[(unsigned __int8)(v7 + v12)] ^ v15[v4 - 1];`

## 3. `sub_401470` 函数分析

`sub_401470` 函数进去可以看到如下信息（去掉反调试和干扰代码整理后），可以看到 `a2`（第二个参数就是传递来的 `v16` 了）就是不断进行比较，接下来大胆假设 `a2` 就是相等的，那么 `v16` 就是 `dbappsec` 了。

```c
unsigned int *__usercall sub_401470@(int a1@, _BYTE *a2, unsigned int *a3)
{
  int *_EAX; // eax
  char v5; // al
  char _AL; // al
  unsigned int *result; // eax

  if ( *a2 != 'd' )
    *a3 ^= 3u;
  else
    *a3 |= 4u;
  if ( a2[1] != 'b' )
  {
    *a3 &= 0x61u;
    _EAX = (int *)*a3;
  }
  else
  {
    _EAX = (int *)a3;
    *a3 |= 0x14u;
  }

  if ( a2[2] != 'a' )
    *a3 &= 0xAu;
  else
    *a3 |= 0x84u;
  if ( a2[3] != 'p' )
    *a3 >>= 7;
  else
    *a3 |= 0x114u;
  if ( a2[4] != 'p' )
    *a3 *= 2;
  else
    *a3 |= 0x380u;

  if ( a2[5] != 's' )
  {
    v5 = (char)a3;
    *a3 ^= 0x1ADu;
  }
  else
  {
    *a3 |= 0xA04u;
    v5 = (char)a3;
  }

  if ( a2[6] != 'e' )
    *a3 |= 0x4Au;
  else
    *a3 |= 0x2310u;
  if ( a2[7] != 'c' )
  {
    *a3 &= 0x3A3u;
    return (unsigned int *)*a3;
  }
  else
  {
    result = a3;
    *a3 |= 0x8A10u;
  }
  return result;
}
```

## 4. 动态数组 `byte_416050` 调试

`byte_416050` 是一个动态数组，我们调试看一看，`v16` 到底怎么来的。OD 往下翻，`00401A4E` 这一行对应的 `while ( v5`，汇编如下：

```asm
00401A4E   >  83BD E4FDFFFF>cmp dword ptr ss:[ebp-0x21C],0x8
00401A55   . |0F8D B0010000 jge crack2.00401C0B
00401A5B   . |0FB695 F6FDFF>movzx edx,byte ptr ss:[ebp-0x20A]
00401A62   . |83C2 01       add edx,0x1
00401A65   . |8895 F6FDFFFF mov byte ptr ss:[ebp-0x20A],dl
00401A6B   . |0FB685 F5FDFF>movzx eax,byte ptr ss:[ebp-0x20B]
00401A72   . |0FB68D F6FDFF>movzx ecx,byte ptr ss:[ebp-0x20A]
00401A79   . |0FB691 506041>movzx edx,byte ptr ds:[ecx+0x416050]

00401AE8   . /74 16         je short crack2.00401B00                 ;  
00401AEA   . |0FB695 F6FDFF>movzx edx,byte ptr ss:[ebp-0x20A]
00401AF1   . |0FB685 F5FDFF>movzx eax,byte ptr ss:[ebp-0x20B]
00401AF8   . |03D0          add edx,eax
00401AFA   . |8895 F7FDFFFF mov byte ptr ss:[ebp-0x209],dl
00401B00   > \0FB68D F7FDFF>movzx ecx,byte ptr ss:[ebp-0x209]        
```

## 5. 关键汇编解读

从 `401B00` 这里开始就很关键了，一定到定位到 `v16` 赋值的地方，下面逐行解读汇编代码。

```asm
00401B00   > \0FB68D F7FDFF>movzx ecx,byte ptr ss:[ebp-0x209]        ;  ecx=v12

00401B07   .  0FB695 F2FDFF>movzx edx,byte ptr ss:[ebp-0x20E]        ;  edx=v7

00401B0E   .  03CA          add ecx,edx                              ;  ecx=v7+v12

00401B10   .  888D F7FDFFFF mov byte ptr ss:[ebp-0x209],cl

00401B16   .  0FB685 F7FDFF>movzx eax,byte ptr ss:[ebp-0x209]        ;  eax=ecx

00401B1D   .  8A88 50604100 mov cl,byte ptr ds:[eax+0x416050]

00401B23   .  888D F7FDFFFF mov byte ptr ss:[ebp-0x209],cl

00401B29   .  8B95 D8FDFFFF mov edx,dword ptr ss:[ebp-0x228]         ;  edx=v4

00401B2F   .  0FB68415 FCFD>movzx eax,byte ptr ss:[ebp+edx-0x204]    ;  eax=v15[v4 - 1]

00401B37   .  0FB68D F7FDFF>movzx ecx,byte ptr ss:[ebp-0x209]        ;  ecx=byte_416050[(unsigned __int8)(v7 + v12)]

00401B3E   .  33C1          xor eax,ecx                              ;  v16=eax^ecx
```

## 6. 已知 `v16` 为 `dbappsec`

我们已知 `v16` 是 `dbappsec`，通过 `F2` 下断点能读出 `ecx` 的值 `0x2a,0xd7,0x92,0xe9,0x53,0xe2,0xc4,0xcd`，`v15` 就是密码。

```c
 if ( isdigit(a2[v6]) )
    {
      v8 = a2[v6] - 48;

    v9 = v8 + 16 * v9;
    if ( !((int)(v6 + 1) % 2) )  //v6如果为奇数，条件成立
    {
      v15[v3++ - 1] = v9; v15就是前一位*16+后一位
      v9 = 0;
    }
    ++v6;
```

## 7. 编写解密脚本

所以最后可写出如下脚本，用 C 实在不知道咋转 16 进制，懵逼，还好 Java 基础也懂一点，勉强写出来了。

```java
package ctf;

import java.io.UnsupportedEncodingException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Base64;
import java.util.Iterator;

public class test01 {
    public static void main(String[] args) throws NoSuchAlgorithmException, UnsupportedEncodingException {
        int arr[] = {0x2a,0xd7,0x92,0xe9,0x53,0xe2,0xc4,0xcd};
        char password[] = {'d','b','a','p','p','s','e','c'};
        byte flag[] = new byte[8];

        for (int i = 0; i = 0 && number 
```

## 8. 运行结果

运行后输出：`4eb5f3992391a1ae`，放到 md5 在线加密网站，加密得到 `flag`：`flag{d2be2981b84f2a905669995873d6a36c}`。

## 3> `0x5` 参考资料

1. [`strivemario.work/archives/b40b8192.html`](http://strivemario.work/archives/b40b8192.html)
2. [`blog.csdn.net/qq_39542714/article/details/106834898`](https://blog.csdn.net/qq_39542714/article/details/106834898)
3. [`wwtc.lanzoum.com/iV3sJ0y0rfzc`](https://wwtc.lanzoum.com/iV3sJ0y0rfzc)
4. [`www.52pojie.cn/thread-1584115-1-1.html`](https://www.52pojie.cn/thread-1584115-1-1.html)
