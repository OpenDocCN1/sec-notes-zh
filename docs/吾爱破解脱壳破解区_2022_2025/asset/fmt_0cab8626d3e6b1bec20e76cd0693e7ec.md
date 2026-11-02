
```c
char flag[1000]="\\0";

 void DFS (int x,int y,int n){

     if(map[x][y]==1){

         return;

         }

     if(map[x][y]==7){

         flag[n]='\\0';

          printf("%s",flag);

         exit(0);

         }

         flag[n]='s';

         map[x][y]=1;//将走过的路径标记一下防止走回去

         DFS(x+1,y,n+1);

         flag[n]='w';

         map[x][y]=1;

         DFS(x-1,y,n+1);

         flag[n]='a';

         map[x][y]=1;

         DFS(x,y-1,n+1);

         flag[n]='d';

         map[x][y]=1;

         DFS(x,y+1,n+1);

 }

int main(){

    DFS(1,1,0);

    printf("%s",flag);

    return 0;

}
```
//输出ssssddssaassddddwwwwddwwddddddwwddddddssddwwddddddddssssaawwaassaassaassaassddssaassaawwwwwwaaaaaaaassaassddddwwddssddssssaassddssssaaaaaawwddwwaawwwwaassssssssssssddddssddssddddddddwwaaaaaawwwwddssddwwwwwwwwddssddssssssssddddss

将输出md5一下，得出flag为：vnctf{801f190737434100e7d2790bd5b0732e}

## 时空飞行

### 首先

