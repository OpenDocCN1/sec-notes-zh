
# 增强版 DJB2 哈希碰撞生成器

```python
# -*- coding: utf-8 -*-
"""
增强版 DJB2 哈希碰撞生成器

功能：
&#9989; 给定目标哈希值，生成能产生该哈希的输入字符串
&#9989; 支持指定生成数量（如只生成 5 个）
&#9989; 可选仅使用“可见字符”（可打印 ASCII）
&#9989; 自动扩展长度以寻找更多碰撞
"""

from __future__ import print_function

import warnings
import random
import copy
import time
import sys

# 导入哈希函数
from hash_functions import djb2_32, djb2_64, djb2_xor_32

class GreedyGenerator:
    """
    贪心式哈希碰撞生成器（支持可见字符限制）
    """

    DEFAULT_LENGTH = 16              # 减小默认长度以便更快找到结果
    TOTAL_ROUNDS = 10                # 每轮优化次数
    MUTATION_MOD = 2                 # 突变比例：修改一半字节
    VERBOSE = False                  # 是否输出调试信息

    # 定义可见字符范围（ASCII 32 ~ 126）
    PRINTABLE_RANGE = list(range(32, 127))

    def __init__(self, hash_function, target_value, total_letters=DEFAULT_LENGTH, only_printable=False):
        """
        初始化生成器
        :param hash_function: 哈希函数（如 djb2_32）
        :param target_value: 目标哈希值
        :param total_letters: 字符串长度
        :param only_printable: 是否仅使用可见字符
        """
        if total_letters  0")
        old_val = self.string_to_guess[:min(self.TOTAL_LETTERS, new_size)]
        self.TOTAL_LETTERS = new_size
        self.BYTES_TO_MUTATE = max(1, self.TOTAL_LETTERS // self.MUTATION_MOD)
        new_guess = bytearray(b'\x00') * self.TOTAL_LETTERS
        for i in range(len(old_val)):
            new_guess[i] = old_val[i]
        self.string_to_guess = new_guess

    def __iter__(self):
        return self

    def __next__(self):
        return self.next()

    def next(self):
        """生成下一个可能的碰撞（返回 bytes）"""
        self.string_to_guess = self.mutate_full(self.string_to_guess)

        while True:
            prev_state = None

            for round_idx in range(self.TOTAL_ROUNDS):
                improved = False

                for pos in range(self.TOTAL_LETTERS):
                    candidates = self._get_hash_for_position(pos)
                    match = self._check_exact_hit(pos, candidates)
                    if match is not None:
                        return match

                    # 在允许的字节范围内选择最优值
                    best_byte = min(
                        self.allowed_bytes,
                        key=lambda b: abs(candidates.get(b, float('inf')) - self.target)
                    )

                    if self.string_to_guess[pos] != best_byte:
                        self.string_to_guess[pos] = best_byte
                        improved = True

                current_hash = self.checker(self.string_to_guess)
                if self.VERBOSE:
                    print(f"第 {round_idx} 轮 | 哈希={current_hash} | 误差={abs(current_hash - self.target)}")

                if not improved or self.string_to_guess == prev_state:
                    break
                prev_state = copy.copy(self.string_to_guess)

            self.string_to_guess = self.mutate_partial(self.string_to_guess)

    def _get_hash_for_position(self, index):
        """枚举当前位置所有允许字节对应的哈希值"""
        original = self.string_to_guess[index]
        results = {}

        for b in self.allowed_bytes:
            self.string_to_guess[index] = b
            results[b] = self.checker(self.string_to_guess)

        self.string_to_guess[index] = original
        return results

    def _check_exact_hit(self, index, candidates):
        """检查是否有字节能命中目标哈希"""
        for byte_val, h in candidates.items():
            if h == self.target and byte_val in self.allowed_bytes:
                self.string_to_guess[index] = byte_val
                return bytes(copy.copy(self.string_to_guess))
        return None

    def mutate_partial(self, data):
        """部分突变：只在允许字符集中随机替换"""
        for _ in range(self.BYTES_TO_MUTATE):
            pos = random.randint(0, self.TOTAL_LETTERS - 1)
            new_val = random.choice(self.allowed_bytes)
            data[pos] = new_val
        return data

    def mutate_full(self, data):
        """完全随机初始化（仅用允许字符）"""
        return bytearray(random.choices(self.allowed_bytes, k=self.TOTAL_LETTERS))

if __name__ == "__main__":
    """
    主入口：支持参数控制
    用法: python generator.py  [选项...]
    示例:
        python generator.py 1717864659
        python generator.py 123456789 -n 5 -p -l 20
    """

    if len(sys.argv)  [-n 数量] [-p] [-l 长度]", file=sys.stderr)
        print("选项:")
        print("  -n N      : 只生成 N 个结果（默认无限）")
        print("  -p        : 仅使用可见字符（可打印 ASCII）")
        print("  -l LENGTH : 设置初始字符串长度（默认 16）")
        sys.exit(1)

    # 解析目标哈希
    try:
        target_hash = int(sys.argv[1])
    except ValueError:
        print("&#10060; 错误：目标哈希必须是整数", file=sys.stderr)
        sys.exit(1)

    # 默认参数
    max_count = None           # 默认不限制数量
    only_printable = False     # 是否仅可见字符
    string_length = 16         # 初始长度

    # 解析命令行参数
    args = sys.argv[2:]
    i = 0
    while i = 2**64:
        print("&#10060; 哈希值必须在 [0, 2^64) 范围内", file=sys.stderr)
        sys.exit(1)

    # hash_func = djb2_64 if target_hash >= 2**32 else djb2_32
    hash_func = djb2_xor_32
    bits = "64位" if target_hash >= 2**32 else "32位"

    # 输出配置信息
    print(f"&#127919; 开始寻找哈希值 {target_hash}")
    print(f"&#128295; 使用 {bits} DJB2 算法")
    print(f"&#128207; 初始长度: {string_length}")
    print(f"&#128292; {'仅限可见字符' if only_printable else '允许所有字节'}")
    print(f"&#128202; {'最多生成 %d 个' % max_count if max_count else '无限生成'}")

    start_time = time.time()
    generator = GreedyGenerator(hash_func, target_hash, string_length, only_printable=only_printable)

    found_collisions = set()
    consecutive_misses = 0

    try:
        while True:
            try:
                candidate = next(generator)
                if candidate not in found_collisions:
                    try:
                        # 尝试以 UTF-8 解码显示（非必须）
                        # decoded = candidate.decode('utf-8', errors='backslashreplace')
                        print(f"{candidate.decode('utf-8', errors='backslashreplace')}")
                    except:
                        print(candidate)
                    found_collisions.add(candidate)
                    consecutive_misses = 0

                    # 如果达到指定数量，退出
                    if max_count and len(found_collisions) >= max_count:
                        break
                else:
                    consecutive_misses += 1

            except StopIteration:
                print(f"&#10145;&#65039;  当前长度({generator.TOTAL_LETTERS})无新解，扩展至 {generator.TOTAL_LETTERS + 1}...")
                generator.set_collision_size(generator.TOTAL_LETTERS + 1)
                consecutive_misses = 0

            # 提前结束判断
            if max_count and len(found_collisions) >= max_count:
                break

        # 主动结束输出统计
        end_time = time.time()
        elapsed = end_time - start_time
        print("\n" + "=" * 60)
        print("&#9989; 任务完成！")
        print(f"&#128278; 目标哈希: {target_hash}")
        print(f"&#128230; 生成数量: {len(found_collisions)}")
        print(f"&#9201;  耗时: {elapsed:.2f} 秒")
        if elapsed > 0:
            print(f"&#128640; 平均速度: {len(found_collisions) / elapsed:.4f} 个/秒")
        print("=" * 60)

    except KeyboardInterrupt:
        end_time = time.time()
        elapsed = end_time - start_time
        print("\n\n&#9208;&#65039;  用户中断")
        print(f"&#128204; 共生成 {len(found_collisions)} 个碰撞")
        print(f"&#9200; 耗时: {elapsed:.2f} 秒")
        if elapsed > 0:
            print(f"&#9889; 速度: {len(found_collisions) / elapsed:.4f} 个/秒")
```

## `hash_functions.py`

```python
# -*- coding: utf-8 -*-

from functools import partial

MOD_32 = 2 ** 32

MOD_64 = 2 ** 64

def djb2(data, modulo=MOD_32):
    h = 5381

    for c in data:
        h = (h * 33 + c) % modulo

    return h

def djb2_xor_32(data, modulo=MOD_32):
    """
    &#9888;&#65039; 非标准“异或型”DJ B2（常出现在 CTF 或虚拟机中）

    公式: h = h * 33 ^ c

    注意：这不是原始 DJB2！但常被误称为 DJB2

    用途：混淆、反分析、小型 VM 验证
    """
    h = 5381

    for c in data:
        h = ((h * 33) ^ c) % modulo

    return h
```
