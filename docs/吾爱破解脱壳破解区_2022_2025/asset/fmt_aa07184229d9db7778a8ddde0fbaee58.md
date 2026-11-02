
# 逆向分析笔记

## 汇编代码

```
.text:00F91F03 movzx   ecx, [ebp+eax+var_1C+0Ch]
.text:00F91F08 add     edx, ecx
.text:00F91F0A mov     eax, [ebp+var_20]
.text:00F91F0D mov     [ebp+eax+var_1C+4], dl
.text:00F91F11 mov     ecx, [ebp+var_20]
.text:00F91F14 movsx   edx, [ebp+ecx+var_1C]
.text:00F91F19 mov     eax, [ebp+var_20]
.text:00F91F1C movzx   ecx, [ebp+eax+var_1C+8]
.text:00F91F21 shl     edx, cl
.text:00F91F23 mov     ecx, [ebp+var_20]
.text:00F91F26 mov     [ebp+ecx+var_1C+10h], dl
.text:00F91F2A mov     edx, [ebp+var_20]
.text:00F91F2D mov     eax, [ebp+var_20]
.text:00F91F30 mov     dl, [ebp+edx+var_1C+10h]
.text:00F91F34 mov     cl, [ebp+eax+var_1C+4]
.text:00F91F38 shr     dl, cl
.text:00F91F3A mov     eax, [ebp+var_20]
.text:00F91F3D mov     [ebp+eax+var_1C+10h], dl
.text:00F91F41 mov     ecx, [ebp+var_20]
.text:00F91F44 movzx   edx, [ebp+ecx+var_1C+8]
.text:00F91F49 mov     eax, 8
.text:00F91F4E sub     eax, edx
.text:00F91F50 mov     [ebp+var_22], al
.text:00F91F53 mov     ecx, [ebp+var_20]
.text:00F91F56 movsx   edx, [ebp+ecx+var_1C]
.text:00F91F5B movzx   ecx, [ebp+var_22]
.text:00F91F5F sar     edx, cl
.text:00F91F61 mov     [ebp+var_24], dl
.text:00F91F64 mov     eax, [ebp+var_20]
.text:00F91F67 movzx   ecx, [ebp+eax+var_1C+0Ch]
.text:00F91F6C mov     edx, 8
.text:00F91F71 sub     edx, ecx
.text:00F91F73 mov     [ebp+var_23], dl
.text:00F91F76 mov     eax, [ebp+var_20]
.text:00F91F79 movsx   edx, [ebp+eax+var_1C]
.text:00F91F7E movzx   ecx, [ebp+var_23]
.text:00F91F82 shl     edx, cl
.text:00F91F84 mov     [ebp+var_21], dl
.text:00F91F87 mov     eax, [ebp+var_20]
.text:00F91F8A mov     dl, [ebp+var_21]
.text:00F91F8D mov     cl, [ebp+eax+var_1C+4]
.text:00F91F91 shr     dl, cl
.text:00F91F93 mov     [ebp+var_21], dl
.text:00F91F96 movzx   eax, [ebp+var_24]
.text:00F91F9A mov     ecx, [ebp+var_20]
.text:00F91F9D movzx   ecx, [ebp+ecx+var_1C+8]
.text:00F91FA2 shl     eax, cl
.text:00F91FA4 movzx   edx, [ebp+var_21]
.text:00F91FA8 or      eax, edx
.text:00F91FAA mov     ecx, [ebp+var_20]
.text:00F91FAD mov     [ebp+ecx+var_1C+14h], al
.text:00F91FB1 jmp     loc_F91EE5
```

## 分析说明

然后我们看这部分汇编, `ebp+var_20` 是一个循环变量,我们要循环四次,对应每段flag的四位

我们把刚刚两个数组分别称为 `high[]` 和 `low[]`

### 手动反汇编

```
定义 : 
ebp+var_20 : i
ebp+var_1C : input
ebp+var_1C + 8 : high
ebp+var_1C + 0xC : low
ebp+var_1C + 4 : tmp
ebp+var_1C + 0x10 : dst
ebp+var_1C + 0x14 : target

for(int i = 0; i >= tmp[i] // SHR 
    int tmp1 = 8 - h[i] // ebp + var_22
    int tmp2 = input[i] >> tmp3 // ebp + var_24 // SAR
    int tmp3 = 8 - l[i] // ebp + var_23
    int tmp4 = (input[i] >= tmp[i]
    target[i] = (tmp2 
```

于是设计一个结构体存每一段的信息, 因为发现一会还有一个函数要用...

