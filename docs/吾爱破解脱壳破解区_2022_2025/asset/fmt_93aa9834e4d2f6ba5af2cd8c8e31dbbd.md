
```
void __fastcall __noreturn main(__int64 a1, char **a2, char **a3)
{
  __int64 v3; // rax
  __int64 v4; // rax
  __int64 v5; // rax
  __int64 v6; // rax
  int id; // [rsp+0h] [rbp-420h]
  int new_id; // [rsp+4h] [rbp-41Ch]
  Info *info; // [rsp+8h] [rbp-418h]
  Info info1; // [rsp+10h] [rbp-410h] BYREF
  Info info2; // [rsp+160h] [rbp-2C0h] BYREF
  Info info3; // [rsp+2B0h] [rbp-170h] BYREF
  unsigned __int64 v13; // [rsp+408h] [rbp-18h]

  v13 = __readfsqword(0x28u);
  init_state();
  welcome();
  init_info1(&info1);
  init_info2(&info2);
  init_info3(&info3);
  id = 1;
  v3 = std::operator>(&std::cout, "Peppa Pig first~");
  std::ostream::operator>);
  info = &info1;
  get_info1(&info1);
  while ( 1 )
  {
    menu();
    switch ( get_num() )
    {
      case 1:
        add(info, id);
        break;
      case 2:
        show(info, id);
        break;
      case 3:
        edit(info, id);
        break;
      case 4:
        delete(info, id);
        break;
      case 5:
        new_id = login();
        if ( new_id && new_id != id )
        {
          switch ( id )
          {
            case 1:
              set_info1(info);
              break;
            case 2:
              set_info2(info);
              break;
            case 3:
              set_info3(info);
              break;
          }
          id = new_id;
          switch ( new_id )
          {
            case 1:
              v4 = std::operator>(&std::cout, "This is Peppa Pig~");
              std::ostream::operator>);
              info = &info1;
              get_info1(&info1);
              break;
            case 2:
              v5 = std::operator>(&std::cout, "This is Mummy Pig~");
              std::ostream::operator>);
              info = &info2;
              get_info2(&info2);
              break;
            case 3:
              v6 = std::operator>(&std::cout, "This is Daddy Pig~");
              std::ostream::operator>);
              info = &info3;
              get_info3(&info3);
              break;
          }
        }
        break;
      default:
        puts("Invalid...");
        break;
    }
  }
}
```

一道 c++ 的 pwn 题。三只猪用户，一共五种操作，添加，查看，修改，删除，登录。最开始默认 peppa（猪A）先操作，猪A的 `id=1`，Mummy（猪B）的 `id=2`，Daddy（猪C）的 `id=3`。

