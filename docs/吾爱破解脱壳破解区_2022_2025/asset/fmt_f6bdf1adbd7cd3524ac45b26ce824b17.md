
```
vdq::get_opr_lst::h470c4d46db5f8252(&v0);//读取opr

vdq::handle_opr_lst::h7fb2393547b96358(v1.buf.alloc.gap0);//处理opr
```

进入 `get_opr_lst` 之后，注意到如下代码：

```
  core::result::Result,serde_json::error::Error> v29;
  serde_json::de::from_str::h2ed086b1a84205ca(&v29, v12);
```

因此我们就可以推测，程序提供了一个反序列化服务，`v29` 是其对象。

接下来就向下搜索反序列化的关键字和翻译格式。

顺着如下函数向下搜索：

```
vdq::get_opr_lst::h470c4d46db5f8252(&v0);
serde_json::de::from_str::h2ed086b1a84205ca(&v29, v12);
serde_json::de::from_trait::h010df4f45829b4ad(retstr, read);
serde::de::impls::_$LT$impl$u20$serde..de..Deserialize$u20$for$u20$alloc..vec..Vec$LT$T$GT$$GT$::deserialize::h3d140fae89f3cb33
_$LT$$RF$mut$u20$serde_json..de..Deserializer$LT$R$GT$$u20$as$u20$serde..de..Deserializer$GT$::deserialize_seq::hbd3934c1f9eb2161
_$LT$serde..de..impls..$LT$impl$u20$serde..de..Deserialize$u20$for$u20$alloc..vec..Vec$LT$T$GT$$GT$..deserialize..VecVisitor$LT$T$GT$$u20$as$u20$serde..de..Visitor$GT$::visit_seq::h004d517e1abba1bd
serde::de::SeqAccess::next_element::h66a6a37c3fe5b12c
_$LT$serde_json..de..SeqAccess$LT$R$GT$$u20$as$u20$serde..de..SeqAccess$GT$::next_element_seed::hdf4677aba76d625b
_$LT$core..marker..PhantomData$LT$T$GT$$u20$as$u20$serde..de..DeserializeSeed$GT$::deserialize::ha3e4760fc98c681a
vdq::_::_$LT$impl$u20$serde..de..Deserialize$u20$for$u20$vdq..Operation$GT$::deserialize::h5d3aaf882e3017d5
_$LT$$RF$mut$u20$serde_json..de..Deserializer$LT$R$GT$$u20$as$u20$serde..de..Deserializer$GT$::deserialize_enum::h4c7b4e2d01c35d8
_$LT$vdq.._..$LT$impl$u20$serde..de..Deserialize$u20$for$u20$vdq..Operation$GT$..deserialize..__Visitor$u20$as$u20$serde..de..Visitor$GT$::visit_enum::he6941ccdf9c46f1c
serde::de::EnumAccess::variant::hc394608857e1e375
_$LT$serde_json..de..UnitVariantAccess$LT$R$GT$$u20$as$u20$serde..de..EnumAccess$GT$::variant_seed::h3111f0a59a2c8909
_$LT$core..marker..PhantomData$LT$T$GT$$u20$as$u20$serde..de..DeserializeSeed$GT$::deserialize::hae2cb777484d7d0f
_$LT$vdq.._..$LT$impl$u20$serde..de..Deserialize$u20$for$u20$vdq..Operation$GT$..deserialize..__Field$u20$as$u20$serde..de..Deserialize$GT$::deserialize::h771926e8bf89d42b
_$LT$$RF$mut$u20$serde_json..de..Deserializer$LT$R$GT$$u20$as$u20$serde..de..Deserializer$GT$::deserialize_identifier::h043dc575c5a1b557
_$LT$$RF$mut$u20$serde_json..de..Deserializer$LT$R$GT$$u20$as$u20$serde..de..Deserializer$GT$::deserialize_str::h8ad76558a0a689aa
_$LT$vdq.._..$LT$impl$u20$serde..de..Deserialize$u20$for$u20$vdq..Operation$GT$..deserialize..__FieldVisitor$u20$as$u20$serde..de..Visitor$GT$::visit_str::h9d16723e30de37b2
```

最终能够在最后一个函数处找到解析关键字：

```
core::result::Result *__cdecl _$LT$vdq.._..$LT$impl$u20$serde..de..Deserialize$u20$for$u20$vdq..Operation$GT$..deserialize..__FieldVisitor$u20$as$u20$serde..de..Visitor$GT$::visit_str::h9d16723e30de37b2(core::result::Result *retstr, vdq::_::_{impl}}::deserialize::__FieldVisitor self, _str __value)
{
  _str v3; // rdx
  _str v4; // rdx
  _str v5; // rdx
  _str v6; // rdx
  _str v7; // rdx
  unsigned __int64 v8; // r8
  unsigned __int64 v9; // rsi
  core::result::Result *v11; // [rsp+28h] [rbp-30h]

  v3.data_ptr = &unk_62AB2;                     // add
  v3.length = 3LL;
  if ( core::str::traits::_$LT$impl$u20$core..cmp..PartialEq$u20$for$u20$str$GT$::eq::he9d7a829c76bba8b(*&retstr, v3) )
  {
    LOWORD(v11) = 0;
  }
  else
  {
    v4.data_ptr = &byte_62AB5;                  // remove
    v4.length = 6LL;
    if ( core::str::traits::_$LT$impl$u20$core..cmp..PartialEq$u20$for$u20$str$GT$::eq::he9d7a829c76bba8b(*&retstr, v4) )
    {
      LOWORD(v11) = 256;
    }
    else
    {
      v5.data_ptr = &unk_62ABB;                 // append
      v5.length = 6LL;
      if ( core::str::traits::_$LT$impl$u20$core..cmp..PartialEq$u20$for$u20$str$GT$::eq::he9d7a829c76bba8b(
             *&retstr,
             v5) )
      {
        LOWORD(v11) = 512;
      }
      else
      {
        v6.data_ptr = &unk_62AC1;               // archive
        v6.length = 7LL;
        if ( core::str::traits::_$LT$impl$u20$core..cmp..PartialEq$u20$for$u20$str$GT$::eq::he9d7a829c76bba8b(
               *&retstr,
               v6) )
        {
          LOWORD(v11) = 768;
        }
        else
        {
          v7.data_ptr = &unk_62AA4;             // view
          v7.length = 4LL;
          if ( core::str::traits::_$LT$impl$u20$core..cmp..PartialEq$u20$for$u20$str$GT$::eq::he9d7a829c76bba8b(
                 *&retstr,
                 v7) )
          {
            LOWORD(v11) = 1024;
          }
          else
          {
            serde::de::Error::unknown_variant::hc8291a7390e93cb5(
              retstr,
              __PAIR128__(&off_7BD80, v9),
              __PAIR128__(v8, (&stru_2._marker + 3)));
            LOBYTE(v11) = 1;
          }
        }
      }
    }
  }
  return v11;
}
```

