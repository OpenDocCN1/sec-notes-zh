
# EFLAGS 使用与定义分析

## 枚举定义

```cpp
enum class x86_flag : uint8_t { AF, CF, SF, ZF, PF, OF, TF, IF, DF, NT, RF };
```

然后`intel`规定一条指令对`eflags`的影响有7种形式，分别是：（来自`Volume 1 -> Appendix A EFLAGES Cross-Reference`）

## Intel EFLAGS 影响类型

- `T` Instruction tests flag.
- `M` Instruction modifies flag (either sets or resets depending on operands).
- `0` Instruction resets flag.
- `1` Instruction sets flag.
- `—` Instruction's effect on flag is undefined.
- `R` Instruction restores prior value of flag.
- Blank Instruction does not affect flag.

![](https://attach.52pojie.cn/forum/202501/20/090006itzzlgegzoww9j7n.png)
微信截图_20250120085923.png (70.42 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc1MDcxN3w2ZjE3YThkNXwxNzkzNTcxNDAxfDE2ODA4NnwyMDAxMTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MDcxN3w2ZjE3YThkNXwxNzkzNTcxNDAxfDE2ODA4NnwyMDAxMTky&nothumb=yes)
2025-1-20 09:00 上传

## calc_eflags_use 实现

按照其内容，我将 `T` 归为了 `use`

```cpp
std::vector calc_eflags_use(csh hCs, const Insn& ins) {
cs_insn cinsn{};
cs_detail cdetail{};
disasm(hCs, ins.addr, ins.viraddr, ins.size, &cinsn, &cdetail);
uint64_t eflags = cdetail.x86.eflags;

std::vector ret = {};

// TEST:    Instruction tests flag
std::vector useChecks = { X86_EFLAGS_TEST_OF, X86_EFLAGS_TEST_SF, X86_EFLAGS_TEST_ZF,
    X86_EFLAGS_TEST_PF, X86_EFLAGS_TEST_CF, X86_EFLAGS_TEST_NT, X86_EFLAGS_TEST_DF,
    X86_EFLAGS_TEST_RF, X86_EFLAGS_TEST_IF, X86_EFLAGS_TEST_TF, X86_EFLAGS_TEST_AF 
};

for (uint64_t check : useChecks) {
    if (eflags & check) {
        x86_flag ret_flag;
        switch (check){
            case X86_EFLAGS_TEST_OF: ret_flag = x86_flag::OF; break;
            case X86_EFLAGS_TEST_SF: ret_flag = x86_flag::SF; break;
            case X86_EFLAGS_TEST_ZF: ret_flag = x86_flag::ZF; break;
            case X86_EFLAGS_TEST_PF: ret_flag = x86_flag::PF; break;
            case X86_EFLAGS_TEST_CF: ret_flag = x86_flag::CF; break;
            case X86_EFLAGS_TEST_NT: ret_flag = x86_flag::NT; break;
            case X86_EFLAGS_TEST_DF: ret_flag = x86_flag::DF; break;
            case X86_EFLAGS_TEST_RF: ret_flag = x86_flag::RF; break;
            case X86_EFLAGS_TEST_IF: ret_flag = x86_flag::IF; break;
            case X86_EFLAGS_TEST_TF: ret_flag = x86_flag::TF; break;
            case X86_EFLAGS_TEST_AF: ret_flag = x86_flag::AF; break;
        }
        ret.push_back(ret_flag);
    }
}
return ret;
}
```

## calc_eflags_def 实现

而对于`M`、`R`、`1`、`0`，我则将它们归为了 `def`

```cpp
std::vector calc_eflags_def(csh hCs, const Insn& ins) {
cs_insn cinsn{};
cs_detail cdetail{};
disasm(hCs, ins.addr, ins.viraddr, ins.size, &cinsn, &cdetail);
uint64_t eflags = cdetail.x86.eflags;
std::vector ret = {};
// MODIFY:   Instruction modifies flag (either sets or resets depending on operands).
// PRIOR:    Instruction restores prior value of flag
// SET:      Instruction sets flag
// RESET:    Instruction resets flag
std::vector defChecks = { X86_EFLAGS_PRIOR_OF, X86_EFLAGS_PRIOR_SF, X86_EFLAGS_PRIOR_ZF,
    X86_EFLAGS_PRIOR_AF, X86_EFLAGS_PRIOR_PF, X86_EFLAGS_PRIOR_CF, X86_EFLAGS_PRIOR_TF,
    X86_EFLAGS_PRIOR_IF, X86_EFLAGS_PRIOR_DF, X86_EFLAGS_PRIOR_NT, X86_EFLAGS_MODIFY_AF,
    X86_EFLAGS_MODIFY_CF, X86_EFLAGS_MODIFY_SF, X86_EFLAGS_MODIFY_ZF, X86_EFLAGS_MODIFY_PF,
    X86_EFLAGS_MODIFY_OF, X86_EFLAGS_MODIFY_TF, X86_EFLAGS_MODIFY_IF, X86_EFLAGS_MODIFY_DF,
    X86_EFLAGS_MODIFY_NT, X86_EFLAGS_MODIFY_RF, X86_EFLAGS_SET_CF, X86_EFLAGS_SET_DF,
    X86_EFLAGS_SET_IF, X86_EFLAGS_SET_OF, X86_EFLAGS_SET_SF, X86_EFLAGS_SET_ZF, X86_EFLAGS_SET_AF,
    X86_EFLAGS_SET_PF, X86_EFLAGS_RESET_OF, X86_EFLAGS_RESET_CF, X86_EFLAGS_RESET_DF,
    X86_EFLAGS_RESET_IF, X86_EFLAGS_RESET_SF, X86_EFLAGS_RESET_AF, X86_EFLAGS_RESET_TF,
    X86_EFLAGS_RESET_NT, X86_EFLAGS_RESET_PF, X86_EFLAGS_RESET_RF, X86_EFLAGS_RESET_ZF };
for (uint64_t check : defChecks) {
    if (eflags & check) {
        x86_flag ret_flag;
        switch (check) {
            case X86_EFLAGS_MODIFY_AF: ret_flag = x86_flag::AF; break;
            case X86_EFLAGS_MODIFY_CF: ret_flag = x86_flag::CF; break;
            case X86_EFLAGS_MODIFY_SF: ret_flag = x86_flag::SF; break;
            case X86_EFLAGS_MODIFY_ZF: ret_flag = x86_flag::ZF; break;
            case X86_EFLAGS_MODIFY_PF: ret_flag = x86_flag::PF; break;
            case X86_EFLAGS_MODIFY_OF: ret_flag = x86_flag::OF; break;
            case X86_EFLAGS_MODIFY_TF: ret_flag = x86_flag::TF; break;
            case X86_EFLAGS_MODIFY_IF: ret_flag = x86_flag::IF; break;
            case X86_EFLAGS_MODIFY_DF: ret_flag = x86_flag::DF; break;
            case X86_EFLAGS_MODIFY_NT: ret_flag = x86_flag::NT; break;
            case X86_EFLAGS_MODIFY_RF: ret_flag = x86_flag::RF; break;
            case X86_EFLAGS_PRIOR_OF: ret_flag = x86_flag::OF; break;
            case X86_EFLAGS_PRIOR_SF: ret_flag = x86_flag::SF; break;
            case X86_EFLAGS_PRIOR_ZF: ret_flag = x86_flag::ZF; break;
            case X86_EFLAGS_PRIOR_AF: ret_flag = x86_flag::AF; break;
            case X86_EFLAGS_PRIOR_PF: ret_flag = x86_flag::PF; break;
            case X86_EFLAGS_PRIOR_CF: ret_flag = x86_flag::CF; break;
            case X86_EFLAGS_PRIOR_TF: ret_flag = x86_flag::TF; break;
            case X86_EFLAGS_PRIOR_IF: ret_flag = x86_flag::IF; break;
            case X86_EFLAGS_PRIOR_DF: ret_flag = x86_flag::DF; break;
            case X86_EFLAGS_PRIOR_NT: ret_flag = x86_flag::NT; break;
            case X86_EFLAGS_SET_CF: ret_flag = x86_flag::CF; break;
            case X86_EFLAGS_SET_DF: ret_flag = x86_flag::DF; break;
            case X86_EFLAGS_SET_IF: ret_flag = x86_flag::IF; break;
            case X86_EFLAGS_SET_OF: ret_flag = x86_flag::OF; break;
            case X86_EFLAGS_SET_SF: ret_flag = x86_flag::SF; break;
            case X86_EFLAGS_SET_ZF: ret_flag = x86_flag::ZF; break;
            case X86_EFLAGS_SET_AF: ret_flag = x86_flag::AF; break;
            case X86_EFLAGS_SET_PF: ret_flag = x86_flag::PF; break;
            case X86_EFLAGS_RESET_OF: ret_flag = x86_flag::OF; break;
            case X86_EFLAGS_RESET_CF: ret_flag = x86_flag::CF; break;
            case X86_EFLAGS_RESET_DF: ret_flag = x86_flag::DF; break;
            case X86_EFLAGS_RESET_IF: ret_flag = x86_flag::IF; break;
            case X86_EFLAGS_RESET_SF: ret_flag = x86_flag::SF; break;
            case X86_EFLAGS_RESET_AF: ret_flag = x86_flag::AF; break;
            case X86_EFLAGS_RESET_TF: ret_flag = x86_flag::TF; break;
            case X86_EFLAGS_RESET_NT: ret_flag = x86_flag::NT; break;
            case X86_EFLAGS_RESET_PF: ret_flag = x86_flag::PF; break;
            case X86_EFLAGS_RESET_RF: ret_flag = x86_flag::RF; break;
            case X86_EFLAGS_RESET_ZF: ret_flag = x86_flag::ZF; break;
        }
        ret.push_back(ret_flag);
    }
}
return ret;
}
```

## 活跃性分析与垃圾指令判定

紧接类似地，用这些去创建`eflags`的`use`、`def`、`in`、`out`集合，和寄存器一样的流程去解析它们的`use`、`def`，去计算它们的`in`、`out`；最后再和寄存器的`def`/`out`的交集结果用`&&`连起来判断就可以了

```cpp
if (my_set_intersection(def[i], out[i]).empty() &&
    my_set_intersection(def_flags[i], out_flags[i]).empty() &&
    my_set_intersection(def_fpuflags[i], out_fpuflags[i]).empty()) { // 判断 mbb[i] 是否为垃圾指令
    changed = true; // 准备进行新一轮迭代
    delList[i] = true; // 标记垃圾指令
}
```

## 关于 fpu flags

这是只是我顺手写的，为了保证最大兼容性，写一下这个应该也无妨

## 参考

【利用活跃变量分析来去掉vmp的大部分垃圾指令】 [//bbs.kanxue.com/thread-265950.htm" target="_blank" rel="noopener noreferrer nofollow"](https://bbs.kanxue.com/thread-265950.htm:rel="nofollow noopener" href="https)
【C++利用活跃变量分析清除基本块中的垃圾指令】
[//bbs.kanxue.com/thread-277825.htm" target="_blank" rel="noopener noreferrer nofollow"](https://bbs.kanxue.com/thread-277825.htm:rel="nofollow noopener" href="https)
【中南大学 编译原理 - 课时42: 数据流分析(2) 】
[//www.bilibili.com/video/BV1jU4y1W7vG" target="_blank" rel="noopener noreferrer nofollow"](https://www.bilibili.com/video/BV1jU4y1W7vG:rel="nofollow noopener" href="https)