```
unsigned __int64 __fastcall add(Info *info, int id)
{
  unsigned __int64 v3; // [rsp+18h] [rbp-8h]

  v3 = __readfsqword(0x28u);
  switch ( id )
  {
    case 1:
      add_1(info);
      break;
    case 2:
      add_2(info);
      break;
    case 3:
      add_3(info);
      break;
  }
  return __readfsqword(0x28u) ^ v3;
}

unsigned __int64 __fastcall add_1(Info *info)
{
  __int64 v1; // rax
  __int64 v2; // rax
  __int64 v3; // rax
  __int64 v4; // rax
  int i; // [rsp+18h] [rbp-18h]
  int j; // [rsp+1Ch] [rbp-14h]
  int size; // [rsp+20h] [rbp-10h]
  unsigned __int64 v9; // [rsp+28h] [rbp-8h]

  v9 = __readfsqword(0x28u);
  for ( i = 0; i ptr[i]; ++i )
    ;
  if ( i == 20 )
  {
    v1 = std::operator>(&std::cout, "Message is full!");
    std::ostream::operator>);
  }
  else
  {
    if ( state->min_size1 min_size1 = 0x90;
    std::operator>(&std::cout, "Input the message size: ");
    size = get_num();
    if ( size >= state->min_size1 && size min_size1 = size;
      info->ptr[i] = (char *)calloc(1uLL, size);
      if ( !info->ptr[i] )
      {
        v3 = std::operator>(&std::cout, "Error calloc!");
        std::ostream::operator>);
        exit(-1);
      }
      info->size[i] = size;
      info->flag1[i] = 0;
      info->flag2[i] = 0;
      std::operator>(&std::cout, "Input the Peppa's message: ");
      for ( j = 0; j ptr[i][0x30 * j], 0x10LL);
      v4 = std::operator>(&std::cout, "Success!");
      std::ostream::operator>);
    }
    else
    {
      v2 = std::operator>(&std::cout, "Error size!");
      std::ostream::operator>);
    }
  }
  return __readfsqword(0x28u) ^ v9;
}

unsigned __int64 __fastcall add_2(Info *info)
{
  __int64 v1; // rax
  __int64 v2; // rax
  __int64 v3; // rax
  __int64 v4; // rax
  int i; // [rsp+18h] [rbp-18h]
  int j; // [rsp+1Ch] [rbp-14h]
  int size; // [rsp+20h] [rbp-10h]
  unsigned __int64 v9; // [rsp+28h] [rbp-8h]

  v9 = __readfsqword(0x28u);
  for ( i = 0; i ptr[i]; ++i )
    ;
  if ( i == 10 )
  {
    v1 = std::operator>(&std::cout, "Message is full!");
    std::ostream::operator>);
  }
  else
  {
    if ( state->min_size2 min_size2 = 0x90;
    std::operator>(&std::cout, "Input the message size: ");
    size = get_num();
    if ( size >= state->min_size2 && size min_size2 = size;
      info->ptr[i] = (char *)calloc(1uLL, size);
      if ( !info->ptr[i] )
      {
        v3 = std::operator>(&std::cout, "Error calloc!");
        std::ostream::operator>);
        exit(-1);
      }
      info->size[i] = size;
      info->flag1[i] = 0;
      info->flag2[i] = 0;
      std::operator>(&std::cout, "Input the Mummy's message: ");
      for ( j = 0; j ptr[i][0x30 * j + 0x10], 0x10LL);
      v4 = std::operator>(&std::cout, "Success!");
      std::ostream::operator>);
    }
    else
    {
      v2 = std::operator>(&std::cout, "Error size!");
      std::ostream::operator>);
    }
  }
  return __readfsqword(0x28u) ^ v9;
}

unsigned __int64 __fastcall add_3(Info *info)
{
  __int64 v1; // rax
  __int64 v2; // rax
  __int64 v3; // rax
  __int64 v4; // rax
  __int64 v5; // rax
  __int64 v6; // rax
  int i; // [rsp+10h] [rbp-20h]
  int j; // [rsp+14h] [rbp-1Ch]
  int size; // [rsp+18h] [rbp-18h]
  _BYTE *v11; // [rsp+20h] [rbp-10h]
  unsigned __int64 v12; // [rsp+28h] [rbp-8h]

  v12 = __readfsqword(0x28u);
  for ( i = 0; i ptr[i]; ++i )
    ;
  if ( i == 5 )
  {
    v1 = std::operator>(&std::cout, "Message is full!");
    std::ostream::operator>);
  }
  else
  {
    if ( state->min_size3 min_size3 = 0x90;
    std::operator>(&std::cout, "Input the message size: ");
    size = get_num();
    if ( size > 0x8F && size min_size3 = size;
      info->ptr[i] = (char *)calloc(1uLL, size);
      if ( !info->ptr[i] )
      {
        v3 = std::operator>(&std::cout, "Error calloc!");
        std::ostream::operator>);
        exit(-1);
      }
      info->size[i] = size;
      info->flag1[i] = 0;
      info->flag2[i] = 0;
      std::operator>(&std::cout, "Input the Daddy's message: ");
      for ( j = 0; j ptr[i][0x30 * j + 0x20], 0x10LL);
      v4 = std::operator>(&std::cout, "Success!");
      std::ostream::operator>);
      if ( i == 4 )
      {
        v11 = calloc(1uLL, 0xE8uLL);
        v5 = std::operator>(&std::cout, "01dwang's Gift:");
        std::ostream::operator>);
        read_n(v11, 0xE8LL);
        v6 = std::operator>(&std::cout, "Success!");
        std::ostream::operator>);
      }
    }
    else
    {
      v2 = std::operator>(&std::cout, "Error size!");
      std::ostream::operator>);
    }
  }
  return __readfsqword(0x28u) ^ v12;
}
```

添加操作一共有三种，分别对应猪A、猪B、猪C。猪A可以遍历 0~19 的索引，并添加一个大小在 0x90~0x430 的 chunk，猪B只能遍历 0~9 的索引，并添加大小在 0x90~0x450 的 chunk。对于猪C，则是 0~4 的索引和 0x90~0x440 的 chunk。猪A、猪B分配的 chunk 大小只能一次比一次大或者本次与上一次相等，但猪C没有这个限制。另外，在猪C函数中如果添加 chunk 的索引为 4，则还可以再分配一个大小为 0xE8 的 chunk 并写入最大长度为 0xE8 的内容。

3只猪在 `add` 之后可以立即向新分配的 chunk 中写入内容，但不是 chunk 中任何位置都能写，chunk 空间以 48 字节为大小分组。对于猪A，每一组 48 字节空间只能写前面 16 字节，对于猪B则是只能写中间 16 字节，对于猪C只能写后面 16 字节。在写入后，会设置两个标志位为 0。

```
unsigned __int64 __fastcall show(Info *info, int id)
{
  __int64 v2; // rax
  unsigned __int64 v4; // [rsp+18h] [rbp-8h]

  v4 = __readfsqword(0x28u);
  if ( state->show_times >(&std::cout, "No view for you...");
    std::ostream::operator>);
  }
  else
  {
    switch ( id )
    {
      case 1:
        show_1(info);
        break;
      case 2:
        show_2(info);
        break;
      case 3:
        show_3(info);
        break;
    }
    --state->show_times;
  }
  return __readfsqword(0x28u) ^ v4;
}

unsigned __int64 __fastcall show_1(Info *a1)
{
  __int64 v1; // rax
  __int64 v2; // rax
  unsigned int index; // [rsp+14h] [rbp-Ch]
  unsigned __int64 v5; // [rsp+18h] [rbp-8h]
```
