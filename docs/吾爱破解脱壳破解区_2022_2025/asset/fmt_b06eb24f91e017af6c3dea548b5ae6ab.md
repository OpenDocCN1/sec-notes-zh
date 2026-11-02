
```cpp
char __fastcall Watermark::Compile(_QWORD *this)
{
  _QWORD *mask_; // rdi
  __int64 size_value_; // rax (value_.size())
  unsigned __int64 i; // rsi (iteration variable)
  unsigned __int64 p; // r15 (i / 2, index for dump_ and mask_)
  unsigned __int64 new_size; // rax (used during vector expansion)
  unsigned __int64 dump_condition; // rcx (condition for growth check)
  char *value_pointer; // r14
  _BYTE *dump_bytes; // rdx
  _BYTE *mask_bytes; // rax
  unsigned __int64 mask_new_size; // rax (used during vector expansion)
  char *mask_pointer; // r14
  _BYTE *mask_data; // rdx
  _BYTE *dump_expansion; // rax
  char mask_byte; // bp (temporary bit mask)
  _QWORD *value_data; // rax (pointer to value_ data)
  char c_value; // al (current character in value_)
  char hex_value; // al (hexadecimal representation for current character)
  __int64 dump_offset; // rdx (offset into dump_)
  char current_byte; // cl (current byte in dump_)
  char v22[8]; // [rsp+50h] [rbp+8h] BYREF (used for temporary growth checking)

  mask_ = this + 16;
  this[14] = this[13];
  this[17] = this[16];
  size_value_ = this[9];
  if ( size_value_ )
  {
    i = 0i64;
    do
    {
      p = i >> 1;
      if ( p >= this[14] - this[13] )
      {
        new_size = this[14];
        v22[0] = 0;
        if ( (unsigned __int64)v22 >= new_size || (dump_condition = this[13], dump_condition > (unsigned __int64)v22) )
        {
          if ( new_size == this[15] )
            ResizeDump(this + 13, 1i64);
          mask_bytes = (_BYTE *)this[14];
          if ( mask_bytes )
            *mask_bytes = 0;
        }
        else
        {
          value_pointer = &v22[-dump_condition];
          if ( new_size == this[15] )
            ResizeDump(this + 13, 1i64);
          dump_bytes = (_BYTE *)this[14];
          if ( dump_bytes )
            *dump_bytes = value_pointer[this[13]];
        }
        ++this[14];
        mask_new_size = mask_[1];
        v22[0] = 0;
        if ( (unsigned __int64)v22 >= mask_new_size || *mask_ > (unsigned __int64)v22 )
        {
          if ( mask_new_size == mask_[2] )
            ResizeMask(mask_, 1i64);
          dump_expansion = (_BYTE *)mask_[1];
          if ( dump_expansion )
            *dump_expansion = 0;
        }
        else
        {
          mask_pointer = &v22[-*mask_];
          if ( mask_new_size == mask_[2] )
            ResizeMask(mask_, 1i64);
          mask_data = (_BYTE *)mask_[1];
          if ( mask_data )
            *mask_data = mask_pointer[*mask_];
        }
        ++mask_[1];
      }
      mask_byte = -1;
      if ( this[10]  9u )
      {
        if ( (unsigned __int8)(c_value - 65) > 5u )
        {
          if ( (unsigned __int8)(c_value - 97) > 5u )
          {
            mask_byte = 0;
            hex_value = RandomFunction();
          }
          else
          {
            hex_value = c_value - 87;
          }
        }
        else
        {
          hex_value = c_value - 55;
        }
      }
      else
      {
        hex_value = c_value - 48;
      }
      dump_offset = this[13];
      current_byte = *(_BYTE *)(dump_offset + p);
      if ( (i & 1) != 0 )
      {
        *(_BYTE *)(dump_offset + p) ^= (hex_value ^ current_byte) & 0xF;
        LOBYTE(size_value_) = (mask_byte ^ *(_BYTE *)(*mask_ + p)) & 0xF;
        *(_BYTE *)(*mask_ + p) ^= size_value_;
      }
      else
      {
        *(_BYTE *)(dump_offset + p) = (16 * hex_value) | current_byte & 0xF;
        LOBYTE(size_value_) = (16 * mask_byte) | *(_BYTE *)(*mask_ + p) & 0xF;
        *(_BYTE *)(*mask_ + p) = size_value_;
      }
      ++i;
    }
    while ( i 
```

其中的 `value_data` 就是我们想找的水印字符串的指针，对应汇编为

