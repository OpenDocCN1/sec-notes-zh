
```java
private void hook_A7A() {
    // A7A
    emulator.attach().addBreakPoint(module.base + 0xA7A + 1, new BreakPointCallback() {
        @Override
        public boolean onHit(Emulator emulator, long address) {
            Number number2 = emulator.getBackend().reg_read(ArmConst.UC_ARM_REG_R2);
            System.out.println("[+] hook_A7A 第2轮结果-->>" + Integer.toHexString(number2.intValue()));
            return true;
        }
    });
}
```

- 看看结果如何；

![](https://www.xiayutian.xyz/usr/assets/android/%E9%AD%94%E6%94%B9md5%E5%88%86%E6%9E%90_img/image-20250204121017655.png)

- 第二轮也就还原完成了，再总结一下第二轮修改了什么：

- `k`值；
- 修改了 `b`, `c`, `d`, `a` 的位置；
- `FF`函数；

## 5> 2.5 第3-4轮

- 先看第三轮，分析如下：

```
md5_a = md5_d + v15;
md5_b = md5_a + v17;
v16 = md5_a & md5_d;
md5_c = md5_b + __ROR4__(md5_c + *(v10 - 1) + *v11 + (md5_b & ~md5_d | md5_a & md5_d), 15);
```

标准：`md5_c = FF(c, d, a, b, x[2], 17, 0x242070DB); /* 3 */`

```c
#define FF(a,b,c,d,x,s,ac) \
          { \
          a += F(b,c,d) + x + ac; \
          a = ROTATE_LEFT(a,s); \
          a += b; \
          }
#define F(x,y,z) ((x & y) | (~x & z))
```

`ida`：`md5_c = FF(c, d, a, b, x[2], 17, 0xFD7025F4); /* 3 */`

- 这个比较简单，非常好分析，最终只是魔改了`k`值，看看结果对比；

- `unidbg`结果：

```
[+] hook_A90 第3轮结果ccc-->>9c63c0bd
```

- 本地结果：

```
[+] 第3轮x2-->>37303036
[+] 第3轮c-->>9c63c0bd
```

可以看到第三轮就只是修改了`k`值；

再看第四轮；

```
ida：md5_d = md5_c + __ROR4__(v19 + v20 + (md5_c & ~md5_a | (md5_a + v17) & md5_a), 10);
标准：FF(b, c, d, a, x[3], 22, 0xc1bdceee); /* 4 */
#define FF(a,b,c,d,x,s,ac) \
          { \
          a += F(b,c,d) + x + ac; \
          a = ROTATE_LEFT(a,s); \
          a += d; \
          }
F(x,y,z) ((x & y) | (~x & z))     
v19 = md5_d + v18;
v20 = v11[2];
```

- 分析也是比较容易的，这里需要看一下`k`值，根据上面的分析，`v20`应该是`k`值，那么它等于`v11`的第二个数据，那就是往后推八个字节，看看是什么数据；

![](https://www.xiayutian.xyz/usr/assets/android/%E9%AD%94%E6%94%B9md5%E5%88%86%E6%9E%90_img/image-20250204134413860.png)

- 那就是`0xA7553036`，再看具体的`a b c d`，这里肯定是有变化的，因为`b`的值在前面已经被计算了，所以这里肯定是`d`的值，具体的修改如下：

```
FF(d, a, b, c, x[3], 22, 0xA7553036); /* 4 */
printf("[+] 第4轮x3-->>%x\n", x[3]);
printf("[+] 第4轮d-->>%x\n", d);
```

- 对比看看，`unidbg`结果：

```
[+] hook_AAC 第4轮结果ddd-->>466fc264
```

- 本地结果：

```
[+] 第4轮x3-->>8034
[+] 第4轮d-->>466fc264
```

- 那么第3-4轮也还原完毕，大致魔改位置汇总：

- `k`值；
- `a`,`b`,`c`,`d`顺序；
- `FF`函数；

## 5> 2.6 第5-16轮

- 根据伪代码分析，会把我们上述的步骤重复四次，但是每次的明文和`k`值是在往后移的，这里我们最好的选择肯定是和它一样写一个循环，因为具体的步骤都是一样的，只需要改变明文和`k`值；

```
for (int i = 0; i >%x\n", x[0]);
    printf("[+] 第1轮a-->>%x\n", a);
// 修改了k值 b, c, d, a的位置
    FF(b, c, d, a, x[i * 4 + 1], 12, k_table[i * 4 + 1]); /* 2 */
//        printf("[+] 第2轮x1-->>%x\n", x[1]);
    printf("[+] 第2轮b-->>%x\n", b);
// 修改了k值
    FF(c, d, a, b, x[i * 4 + 2], 17, k_table[i * 4 + 2]); /* 3 */
//        printf("[+] 第3轮x2-->>%x\n", x[2]);
    printf("[+] 第3轮c-->>%x\n", c);
// 修改了k值
    FF(d, a, b, c, x[i * 4 + 3], 22, k_table[i * 4 + 3]); /* 4 */
//        printf("[+] 第4轮x3-->>%x\n", x[3]);
    printf("[+] 第4轮d-->>%x\n", d);
    printf("[+++++++++++++++++++++++++++]\n");
}
```

- 这里的`k`表我们就需要腾出来了，并且分析一下代码；

![](https://www.xiayutian.xyz/usr/assets/android/%E9%AD%94%E6%94%B9md5%E5%88%86%E6%9E%90_img/image-20250204144836484.png)

- 它把`v11+8`，也就是向后移32个字节，去看看；

![](https://www.xiayutian.xyz/usr/assets/android/%E9%AD%94%E6%94%B9md5%E5%88%86%E6%9E%90_img/image-20250204144927707.png)

- 原始位置是2c00，加上32的十六进制也就是20，会定位到2c20，它的`k`值是从上往下取的，这是根据前四轮的分析得到的，我们就可以将具体的`k`值拿下来放在一个数组里，这里若不一样可以按下D改变数据的类型；

- 因此，前十六轮就可以还原为如下代码：

```
int k_table[16] = {
        0x8A51407D, 0x6A88995D, 0xFD7025F4, 0xA7553036,
        0x489E15C1,0xF5CDB84B, 0xC0FFBCF6, 0x253F7D7E,
        0xE93FD535, 0xD6CD6448,0x1220AE4, 0xD806D023,
        0xE84E6EA9, 0x230135D8, 0xC27AE834,0xF5292BF4
};
for (int i = 0; i >%x\n", a);
    FF(b, c, d, a, x[i * 4 + 1], 12, k_table[i * 4 + 1]); /* 2 */
    printf("[+] 第2轮b-->>%x\n", b);
    FF(c, d, a, b, x[i * 4 + 2], 17, k_table[i * 4 + 2]); /* 3 */
    printf("[+] 第3轮c-->>%x\n", c);
    FF(d, a, b, c, x[i * 4 + 3], 22, k_table[i * 4 + 3]); /* 4 */
    printf("[+] 第4轮d-->>%x\n", d);
    printf("[+++++++++++++++++++++++++++]\n");
}
```

- `unidbg`前16轮结果：

```
[+] hook_A66 第1轮结果aaa-->>559a829a
[+] hook_A7A 第2轮结果bbb-->>90a37ab1
[+] hook_A90 第3轮结果ccc-->>9c63c0bd
[+] hook_AAC 第4轮结果ddd-->>466fc264
[++++++++++++++++++++++++++++++++++++++]
[+] hook_A66 第1轮结果aaa-->>9a9c6a9e
[+] hook_A7A 第2轮结果bbb-->>ac6e9375
[+] hook_A90 第3轮结果ccc-->>4dfea254
[+] hook_AAC 第4轮结果ddd-->>db8ce9c4
[++++++++++++++++++++++++++++++++++++++]
[+] hook_A66 第1轮结果aaa-->>c1223db5
[+] hook_A7A 第2轮结果bbb-->>4c7c6281
[+] hook_A90 第3轮结果ccc-->>fdf68ba2
[+] hook_AAC 第4轮结果ddd-->>1882adb9
[++++++++++++++++++++++++++++++++++++++]
[+] hook_A66 第1轮结果aaa-->>4c20b8bc
[+] hook_A7A 第2轮结果bbb-->>5142113c
[+] hook_A90 第3轮结果ccc-->>ab3624a0
[+] hook_AAC 第4轮结果ddd-->>25b2551b
[++++++++++++++++++++++++++++++++++++++]
76B5CEBECD2484F78CDF7A468C133EFB
```

- 本地前16轮结果：

```
[+] 第1轮a-->>559a829a
[+] 第2轮b-->>90a37ab1
[+] 第3轮c-->>9c63c0bd
[+] 第4轮d-->>466fc264
[+++++++++++++++++++++++++++]
[+] 第1轮a-->>9a9c6a9e
[+] 第2轮b-->>ac6e9375
[+] 第3轮c-->>4dfea254
[+] 第4轮d-->>db8ce9c4
[+++++++++++++++++++++++++++]
[+] 第1轮a-->>c1223db5
[+] 第2轮b-->>4c7c6281
[+] 第3轮c-->>fdf68ba2
[+] 第4轮d-->>1882adb9
[+++++++++++++++++++++++++++]
[+] 第1轮a-->>4c20b8bc
[+] 第2轮b-->>5142113c
[+] 第3轮c-->>ab3624a0
[+] 第4轮d-->>25b2551b
[+++++++++++++++++++++++++++]
b47e51f64bac1c36d6b65dbc2ab2728c
```

- 可以发现前16轮也是还原出来了，但是最终结果依旧是不一样，所以需要继续还原；

## 5> 2.7 第17-32轮

- 与上面的逻辑其实无异了，先看第一轮；

```
v28 = __ROR4__(md5_a + (md5_c & ~md5_d | md5_b & md5_d) + v77[(v22 - 15) & 0xF] + v27, 27);
md5_a = md5_d + v28;
```

标准：`GG(a, b, c, d, x[1], 5, 0xf61e2562); /* 17 */`

```c
#define GG(a,b,c,d,x,s,ac) \
          { \
          a += G(b,c,d) + x + ac; \
          a = ROTATE_LEFT(a,s); \
          a += b; \
          }
#define G(x,y,z) ((x & z) | (y & ~z))
```

- `a += b -->> a += d`
- 依旧是一样的分析方式，这里最终其实就是修改了`k`值，然后同样修改了`GG`函数；
`GG(a, b, c, d, x[1], 5, 0x46711AC1); /* 17 */`
`printf("[+] Round 2 第1轮明文-->>%x\n", x[1]);`
`printf("[+] Round 2 第1轮a-->>%x\n", a);`

- 对比一下，先看`unidbg`的结果：

```
[+] hook_AFA Round 2 第1轮结果aaa-->>feba0245
```

- 本地模拟结果：

```
[+] Round 2 第1轮明文-->>39303834
[+] Round 2 第1轮a-->>feba0245
```

- 再看第二轮；

```
v29 = __ROR4__(md5_b + v77[(v22 - 10) & 0xF] + v24 + (md5_d & ~(md5_d + v28) | (md5_d + v28) & md5_c), 23);
md5_b = md5_a + v29;
md5_a = md5_d + v28;
//所以
v29 = __ROR4__(md5_b + v77[(v22 - 10) & 0xF] + v24 + (md5_d & ~(md5_a) | (md5_a) & md5_c), 23);
md5_b = md5_a + v29;
// 标准：
GG(d, a, b, c, x[6], 9, 0xc040b340); /* 18 */
#define GG(a,b,c,d,x,s,ac) \
          { \
          a += G(b,c,d) + x + ac; \
          a = ROTATE_LEFT(a,s); \
          a += b; \
          }
// a += b -->> a += d
#define G(x,y,z) ((x & z) | (y & ~z))
// 依旧分析 这里原本赋值应该是d 但是顺序又不一样了 那么这里是改了;
// 
GG(b, c, d, a, x[6], 9, 0xA90A840A); /* 18 */
```

- `unidbg`结果：

```
[+] hook_B1A Round 2 第1轮结果bbb-->>fc8ec38f
```

- 本地模拟结果：

```
[+] Round 2 第2轮明文-->>0
[+] Round 2 第2轮b-->>fc8ec38f
```

- 再看第三轮；

```
v30 = __ROR4__(md5_c + v77[(v22 - 5) & 0xF] + v25 + (md5_a & ~(md5_a + v29) | (md5_a + v29) & md5_d), 18);
md5_c = md5_b + v30;
md5_b = md5_a + v29;
// 所以：
v30 = __ROR4__(md5_c + v77[(v22 - 5) & 0xF] + v25 + (md5_a & ~(md5_b) | (md5_b) & md5_d), 18);
md5_c = md5_b + v30;
// 标准：
GG(c, d, a, b, x[11], 14, 0x265e5a51); /* 19 */
#define GG(a,b,c,d,x,s,ac) \
          { \
          a += G(b,c,d) + x + ac; \
          a = ROTATE_LEFT(a,s); \
          a += b; \
          }
// a += b -->> a += d
#define G(x,y,z) ((x & z) | (y & ~z))
// 改了k值；
// 
GG(c, d, a, b, x[11], 14, 0xFD1BBEF0); /* 19 */
```

- 结果依旧是没问题，看第四轮；

```
md5_d = md5_b + v30 + __ROR4__(md5_d + v77[v31] + v26 + (md5_b & ~(md5_b + v30) | (md5_b + v30) & md5_a), 12);
md5_c = md5_b + v30;
// 所以：
md5_d = md5_c + __ROR4__(md5_d + v77[v31] + v26 + (md5_b & ~(md5_c) | (md5_c) & md5_a), 12);
// 这里c肯定就是最后一个
// 标准：
GG(b, c, d, a, x[0], 20, 0xe9b6c7aa); /* 20 */
#define GG(a,b,c,d,x,s,ac) \
          { \
          a += G(b,c,d) + x + ac; \
          a = ROTATE_LEFT(a,s); \
          a += b; \
          }
```