![](https://s0rry-1308583710.cos.ap-chengdu.myqcloud.com/markdown/202203301821428.png)

无壳且64位。丢入`IDA`分析一手。

![](https://s0rry-1308583710.cos.ap-chengdu.myqcloud.com/markdown/202203301821262.png)

分析出大致逻辑，先输入一组数据进入一个函数加密，判断正确后，由输入的数据得到一个秘钥，带上这组秘钥又输入一组数据进入一个函数加密，再将加密后的数据再进行了多次不可逆的异或加密，最后比较是否成功。

### 分析函数

进入第一个函数分析了一下，没看出这是一个不带S盒的`DES`加密，

![](https://s0rry-1308583710.cos.ap-chengdu.myqcloud.com/markdown/202203301821610.png)

后来知道这是一个`DES`之后就有会了，由于`DES`加密算法是对合算法，也就是说`DES`解密和加密可以共用同一运算(只是子密钥使用的顺序不同)，所以我们直接写解题脚本：

但是要注意的是`IDA`的这个函数是指循环移动一个数，比如一个数是12345循环向右移动2位就是45123

![](https://s0rry-1308583710.cos.ap-chengdu.myqcloud.com/markdown/202203301822253.png)

```
#include 
#include 
unsigned ROL4(unsigned int value, unsigned int count)
{
  const unsigned int nbits = sizeof(unsigned int) * 8;
  count %= nbits;

  unsigned int low = value >> (nbits - count);
  value = 4; i-- )
    input[i - 4] = input[i] ^ ( fun(input[i - 3] ^ input[i - 2] ^ input[i - 1] ^ key[i - 4]) );
    int t[4];
    for(int i=0;i
```

当我们第一个输入确定之后，那么

![](https://s0rry-1308583710.cos.ap-chengdu.myqcloud.com/markdown/202203301822732.png)

这个就是一个固定值我们直接动态调试就可以出来。

第二个加密函数

同样的我也没能看出来是简化的`AES`加密，继续分析，这里比较复杂，得到`AES`加密后的数据，需要先出下面的异或加密，这个异或加密了次，破坏了加密的结构，所以没办法直接计算出来，所以我们得爆破它

![](https://s0rry-1308583710.cos.ap-chengdu.myqcloud.com/markdown/202203301822574.png)

爆破的函数如下，这里还是用了递归的方式来爆破的

```
unsigned char flag[24]={0,};

unsigned char flags[40][24]={0,};// 由于我们不能确定爆破出来的哪一个是真正的答案，所以需要储存多个答案去尝试

unsigned int result[24] = {

    0x00000025, 0x00000015, 0x000000DF, 0x000000A2, 0x000000C0, 0x00000093, 0x000000AD, 0x00000014, 

    0x00000046, 0x000000C5, 0x0000000F, 0x0000002E, 0x0000009A, 0x000000EB, 0x00000030, 0x000000F8, 

    0x00000020, 0x000000E9, 0x000000CB, 0x00000088, 0x000000C6, 0x000000BE, 0x0000008D, 0x000000E3

};

int x=0;

void fun(unsigned char flag[], int n){

    int i=0;

    if(n==0){

        for(i=0;i
```

接着就可以开始逆向这个`AES`加密了,这里我直接结合上面写出完整的代码：

```
#include 

unsigned char flag[24]={0,};
unsigned char flag1[40][24]={0,};
int x=0;
unsigned int result[24] = {
    0x00000025, 0x00000015, 0x000000DF, 0x000000A2, 0x000000C0, 0x00000093, 0x000000AD, 0x00000014, 
    0x00000046, 0x000000C5, 0x0000000F, 0x0000002E, 0x0000009A, 0x000000EB, 0x00000030, 0x000000F8, 
    0x00000020, 0x000000E9, 0x000000CB, 0x00000088, 0x000000C6, 0x000000BE, 0x0000008D, 0x000000E3
};
unsigned int key[10] = {
    0x01000000, 0x02000000, 0x04000000, 0x08000000, 0x10000000, 0x20000000, 0x40000000, 0x80000000, 
    0x1B000000, 0x36000000
};
void fun(unsigned char flag[], int n){

    int i=0;
    if(n==0){
        for(i=0;i>24)&0x000000ff;
    tmp[1]=(t>>16)&0x000000ff;
    tmp[2]=(t>>8)&0x000000ff;
    tmp[3]=t&0x000000ff;
    int tmp_tmp[4]={0};
    for(int i=0;i=6;k--){
            if(k%6){
                t[k-6]=t[k]^t[k-1];
            }
            else{
                t[k-6]=t[k]^T(t[k-1],num);
                num--;
            }
        }
        unsigned char * p = (unsigned char *)t;
        for ( int i = 0; i 
```

输出结果

![](https://s0rry-1308583710.cos.ap-chengdu.myqcloud.com/markdown/202203301822901.png)

`VNCTF{TimeFlightMachine}`这还不是最后的结果，我们得把得到的这些数据输回题目中

![](https://s0rry-1308583710.cos.ap-chengdu.myqcloud.com/markdown/202203301822224.png)

最后得出flag：`VNCTF{TimeFl20211205ightMachine}`

## CM狗

这道题运用了`go`语言的`vm`技术模拟了一个`tea`加密

### 首先

![](https://s0rry-1308583710.cos.ap-chengdu.myqcloud.com/markdown/202203301822758.png)

无壳且64位。丢入`IDA`分析

![](https://s0rry-1308583710.cos.ap-chengdu.myqcloud.com/markdown/202203301822008.png)

还好`go`没去符号直接找到`main`函数进去看看

![](https://s0rry-1308583710.cos.ap-chengdu.myqcloud.com/markdown/202203301822087.png)

只有两个有用的函数，我们直接看`run`函数，我弄`vm`是通过直接看汇编，然后在每个指令的前面下断来分析哪个函数是哪个指令，找到寄存器位置，以及分清操作数与操作符。

![](https://s0rry-1308583710.cos.ap-chengdu.myqcloud.com/markdown/202203301822364.png)

这是刚进入`run`函数的时候，经过分析发现大致过程就是先取一个操作符，然后拿两个操作数出来，跳转到上面去运行。

依次在每一个函数前面下断点

![](https://s0rry-1308583710.cos.ap-chengdu.myqcloud.com/markdown/202203301822385.png)

动态调试起来判断每一个函数代表的汇编指令，但是有几个函数在程序结束的时候都没有用到，这是中间有提前退出了，我们只能通过静态分析，去弄，虽然有点麻烦但是还是可以解决。分析出每个函数代表的汇编指令入下

![](https://s0rry-1308583710.cos.ap-chengdu.myqcloud.com/markdown/202203301822455.png)

我这个分析的太粗糙了，没有发现数字之间的内在的联系，这里的`a`，`b`，`c`与题中给的操作数是有关系的，我这样分析出来导致了我没办法进行下一步的操作。这里我抄`Mas0n`大佬的`wp`。

```
from string import printable

_m = list(printable.encode())

_m.remove(10)

_m.remove(13)

_hex = lambda _d: "0x"+hex(_d)[2:].zfill(2) if _d > 15 else _d

_chr = lambda _v: hex(_v) if _v not in _m else "'{ch}'".format(ch=chr(_v))

maps = {

    0x1: "nop",

    0x2: "mov R[{num}], {right} ; {ch}",

    0x3: "mov R[{num}], R[{right}]",

    0x4: "mov R[{num}], Memory[{right}]",

    0x5: "mov Memory[num] R[{right}]",

    0x6: "push R[{num}]",

    0x7: "pop R[{num}]",

    0x8: "add R[{num}], R[{right}]",

    0x9: "dec R[{num}], R[{right}]",

    0xa: "div R[{num}], R[{right}]",

    0xb: "mul R[{num}], R[{right}]",

    0xc: "xor R[{num}], R[{right}]",

    0xd: "jmp R[{num}]",

    0xe: "cmp R[{num}] R[{right}], je R[19]",

    0xf: "cmp R[{num}] R[{right}], jne R[19]",

    0x10: "cmp R[{num}] R[{right}], jb R[19]",

    0x11: "cmp R[{num}] R[{right}], ja R[19]",

    0x62: "getchar R[{num}]",

    0x63: "putchar R[{num}]",

    0x64: "vm quit"

}
```
