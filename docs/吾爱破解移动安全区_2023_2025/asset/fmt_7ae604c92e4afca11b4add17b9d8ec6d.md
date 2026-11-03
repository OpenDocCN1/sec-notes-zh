
# [Android 原创] 某安卓端xxtea加密的游戏资源的解密

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1823765
- 抓取时间: 2026-11-01T21:40:37

游戏名称：5a+75YWJ5LmL5peF

这是前段时间，帮一个群友解密的某游戏，代码贴一下，欢迎互相交流

```
import xxtea#pip install xxtea-py

def decrypt_xxtea_file(orig_path, new_path, xxtea_key, xxtea_sign): 
    # 读取原文件
    orig_file = open(orig_path, "rb")
    encrypt_bytes = orig_file.read() 
    if(encrypt_bytes[10]==0):
        
        encrypt_bytes = encrypt_bytes[13:]
        #print(encrypt_bytes)
    else:
        encrypt_bytes = encrypt_bytes[9:]
    orig_file.close()
    # 解密文件 
    allbytes=b''
    j=0 
    decrypt_bytes=b''
    for j in range(0,len(encrypt_bytes)):
        decrypt_bytes = xxtea.decrypt(encrypt_bytes[:j], xxtea_key) 
        if(decrypt_bytes!=b''):
            print(j,decrypt_bytes)
            break
    allbytes=decrypt_bytes+encrypt_bytes[j:]
    #print(allbytes)
    new_file = open(new_path, "wb")
    new_file.write(allbytes)
    new_file.close()
    print("Done with " + orig_path)
    
def getKeyOfFile(filename): 
    data = open(filename,'rb').read()
    dex = bytes.fromhex('436f707972696768742843292c323031372c447261676f6e50756e636853746f726d20546563682e436f2e2c4c74642e')
    def getv12(d): 
        return dex[d]
    v9=0x7
    v10=0x30
    #v12=0x28
    v13=data[v9] 
    #0x6f
    v15=data[v9+1] 

    v16 =( v15 + v13)
     #0x141
    v17 = (v15 + v15 + v13)  

    v18 = (v15 + v13 + v17)
    v19 = (v18 + v17 + v18)
    v20 = (v17 + v18 + v19)
    #print('v20',v20)

    v21=0
    v21=12
    v37=[] 
    print('v21', v16 - 13 * (((1321528399 * v16) >> 32) >> 2))
    if (( v16 - 13 * (((1321528399 * v16) >> 32) >> 2)) > 32) >> 2); 
        v37=[0 for i in range(v21)]
        v22 = v19 + v20;
        v37[8] =  getv12( v22 % v10);
        if ( v21 != 9 ):
          v23 = v20 + v22;
          v37[9] = getv12( v23 % v10);
          if (v21 != 10):
            v24 = v22 + v23;
            v37[10] = getv12(v24 % v10);
            if (v21 == 12):
              v37[11] = getv12( (v23 + v24) % v10);
        print("警告：key长度超过8",v21)
        
    
    v37[0] = getv12(v13 % v10)
    v37[1] = getv12( v15 % v10)
    v37[2] = getv12( v16 % v10)
    v37[3] = getv12 (v17 % v10)
    v37[4] = getv12(v18 % v10)
    v37[5] = getv12 ((v17 + v18) % v10)
    v37[6] = getv12 (v19 % v10)
    v37[7] = getv12( v20 % v10)
    #79 31 44 6d 79 65 2e 63
    #79 31 44 6d 79 65 2e 63 a8 3e 34 8a              y1Dmye.c.>4.
    l=8
    if(v21==8):
        l=8
    else:
        l=v21+1
    return bytes(v37),data[:l]

name='01669ba70974393a58ebb573a7562924'
ext='png'
key,sign = getKeyOfFile(name+'.'+ext)
print('key is ',key,sign)
decrypt_xxtea_file(name+'.'+ext,name+'_out.'+ext,key,sign)
```
