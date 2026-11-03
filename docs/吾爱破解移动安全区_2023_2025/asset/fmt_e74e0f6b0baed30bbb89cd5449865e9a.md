
`xxd -r -p '加密文件' > '解密文件'`

然后就是`mv`移动到`/data/local/tmp`，`chmod`给`777`权限，`sh`执行

## 0x2 base64

解密后的文件里还有一个 `echo '加密内容'| base64 -d|sh` 这种的shell加密，直接把加密内容复制出来，找个base64网站解密就行，这个没啥好说的

https://www.toolhelper.cn/EncodeDecode/Base64

## 0x3 小结

这次偷懒成功，不多说了，黑神话启动！
