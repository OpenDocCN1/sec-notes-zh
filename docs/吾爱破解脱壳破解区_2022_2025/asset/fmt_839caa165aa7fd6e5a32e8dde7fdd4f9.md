
```
&#128721;0002e428  {
0002e453      sub_11014(&arg_8, "                                …", 0x67, 0x23c);
0002e458      void* rdx = *(uint64_t*)((char*)arg1 + 0xb8);
0002e45f      int64_t* rsi = *(uint64_t*)((char*)arg1 + 0x18);
0002e467      uint64_t rdi = ((uint64_t)*(uint32_t*)((char*)rdx + 0x10));
0002e46a      int32_t r8 = *(uint32_t*)((char*)rdx + 8);
0002e46e      void* r12 = *(uint64_t*)(*(uint64_t*)((char*)rdx + 0x30) + 0x18);
0002e472      int32_t rax_1 = *(uint32_t*)((char*)rdx + 0x18);
0002e475      int64_t r9 = *(uint64_t*)((char*)rdx + 0x20);
0002e479      int32_t rbp = 0;
0002e480      int32_t rbx = -0x3ffffff3;
0002e487      int64_t i_5 = 2;
0002e48b      uint64_t rax_31;
0002e48b      int32_t var_80;
0002e48b      char const* const var_78;
0002e48b      void* var_70;
0002e48b      void* var_68;
0002e48b      void* const var_60;
0002e48b      void* var_58;
0002e48b      int32_t rax_8;
0002e48b      if (rax_1 > 0x22242c)
0002e485      {
0002e62d          if (rax_1 != 0x222432)
0002e628          {
0002e636              if (rax_1 != 0x222434)
0002e633              {
0002e63f                  if (rax_1 == 0x222438)
0002e63c                  {
0002e860                      if (rdi != 0)
0002e85e                      {
0002e860                          goto label_2ea42;
0002e860                      }
0002e86a                      if (r8 != 8)
0002e866                      {
0002e86a                          goto label_2ea42;
0002e86a                      }
0002e876                      sub_188cc(r12, rsi);
0002e87b                      rbp = ((int32_t)(rdi + 8));
0002e87e                      goto label_2e6bf;
0002e87e                  }
0002e648                  if (rax_1 == 0x222440)
0002e645                  {
0002e747                      if (rdi  1))
0002e714                  {
0002e728                      sub_12b1c(&data_25cd8, rsi);
0002e72d                      int64_t i = -1;
0002e733                      int64_t* rdi_1 = rsi;
0002e736                      rbx = 0;
0002e738                      while (i != 0)
0002e738                      {
0002e738                          bool cond:5_1 = 0 != *(uint8_t*)rdi_1;
0002e738                          rdi_1 = ((char*)rdi_1 + 1);
0002e738                          i = (i - 1);
0002e738                          if ((!cond:5_1))
0002e738                          {
0002e738                              break;
0002e738                          }
0002e738                      }
0002e73d                      rbp = ((int32_t)(!i));
0002e73a                  }
0002e70c                  goto label_2ea42;
0002e70c              }
0002e88f              if ((rdi == 0 && r8 == 4))
0002e88b              {
0002e8a5                  rbx = sub_18620(r12, &arg_20);
0002e8b0                  *(uint32_t*)rsi = arg_20;
0002e8b2                  if (rbx == 0)
0002e8ae                  {
0002e8b8                      rbp = ((int32_t)(rdi + 4));
0002e8b8                  }
0002e8ae              }
0002e883              goto label_2ea42;
0002e883          }
0002e8c2          if (rdi != 0)
0002e8c0          {
0002e8c2              goto label_2ea42;
0002e8c2          }
0002e8d0          if (r8 != 0x2a980)
0002e8cd          {
0002e8d0              goto label_2ea42;
0002e8d0          }
0002e8d6          void* rcx_12 = *(uint64_t*)((char*)arg1 + 8);
0002e8de          int64_t rax_24;
0002e8de          if ((*(uint8_t*)((char*)rcx_12 + 0xa) & 5) == 0)
0002e8da          {
0002e8e6              var_80 = 0;
0002e8f7              rax_24 = MmMapLockedPagesSpecifyCache(rcx_12, 0, 1, 0, 0, 0);
0002e8e6          }
0002e8e0          else
0002e8e0          {
0002e8e0              rax_24 = *(uint64_t*)((char*)rcx_12 + 0x18);
0002e8e0          }
0002e900          if (rax_24 != 0)
0002e8fd          {
0002ea0c              void* rcx_20 = *(uint64_t*)((char*)arg1 + 8);
0002ea14              int64_t rax_32;
0002ea14              if ((*(uint8_t*)((char*)rcx_20 + 0xa) & 5) == 0)
0002ea10              {
0002ea1c                  var_80 = 0;
0002ea2d                  rax_32 = MmMapLockedPagesSpecifyCache(rcx_20, 0, 1, 0, 0, 0);
0002ea1c              }
0002ea16              else
0002ea16              {
0002ea16                  rax_32 = *(uint64_t*)((char*)rcx_20 + 0x18);
0002ea16              }
0002ea36              rax_8 = sub_180d8(rax_32);
0002ea3d              if (rax_8 == 0)
0002ea3b              {
0002ea3d                  rbp = 0x2a980;
0002ea3d              }
0002ea3b              goto label_2ea40;
0002ea3b          }
0002e90a          int32_t rcx_13 = 0;
0002e90c          uint64_t i_4 = ((uint64_t)*(uint32_t*)(*(uint64_t*)((char*)arg1 + 8) + 0x28));
0002e912          uint64_t i_1 = (!i_4);
0002e918          if (i_1 != 0)
0002e915          {
0002e921              do
0002e921              {
0002e91a                  rcx_13 = (rcx_13 + 1);
0002e91d                  i_1 = (i_1 >> 4);
0002e91d              } while (i_1 != 0);
0002e91d          }
0002e925          int32_t r8_4 = 1;
0002e935          if (rcx_13 != 0)
0002e923          {
0002e935              r8_4 = rcx_13;
0002e935          }
0002e940          void* const rbx_1;
0002e940          if (i_4 >= 0x1000000000000000)
0002e93d          {
0002e942              rbx_1 = &data_1d750;
0002e942          }
0002e94b          else
0002e94b          {
0002e94b              int32_t rax_27 = 0;
0002e950              uint64_t i_2 = i_4;
0002e953              if (i_4 != 0)
0002e94d              {
0002e95c                  do
0002e95c                  {
0002e955                      rax_27 = (rax_27 + 1);
0002e958                      i_2 = (i_2 >> 4);
0002e958                  } while (i_2 != 0);
0002e958              }
0002e960              uint64_t rcx_14 = 1;
0002e965              rbx_1 = &data_1d750;
0002e96c              if (rax_27 != 0)
0002e95e              {
0002e96c                  rcx_14 = ((uint64_t)rax_27);
0002e96c              }
0002e987              if ((i_4 >> ((int8_t)((rcx_14 = 0x10)
0002e9a7          {
0002e9ab              i_6 = i_4;
0002e9ab          }
0002e9af          sub_15ecc("c:\temp\tr_src\tr\drvwdm\totrec7…");
0002e9b4          uint64_t i_7 = i_6;
0002e9d1          var_80 = rbx_1;
0002e9de          sub_15e18("!!! Err %X%s in %s %u, %s%IX", 0xc000009a);
0002e9e3          int64_t* rdx_9 = arg_8;
0002e9ee          if (rdx_9 != 0)
0002e9eb          {
0002e9f3              *(uint8_t*)(*(uint64_t*)rdx_9 + 0x18) = 0x2d;
0002e9f7              rdx_9[-1] = (rdx_9[-1] - 1);
0002e9fc              void* rcx_19 = rdx_9[-2];
0002ea00              *(uint32_t*)((char*)rcx_19 + 0xc) = (*(uint32_t*)((char*)rcx_19 + 0xc) - 1);
0002ea00          }
0002ea05          rax_31 = 0xc000009a;
0002ea05      }
0002e491      else
0002e491      {
0002e491          if (rax_1 == 0x22242c)
0002e485          {
0002e614              if (rdi > 0x3fe)
0002e60e              {
0002e614                  goto label_2ea42;
0002e614              }
0002e61d              if (r8 != 0)
0002e61a              {
0002e61d                  goto label_2ea42;
0002e61d              }
0002e61d              goto label_2e6bf;
0002e61d          }
0002e49c          if (rax_1 != 0x222407)
0002e497          {
0002e4a5              if (rax_1 == 0x222417)
0002e4a2              {
0002e596                  if (rdi != 0)
0002e594                  {
0002e596                      goto label_2ea42;
0002e596                  }
0002e59f                  if (r8 != 0)
0002e59c                  {
0002e59f                      goto label_2ea42;
0002e59f                  }
0002e5a5                  data_264ed = 1;
0002e5ac                  goto label_2e6bf;
0002e5ac              }
0002e4ae              if (rax_1 == 0x22241b)
0002e4ab              {
0002e579                  if (rdi != 0)
0002e577                  {
```