```asm
.text:00007FF74DE06F61 loc_7FF74DE06F61:                       ; CODE XREF: Watermark__Compile+71↑j
.text:00007FF74DE06F61                 cmp     qword ptr [r13+50h], 10h
.text:00007FF74DE06F66                 mov     bpl, 0FFh
.text:00007FF74DE06F69                 jb      short loc_7FF74DE06F71
.text:00007FF74DE06F6B                 mov     rax, [r13+38h]
.text:00007FF74DE06F6F                 jmp     short loc_7FF74DE06F75
.text:00007FF74DE06F71 ; ---------------------------------------------------------------------------
.text:00007FF74DE06F71
.text:00007FF74DE06F71 loc_7FF74DE06F71:                       ; CODE XREF: Watermark__Compile+169↑j
.text:00007FF74DE06F71                 lea     rax, [r13+38h]
```

其中 `[r13+0x38]` 为从实例指针(`this`)+偏移量中取出水印字符串指针赋值给 `rax`，既 `value_data`

进一步追寻 `r13` 的来源，在函数开头发现了这一行

```asm
.text:00007FF74DE06E18                 mov     r13, rcx
```

根据 Windows 平台 x64 下的 `fastcall` 调用约定，第一个传入参数为 `rcx`，既 `this` 指针，因此我们在 `Watermark::Compile` 函数开头直接使用 `[rcx+0x38]` 就可以获取水印字符串指针

注: 关于 x64 调用约定，可参考微软官方文档 [//learn.microsoft.com/zh-cn/cpp/build/x64-calling-convention?view=msvc-170#calling-convention-defaults" target="_blank" rel="noopener noreferrer nofollow"](https://learn.microsoft.com/zh-cn/cpp/build/x64-calling-convention?view=msvc-170#calling-convention-defaults:rel="nofollow noopener" href="https)

在调试器内下断点，获取水印字符串指针，直接看内存获得水印具体内容

![](https://attach.52pojie.cn/forum/202312/16/165104avmluzppulgpuukv.png)

image-5.png (114.26 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY2NDQ1Mnw4MTc0ODIxNnwxNzkzNTcyODMwfDE2ODA4NnwxODY5NzU4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2NDQ1Mnw4MTc0ODIxNnwxNzkzNTcyODMwfDE2ODA4NnwxODY5NzU4&nothumb=yes)

2023-12-16 16:51 上传

```
0000028A4FC35E60  35 30 46 30 31 46 46 44 46 44 38 3F 3F 37 39 32  50F01FFDFD8??792  
0000028A4FC35E70  36 3F 3F 3F 42 34 3F 3F 43 32 3F 3F 3F 30 37 3F  6???B4??C2???07?  
0000028A4FC35E80  34 3F 3F 3F 3F 3F 43 3F 43 3F 3F 46 3F 44 32 3F  4?????C?C??F?D2?  
0000028A4FC35E90  36 3F 3F 31 39 43 42 46 30 3F 39 39 31 32 3F 37  6??19CBF0?9912?7  
0000028A4FC35EA0  31 37 3F 3F 33 36 33 35 43 41 38 41 3F 37 3F 30  17??3635CA8A?7?0  
0000028A4FC35EB0  3F 3F 3F 46 3F 43 3F 44 37 44 37 3F 3F 39 45 35  ???F?C?D7D7??9E5  
0000028A4FC35EC0  3F 31 3F 38 34 45 34 3F 3F 3F 32 34 3F 3F 44 34  ?1?84E4???24??D4  
0000028A4FC35ED0  35 3F 35 3F 43 3F 30 34 42 39 45 3F 44 3F 32 3F  5?5?C?04B9E?D?2?  
0000028A4FC35EE0  31 35 3F 38 39 3F 3F 36 3F 37 38 34 3F 3F 3F 3F  15?89??6?784????  
0000028A4FC35EF0  3F 44 39 3F 3F 31 3F 31 3F 45 3F 3F 30 33 3F 3F  ?D9??1?1?E??03??  
0000028A4FC35F00  3F 3F 3F 34 34 36 3F 36 3F 3F 3F 33 45 43 39 34  ???446?6???3EC94  
0000028A4FC35F10  31 45 3F 36 41 3F 3F 34 3F 35 3F 3F 3F 3F 3F 3F  1E?6A??4?5??????  
0000028A4FC35F20  3F 3F 38 3F 43 3F 3F 38 3F 3F 3F 32 3F 3F 3F 30  ??8?C??8???2???0  
0000028A4FC35F30  43 38 45 42 3F 43 31 3F 44 3F 34 3F 00           C8EB?C1?D?4?.     
```

其中的 `?` 为随机内容，疑似为保留随机性的行为，同时，这里的内容其实为十六进制表示的二进制内容，所以我们在搜索的时候需要将程序转化为 hex 形式的字符串再进行搜索，概念验证代码如下
