# # Table 9.4 Progress in Factorization

| Number of Decimal Digits | Approximate Number of Bits | Date Achieved | MIPS-years | Algorithm |
|---|---|---|---|---|
| 100 | 332 | April 1991 | 7 | quadratic sieve |
| 110 | 365 | April 1992 | 75 | quadratic sieve |
| 120 | 398 | June 1993 | 830 | quadratic sieve |
| 129 | 428 | April 1994 | 5000 | quadratic sieve |
| 130 | 431 | April 1996 | 1000 | generalized number field sieve |
| 140 | 465 | February 1999 | 2000 | generalized number field sieve |
| 155 | 512 | August 1999 | 8000 | generalized number field sieve |
| 160 | 530 | April 2003 | — | Lattice sieve |
| 174 | 576 | December 2003 | — | Lattice sieve |
| 200 | 663 | May 2005 | — | Lattice sieve |

+   - p和q应满足下列约束条件：
+     - P和q的长度应仅相差几位，p和q都应约在\(10^{75}\)到\(10^{100}\)之间
+     - (p-1)和(q-1)都应有一个大的素因子
+     - GCD(p-1, q-1)应该较小
+   
+   另外，已经证明，若e<n且d<n<sup>1/4</sup>，则d很容易确定