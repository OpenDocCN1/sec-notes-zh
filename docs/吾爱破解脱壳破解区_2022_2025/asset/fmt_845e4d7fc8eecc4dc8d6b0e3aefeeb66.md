
```asm
# 验证完长度是40后
# 初始位置XXXX
mov dword ptr ss:[rsp+0x24], 0x0                        
cmp dword ptr ss:[rsp+0x24], 0x20                                # i  [0x00000001400073B0]                

mov rax, qword ptr ss:[rsp+0x30]                                # 取出 00000001400073B0 给 rax
movsxd rax, dword ptr ds:[rax]                        
imul rax, rax, 0x1C                                                                                

lea rcx, ds:[0x00000001400073C0]
mov dword ptr ds:[rcx+rax*1+0x14], 0x1

mov rax, qword ptr ss:[rsp+0x30]                        # 取出 00000001400073B0 给rax
movsxd rax, dword ptr ds:[rax]        
imul rax, rax, 0x1C
lea rcx, ds:[0x00000001400073C0]
mov dword ptr ds:[rcx+rax*1+0xC], 0x1

mov rax, qword ptr ss:[rsp+0x30]        # 取出 00000001400073B0 给rax
movsxd rax, dword ptr ds:[rax]   
imul rax, rax, 0x1C
lea rcx, ds:[0x00000001400073C0]

mov edx,dword ptr ss:[rsp+0x38]        #  地址时: 0000000140016535 | 8B5424 38        取出EDX，即前面的加密后的input
mov dword ptr ds:[rcx+rax*1+0x8], edx

mov rax, qword ptr ss:[rsp+0x30]                # 取出  00000001400073B0 给 RAX
mov eax, dword ptr ds:[rax]
add eax, 0x6

lea rcx, ds:[0x000000014000E1B0]                        # 0x000000014000E1B0        是输入的原始flag的全局位置
movsx eax, byte ptr ds:[rcx+rax*1]                        # 取出 0x31 ('1') 这是自己输入的
mov rcx, qword ptr ss:[rsp+0x30]                    # 00000001400073B0 给 rcx
movsxd rcx, dword ptr ds:[rcx]
imul rcx, rcx, 0x1C

lea rdx, ds:[0x00000001400073C0]
mov dword ptr ds:[rdx+rcx*1+0x18], eax                # 把加密后的那个Input放到那个内存位置，像是一个结构体

##### 暂时不知道这一段是啥意思                (乘 0xbc8f) % 0x7FFFFFFF    谷歌搜到 # https://rvklein.me/proj/rando/rando-code.html
movsxd rax, dword ptr ds:[0x00000001400062C0]        # 0x00000001400062C0的初始值是 0x1DF2ED66   # 本条指令地址是000000014001ABCC 
imul rax, rax, 0xBC8F                #0000185591BAFD68
mov ecx, 0x7FFFFFFF                        
idiv rcx                                                # 第二轮 RAX: 00000000000030AB RDX 0000000011BB2E13(余数)  这里是生成优先级，使其二叉搜索树满足堆的性质
mov rax, rdx                                        
mov dword ptr ds:[0x00000001400062C0], eax                                
mov eax, dword ptr ds:[0x00000001400062C0]
#####

movsxd rcx, dword ptr ds:[0x00000001400073B4]
lea rdx, ds:[0x0000000140007220]
mov dword ptr ds:[rdx+rcx*4], eax                  # 000000002109B018

movsxd rax, dword ptr ds:[0x00000001400073B4]
lea rcx, ds:[0x0000000140007220]
mov rdx, qword ptr ss:[rsp+0x30]                # 00000001400073B0 给 rdx

movsxd rdx, dword ptr ds:[rdx]
imul rdx, rdx, 0x1C

lea r8, ds:[0x00000001400073C0]
mov eax, dword ptr ds:[rcx+rax*4]                
mov dword ptr ds:[r8+rdx*1+0x10], eax                # 000000002109B018

mov eax, dword ptr ds:[0x00000001400073B4]
inc eax
mov dword ptr ds:[0x00000001400073B4], eax
add rsp, 0x28

mov eax, dword ptr ss:[rsp+0x24]                # 一轮循环结束
inc eax    
mov dword ptr ss:[rsp+0x24], eax

cmp dword ptr ss:[rsp+0x24], 0x20                  # 回到了最上面的初始位置XXXX 地址：00000001400158B8

#****************************************************这是不跳的情况****************************************************
mov rax, qword ptr ss:[rsp+0x30]                # 00000001400073B0        # 地址是0000000140012B60
movsxd rax, dword ptr ds:[rax]
imul rax, rax, 0x1C                
lea rcx, ds:[0x00000001400073C0]

mov eax, dword ptr ds:[rcx+rax*1+0xC]
inc eax

mov rcx, qword ptr ss:[rsp+0x30]
movsxd rcx, dword ptr ds:[rcx]                    # 00000001400073B0
imul rcx, rcx, 0x1C
lea rdx, ds:[0x00000001400073C0]
mov dword ptr ds:[rdx+rcx*1+0xC], eax                # 变为了2
mov rax, qword ptr ss:[rsp+0x30]                        # 00000001400073B0
movsxd rax, dword ptr ds:[rax]    
imul rax, rax, 0x1C
lea rcx, ds:[0x00000001400073C0]
mov edx, dword ptr ss:[rsp+0x38]                        # 取出enc_input

cmp dword ptr ds:[rcx+rax*1+0x8], edx
je 0x0000000140011E04   

mov rax, qword ptr ss:[rsp+0x30]
movsxd rax, dword ptr ds:[rax]
imul rax, rax, 0x1C
lea rcx, ds:[0x00000001400073C0]
mov eax, dword ptr ds:[rcx+rax*1+0x8]        # 取出enc_input
cmp dword ptr ss:[rsp+0x38], eax
jg 0x0000000140017AF8

mov rax, qword ptr ss:[rsp+0x30]                # 00000001400073B0
movsxd rax, dword ptr ds:[rax]
imul rax, rax, 0x1C    
lea rcx, ds:[0x00000001400073C0]
add rcx, rax
mov rax, rcx
mov edx, dword ptr ss:[rsp+0x38]
mov rcx, rax
mov dword ptr ss:[rsp+0x10], edx
mov qword ptr ss:[rsp+0x8], rcx    
sub rsp, 0x28
mov rax, qword ptr ss:[rsp+0x30]        #00000001400073DC

cmp dword ptr ds:[rax], 0x0
je ****
mov eax, dword ptr ds:[0x00000001400073B8]
inc eax   

mov dword ptr ds:[0x00000001400073B8], eax
```

