
```python
data = json.dumps({"page": str(random.randint(1,100)),"sortType":3,"isSel":"false","rows":9,"isFavorites":False,"wpType":1},separators=(',', ':'))
encrypted_data = aes_encrypt(data, key, iv)
#print("Encrypted Data (Base64):", encrypted_data)
params = {
    "data":encrypted_data
}
res = requests.get(url,headers=headers,verify=False,params=params)
#print(res.text)
data_code = res.status_code
if data_code != 200 :
    return ""
decrypted_data = aes_decrypt(res.json()["data"], key, iv)
print("Decrypted Data:", decrypted_data)
decrypted_data = decrypted_data.replace('\x00', '').replace('true', 'True').strip()
print(type(eval(decrypted_data)))
wtId = eval(decrypted_data)["list"][random.randint(0,8)]["wtId"]
print(wtId)
res = requests.get("https://x.haowallpaper.com/link/common/file/getFileImg/" + wtId + ".png",headers=headers)
with open('1.png','wb') as f:
    f.write(res.content)
if __name__ == "__main__":
    getWallpaperList()
```
