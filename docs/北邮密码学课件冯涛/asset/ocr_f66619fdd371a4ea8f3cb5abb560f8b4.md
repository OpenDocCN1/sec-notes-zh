# RC4 密钥方案

+   - 用 0，…，255 初始化一个数组 S
+   - 使用密钥去实现混淆
+   - S 构成了密码的内部状态

```
for i = 0 to 255 do
    S[i] = i
    T[i] = K[i mod keylen]

j = 0
for i = 0 to 255 do
    j = (j + S[i] + T[i]) (mod 256)
    swap (S[i], S[j])
```