经过无尽的调试（在判断插入节点个数那里下断点，每插入一个节点，就把`0x00000001400073B0`那里的数据提取出来，进行比对）就会发现，程序是把输入的flag按照`Treap`的特性存放在了`00000001400073C0`处，这里是一个长度为33的结构体数组，(第一个数组全是0，因为序号是从1开始的，序号是0表示为空，有效的就是后面32个数组)每个单元的大小是`0x1C`

![](https://attach.52pojie.cn/forum/202203/03/110807m2as2p5koo1ts56s.png)

image-20220303101105891.png (98.08 KB, 下载次数: 248)

[href="forum.php?mod=attachment&aid=MjQ5ODk5OHwzMjQ5M2RhOHwxNzkzNTc2MDkzfDE2ODA4NnwxNTk2NTUw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5ODk5OHwzMjQ5M2RhOHwxNzkzNTc2MDkzfDE2ODA4NnwxNTk2NTUw&nothumb=yes)

2022-3-3 11:08 上传

```c
struct tree_node{

   0x00 DWORD 左孩子;                 //0表示空

   0x04 DWORD 右孩子;

   0X08 DWORD enc_input;        //加密后的每一个字符

   0X0C DWORD 叶子个数;    //假设把此节点当做根节点，整个Tree的叶子个数

   0X10 DWORD 随机数;            //优先级

   0X14 DWORD  1;                 //这个没搞懂啥意思

   0X18 DWORD input;         //输入的flag的每一个字符

}
```

在`00000001400073B0`处存放着根节点，`00000001400073B4`, `00000001400073B8`处存着当前树的所有叶子个数

输入`SUSCTF{01234567890122345679abcdefghijkl}`, 然后根据`00000001400073C0`处的数据画出自己输入的假的flag构造的二叉树

提取出`00000001400073C0`处有效的32个数组
