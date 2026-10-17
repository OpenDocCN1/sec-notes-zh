+   - 换位密码把明文按列写入，按行读出
+   - 密钥包含3方面信息：行宽，列高，读出顺序

```
key:        4  3  1  2  5  6  7
plaintext:  a  t  t  a  c  k  p
            o  s  t  p  o  n  e
            d  u  n  t  i  l  t
            w  o  a  m  x  y  z
ciphertext: TTNAAPT MTSU AODWCOIXPETZ
```

+   - 完全保留字符的统计信息
+   - 使用多轮加密可提高安全性