不过需要注意，rust 编译后的字符串相互连接，通过长度来确定具体的字符串内容；而 IDA 的分析会将整个字符串一并解析，以至于难以准确理解代码，具体表现如下：

```
v3.data_ptr = &unk_62AB2;                     // add
v3.length = 3LL;
v4.data_ptr = &byte_62AB5;                  // remove
v4.length = 6LL;
```

字符串在 IDA 中的样式：

```
unsigned char ida_chars[] ={
0x41, 0x64, 0x64, 0x52, 0x65, 0x6D, 0x6F, 0x76, 0x65, 0x41, 0x70,
0x70, 0x65, 0x6E, 0x64};
//AddRemoveAppend
```

根据上述函数能够分析出具体有哪些操作：

```
Add、Remove、Append、Archive、View
```

并且继续向上跟踪，可以知道其输入格式是：

(事实上如果熟悉反序列化就不用苦恼了，不过笔者也试着搜索过，搜出格式以后直接套也行，不过难道有这种机会，还是试着逆了一下)

```
["Add","Add","Remove"]
$
```

在知道具体的输入以后，就可以尝试 `fuzz` 来进行输入测试了。

但笔者不得不在这里提一句，如果您熟悉 rust 中的 `enum` 或实际拥有编译条件的话，在如下函数就能直接找到答案，不需要一步步深入：

```
// local variable allocation has failed, the output may be wrong!
core::result::Result *__cdecl vdq::_::_$LT$impl$u20$serde..de..Deserialize$u20$for$u20$vdq..Operation$GT$::deserialize::h5d3aaf882e3017d5(core::result::Result *retstr, serde_json::de::Deserializer *__deserializer)
{
  __int64 v2; // r9 OVERLAPPED
  _str v3; // rdx
  core::marker::PhantomData *v4; // r8
  vdq::_::_{impl}}::deserialize::__Visitor v6; // [rsp+0h] [rbp-38h]

  v3.length = &off_7BD80;
  v4 = &stru_2._marker + 3;
  v3.data_ptr = (&stru_2 + 7);
  return _$LT$$RF$mut$u20$serde_json..de..Deserializer$LT$R$GT$$u20$as$u20$serde..de..Deserializer$GT$::deserialize_enum::h4c7b4e2d01c35d85(retstr,&unk_62AA9,v3,*(&v2 - 1),v6);
}
```

阅读函数命 `deserialize_enum` 大概能够知道这是 rust 编译后的 `enum` 表示函数，`unk_62AA9` 是其中的数据：

```
0000000000062AA9 aOperationaddre db 'OperationAddRemoveAppendArchive'
```

结合 `handle_opr_lst` 可知对应的代码应该是：

```
enum vdq::Operation : __int8 {
    Add = 0x0,
    Remove = 0x1,
    Append = 0x2,
    Archive = 0x3,
    View = 0x4
};
```

## 5. 模糊测试

这里参考一下cj神的方法：

```
# fuzz.sh
#!/bin/bash
while ((1))
do 
    python ./vdq_input_gen.py > poc
    cat poc | ./vdq
    if [ $? -ne 0 ]; then
        break
    fi
done
```

```
# vdq_input_gen.py
#!/usr/bin/env python
# coding=utf-8

import random
import string

operations = "["

def Add():
    global operations
    operations += "\"Add\", "

def Remove():
    global operations
    operations += "\"Remove\", "

def Append():
    global operations
    operations += "\"Append\", "

def View():
    global operations
    operations += "\"View\", "

def Archive():
    global operations
    operations += "\"Archive\", "

def DoOperations():
    print(operations[:-2] + "]")
    print("$")

def DoAdd(message):
    print(message)

def DoAppend(message):
    print(message)

total_ops = random.randint(1, 20)
total_adds = 0
total_append = 0
total_remove = 0
total_message = 0

for i in range(total_ops):
    op = random.randint(0, 4)
    if op == 0:
        total_message += 1
        total_adds += 1
        Add()
    elif op == 1:
        total_adds -= 1
        Remove()
    elif op == 2:
        if total_adds > 0:
            total_append += 1
            total_message += 1
            Append()
        Append()
    elif op == 3:
        total_adds = 0
        total_append = 0
        total_remove = 0
        Archive()
    elif op == 4:
        View()

DoOperations()

for i in range(total_message):
```