![](https://tianyu.xin/usr/uploads/2022/01/2889855251.png)

`this_24`调用的另一个函数是`this_36`, 地址在`F928A0`

现在是北京时间1:56, 大半夜的多少有点整不动了, 起床再来

接着跟`this_36`

把`this[26]`放成由一个函数拿到的地址上的值,盲猜都能猜到是刚刚生成的东西, 跳转验证一下

![](https://tianyu.xin/usr/uploads/2022/01/794173212.png)

![](https://tianyu.xin/usr/uploads/2022/01/487473139.png)

正是

![](https://tianyu.xin/usr/uploads/2022/01/789796.png)

把指针类型改正确后, 加密逻辑清晰了一些, 其中`V15` 应该是最后存放数据的地址, `v14` 初始值为 `32`, `F92DC0` 应该是程序获取地址的一个函数, 暂不用去理, 这段程序是这样的

```
int base = 32;

int dst0 = 0;

uint8_t dst1[4];

for(int i = 0; i = 4) {

        base -= flag_sec[i]; //  low_plus_high[i-4]

        dst0 |= data_enc[i]  data_enc2[i-4]

    }

    else {

        base -= 8 - hpl[i];

        dst0 |= data_enc[i] 
```

有了加密逻辑, 有了目标值, 直接进行一个爆破, 不过爆破的范围好像比较大, 我们用 C 爆一波

```
#include 
#include 
#include 
uint8_t tar1[20] = {
    0x08, 0xEA, 0x58, 0xDE, 
    0x94, 0xD0, 0x3B, 0xBE, 
    0x88, 0xD4, 0x32, 0xB6, 
    0x14, 0x82, 0xB7, 0xAF, 
    0x14, 0x54, 0x7F, 0xCF
};
uint8_t tar2[20] = {
    0x20, 0x20, 0x30, 0x33, 
    0x22, 0x33, 0x20, 0x20, 
    0x20, 0x30, 0x20, 0x32, 
    0x30, 0x33, 0x22, 0x20, 
    0x20, 0x20, 0x24, 0x20
};
struct data {
    uint8_t sec[4];
    uint8_t sum[4];
    uint8_t hig[4];
    uint8_t low[4];
    uint8_t enc[4];
    uint8_t enc1[4];
};
uint8_t hig(uint8_t x) {
    uint8_t res = 0;
    while((x & 0x80) == 0) {
        x >= 1;
        res ++;
    }
    return res;
}
void findAns(int tar0, uint8_t* tar1) {
    int x[4];
    for(x[0] = 30; x[0] > tmp.sum[i];
            tmp.enc1[i] = ((x[i] >> (8 - tmp.hig[i])) > tmp.sum[i]);
        }

        // for(int i = 0; i 
```

`swpuctf{we18-l8co-m1e4-58to-swpu}`

## [FBCTF2019] go_get_the_flag

`golang` 逆向, 检查字符串之后推测是通过参数直接给出flag, 然后在`main.checkpassword`函数里check一下

第一个参数是输入的地址,

![](https://tianyu.xin/usr/uploads/2022/01/1492356164.png)

结合动调,第二个参数猜测是字符串长度.

把输入和`s0_M4NY_Func710n2!`做`memequal`, 猜测是判等, 直接运行之, 给出了一串输出

```

fb{.60pcln74b_15_4w350m3}

后面 is awesome 清晰可见,但是前面应该还有些不对, 不管如何,先交一发试试

对了...

## [RCTF2019] DontEatMe

输入, 然后有一个固定种子的`rand`, 动调获取数据

前面奇奇怪怪的东西会让动调挂掉, 直接全扬了防止反调

然后程序又主动给这部分`rand`过的东西赋值了, 看来`rand`的没啥用, 障眼法

函数`4E1090`用了这部分内容给一个长度17的32位栈变量赋值, 输入无关, 直接拿出来

![](https://tianyu.xin/usr/uploads/2022/01/2886189656.png)

从`4E501A`到`4E503A`,又做了一些输入无关的事情

然后是`4E5018`到`5038`, 注意到覆盖了前一段, 翻到下面发现一个用`wasd`的迷宫,迷宫内容在`4E53A8`, 可以动调拿, 输出成地图

```
int main() {
    freopen("maze.txt", "w", stdout);
    int *p = (int*) ida_chars;
    for(int i = 0; i 
```

初始位置`(10, 5)`, 目标位置 `(4, 9)`, 要求是17 步内走到, `ddddwwwwwaaawwwdd`,

步骤来自于`v23`, `v23`来自于`v51` (实际`IDA`版本和环境不同可能不同). `v51`是一个`calloc`的, 长度是输出长度扩展为8的整数倍的地址, 继续追溯可以追溯到这里:

![](https://tianyu.xin/usr/uploads/2022/01/2853921235.png)

`v7`是个指针 `4E11EB`是个`sscanf`, 大概就是把输入转成十六进制,再加密最后的结果要是迷宫步, 这个加密多少有点大, 感觉是现成的加密. 回到刚刚忽略的函数里面想找到点蛛丝马迹和魔数之类的, 点进去一看,好家伙,不用找了, 没去符号, 知道是`BLOWFISH`加密了. 秘钥明文给出的

![](https://tianyu.xin/usr/uploads/2022/01/1413661006.png)

,

![](https://tianyu.xin/usr/uploads/2022/01/2481939670.png)
