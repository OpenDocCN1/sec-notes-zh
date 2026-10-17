
## 轮函数RFx——参数

+   参数 k,s,i
+   k = 每轮由不同置换确定，4轮置换定义如下：
+   ρ1(i)=i mod 16
+   ρ2(i)=(1+5i) mod 16
+   ρ3(i)=(5+3i) mod 16
+   ρ4(i)=7i mod 16
+   s = 每轮循环左移次数都不相同
+   i = 4轮运算中全部64步迭代的顺序序号，范围从1到64

## 轮函数RFx——T表

(b) Table T, constructed from the sine function

| T[1] = D76AA478 | T[17] = F61E2562 | T[33] = FFBA3942 | T[49] = F4292244 |
| T[2] = E8C7B756 | T[18] = C040B340 | T[34] = 8771F681 | T[50] = 432AFF97 |
| T[3] = 242070DB | T[19] = 265E5A51 | T[35] = 699D6122 | T[51] = AB9423A7 |
| T[4] = C1BDCEEE | T[20] = E9B6C7AA | T[36] = FDE5380C | T[52] = FC93A039 |
| T[5] = F57C0FAF | T[21] = D62F105D | T[37] = A4BEEA44 | T[53] = 655B59C3 |
| T[6] = 4787C62A | T[22] = 02441453 | T[38] = 4BDECFA9 | T[54] = 8F0CCC92 |
| T[7] = A8304613 | T[23] = D8A1E681 | T[39] = F6BB4B60 | T[55] = FFEFF47D |
| T[8] = FD469501 | T[24] = E7D3FBC8 | T[40] = BEBFBC70 | T[56] = 85845DD1 |
| T[9] = 698098D8 | T[25] = 21E1CDE6 | T[41] = 289B7EC6 | T[57] = 6FA87E4F |
| T[10] = 8B44F7AF | T[26] = C33707D6 | T[42] = EAA127FA | T[58] = FE2CE6E0 |
| T[11] = FFFF5BB1 | T[27] = F4D50D87 | T[43] = D4EF3085 | T[59] = A3014314 |
| T[12] = 895CD7BE | T[28] = 455A14ED | T[44] = 04881D05 | T[60] = 4E0811A1 |
| T[13] = 6B901122 | T[29] = A9E3E905 | T[45] = D9D4D039 | T[61] = F7537E82 |
| T[14] = FD987193 | T[30] = FCEFA3F8 | T[46] = E6DB99E5 | T[62] = BD3AF235 |
| T[15] = A679438E | T[31] = 676F02D9 | T[47] = 1FA27CF8 | T[63] = 2AD7D2BB |
| T[16] = 49B40821 | T[32] = 8D2A4C8A | T[48] = C4AC5665 | T[64] = EB86D391 |

## 轮函数RFx——基本逻辑函数g

| 轮 | 基本函数g | g(b,c,d) |
|---|---|---|
| 1 | F(b,c,d) | (b∧c) ∨ (¬b∧d) |
| 2 | G(b,c,d) | (b∧d) ∨ (c∧¬d) |
| 3 | H(b,c,d) | b⊕c⊕d |
| 4 | I(b,c,d) | c⊕(b∨¬d) |

(a) Truth table of logical functions

| b | c | d | F | G | H | I |
|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 | 0 | 1 |
| 0 | 0 | 1 | 1 | 0 | 1 | 0 |
| 0 | 1 | 0 | 0 | 1 | 1 | 0 |
| 0 | 1 | 1 | 1 | 0 | 0 | 1 |
| 1 | 0 | 0 | 0 | 0 | 1 | 1 |
| 1 | 0 | 1 | 0 | 1 | 0 | 1 |
| 1 | 1 | 0 | 1 | 1 | 0 | 0 |
| 1 | 1 | 1 | 1 | 1 | 1 | 0 |

## MD5——更新算法（RFC1321）

```
/* Process each 16-word (512-bit) block. */
For q = 0 to (N/16 - 1) do
  /* Copy block q into X. */
  For j = 0 to 15 do
    Set X[j] to M[q*16 + j];
  end /* of loop on j */

/* Save A as AA, B as BB, C as CC, and D as DD. */
AA = A
BB = B
CC = C
DD = D
```

```
/* Round 1. */
/* Let [abcd k s i] denote the operation
a = b + ((a + F(b,c,d) + X[k] + T[i]) <<< s).
Do the following 16 operations. */
[ABCD  0  7  1]
[DABC  1 12  2]
[CDAB  2 17  3]
[BCDA  3 22  4]
[ABCD  4  7  5]
[DABC  5 12  6]
[CDAB  6 17  7]
[BCDA  7 22  8]
[ABCD  8  7  9]
[DABC  9 12 10]
[CDAB 10 17 11]
[BCDA 11 22 12]
[ABCD 12  7 13]
[DABC 13 12 14]
[CDAB 14 17 15]
[BCDA 15 22 16]
```

```
/* Round 2. */
/* Let [abcd k s i] denote the operation
a = b + ((a + G(b,c,d) + X[k] + T[i]) <<< s).
Do the following 16 operations. */
[ABCD  1  5 17]
[DABC  6  9 18]
[CDAB 11 14 19]
[BCDA  0 20 20]
[ABCD  5  5 21]
[DABC 10  9 22]
[CDAB 15 14 23]
[BCDA  4 20 24]
[ABCD  9  5 25]
[DABC 14  9 26]
[CDAB  3 14 27]
[BCDA  8 20 28]
[ABCD 13  5 29]
[DABC  2  9 30]
[CDAB  7 14 31]
[BCDA 12 20 32]
```

```
/* Round 3. */
/* Let [abcd k s i] denote the operation
a = b + ((a + H(b,c,d) + X[k] + T[i]) <<< s).
Do the following 16 operations. */
[ABCD  5  4 33]
[DABC  8 11 34]
[CDAB 11 16 35]
[BCDA 14 23 36]
[ABCD  1  4 37]
[DABC  4 11 38]
[CDAB  7 16 39]
[BCDA 10 23 40]
[ABCD 13  4 41]
[DABC  0 11 42]
[CDAB  3 16 43]
[BCDA  6 23 44]
[ABCD  9  4 45]
[DABC 12 11 46]
[CDAB 15 16 47]
[BCDA  2 23 48]
```

```
/* Round 4. */
/* Let [abcd k s i] denote the operation
a = b + ((a + I(b,c,d) + X[k] + T[i]) <<< s).
Do the following 16 operations. */
[ABCD  0  6 49]
[DABC  7 10 50]
[CDAB 14 15 51]
[BCDA  5 21 52]
[ABCD 12  6 53]
[DABC  3 10 54]
[CDAB 10 15 55]
[BCDA  1 21 56]
[ABCD  8  6 57]
[DABC 15 10 58]
[CDAB  6 15 59]
[BCDA 13 21 60]
[ABCD  4  6 61]
[DABC 11 10 62]
[CDAB  2 15 63]
[BCDA  9 21 64]
```

```
/* Then increment each of the four registers by the value it had before this block was started. */
A = A + AA
B = B + BB
C = C + CC
D = D + DD

end /* of loop on q */
```
