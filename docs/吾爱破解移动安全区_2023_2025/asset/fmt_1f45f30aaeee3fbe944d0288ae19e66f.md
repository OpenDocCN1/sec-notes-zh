
```python
#  Copyright (c) 2023. 秋城落叶, Inc. All Rights Reserved
#  @作者         : 秋城落叶(QiuChenly)
#  @邮件         : 
#  @文件         : 项目 [qqmusic] - Netease.py
#  @修改时间    : 2023-03-02 07:38:37
#  @上次修改    : 2023/3/2 下午7:38
import base64
import json
import os
import cv2
from src.Api.BaseApi import BaseApi
from src.Common import Http
from src.Types.Types import Songs

class Netease(BaseApi):
    __baseUrl = 'http://cloud-music.pl-fe.cn'

    def __init__(self):
        self.__userInfo = None
        self.__httpServer = Http.HttpRequest()

    def http(self, url, method=0, data={}):
        return self.__httpServer.getHttp2Json(self.__baseUrl + url, method, data)

    def search(self, searchKey: str) -> list[Songs]:
        pass

    def cookie(self):
        dt = self.__httpServer.getSession().cookies.get_dict()
        return dt

    def set_cookie(self, ck: dict):
        self.__httpServer.setCookie(ck)

    def checkQrState(self, key: str):
        u = '/login/qr/check?key=' + key
        return self.http(u)

    def getUserDetail(self):
        u = '/user/account'
        if self.__userInfo is not None and self.__userInfo['code'] == 200:
            return self.__userInfo
        self.__userInfo = self.http(u).json()
        return self.__userInfo

    __likeList = []

    def getUserLikeList(self):
        u = f'/likelist?uid={self.__userInfo["account"]["id"]}'
        res = self.http(u).json()
        self.__likeList = res['ids']
        return self.__likeList

    userPlaylist = []

    def getUserPlaylist(self):
        global userPlaylist
        u = f'/user/playlist?uid={self.__userInfo["account"]["id"]}'
        res = self.http(u).json()
        userPlaylist = [
            {
                'userId': l['userId'],
                'trackCount': l['trackCount'],
                'name': l['name'],
                'id': l['id'],
                'coverImgUrl': l['coverImgUrl']
            } for l in res['playlist']
        ]
        # print("用户所有歌单")
        return userPlaylist

    def getPlayListAllMusic(self, playId, size=1000, offset=0):
        u = f'/playlist/track/all?id={playId}&limit={size}&offset={offset}'
        res = self.http(u)
        if res.status_code != 200:
            return None
        if res.text.find(":400}") != -1:
            return None
        js = res.json()
        return [
            {
                "name": li['name'],
                "id": li['id'],
                'author_simple': li['ar'][0]['name'],  # li['ar'][0]['name'] if len(li['ar']) == 1 else
                "author": li['ar'],  # 数组[{'id': 472822, 'name': 'JJD', 'tns': [], 'alias': []}]
                'publishTime': li['publishTime'],
                'album': li['al']
            } for li in js['songs']
        ]

    def qrLogin(self):
        u = '/login/qr/key'
        res = self.http(u)
        unikey = res.json()['data']['unikey']

        u = '/login/qr/create?key=' + unikey + "&qrimg=1"
        res = self.http(u).json()['data']
        b64 = res['qrimg']
        url = res['qrurl']

        img = base64.b64decode(b64.split(",")[1])
        with open("./login.png", "wb+") as p:
            p.write(img)
            p.flush()

        img = cv2.imread("./login.png")
        cv2.imshow("", img)
        cv2.waitKey(0)

        res = self.checkQrState(unikey)
        res = res.json()['code']
        if res == 803:
            # Login Success
            return True
        print("登录失败。")
        return False
```

### Other

Java 图示可以通过 AES 密钥解密出服务器的返回值。

```java
import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.io.IOException;
import java.util.Arrays;
import java.util.zip.Inflater;
import static javax.crypto.Cipher.DECRYPT_MODE;

public class Main {
    public static void main(String[] args) {
        System.out.println("Hello world!");
        String d = new String(unzip(readFileByBytes("/Users/qiuchenly/Downloads/response")));
        System.out.println(d);
        d = new String(unzip(readFileByBytes("/Users/qiuchenly/Downloads/request")));
        d = new String(hex2byte(d));
        byte[] bytes = hex2byte(d);
        d = new String(AesDecrypt(bytes, "6480fedae539deb2".getBytes()));
        System.out.println(d);
    }

    public static byte[] readFileByBytes(String fileName) {
        try {
            //传入文件路径fileName，底层实现 new FileInputStream(new File(fileName));相同
            FileInputStream in = new FileInputStream(fileName);
            //每次读10个字节，放到数组里
            byte[] bytes = new byte[1024];
            ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
            int c;
            while ((c = in.read(bytes)) != -1) {
                byteArrayOutputStream.write(bytes, 0, c);
            }
            return byteArrayOutputStream.toByteArray();
        } catch (Exception e) {
            // TODO: handle exception
        }
        return new byte[0];
    }

    private static byte[] hex2byte(String str) {
        if (str == null || str.length()
```

### Credits & Refs

- 我  谢  我  自  己
- 为了实现我的赛博丁真“网易云曲库流浪计划”，我做了一个小东西：[QQFlacMusicDownloader](https://github.com/QiuChenly/QQFlacMusicDownloader)

### 项目更新记录

#### 20230414 更新

- 增加歌曲关键词过滤，现在可以自定义歌曲名称中包含的特殊关键词进行过滤，如：不需要带DJ只需要自己增加一个DJ关键词即可自动过滤掉歌曲。
- UI改善，体验优化。

#### 20230313 更新

- 修复QQ音乐无损flac解析接口，现在已经可以继续正常使用。
- 酷我接口已经被官方堵了解析漏洞

疑似咱这个项目被官方注意到了大量的解析请求，所以封了漏洞----当然我丝毫没有给自己脸上贴金的意思----但确实就是如此

等后期有机会再修复吧

#### 20230309 更新

- 修复酷我音乐解析Hires直链返回183KB酷我版权mp3文件的问题
2. 其他小问题改正，参见Github的提交记录。

#### 20230305 更新

用vue配合flask撸了一个简易前端管理界面 已开源

![](https://attach.52pojie.cn/forum/202303/08/014920obn5fvn5niidvino.png)

`img_2.png` (376.3 KB，下载次数: 1)

[附件原图](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU5NDM3OXw0NzcwNjQ5MnwxNzkzNTY5NjY0fDE2ODA4NnwxNzUzNTg1&nothumb=yes)

2023-3-8 01:50 上传

![](https://attach.52pojie.cn/forum/202303/05/200424mop6y5e4oi75vtii.jpg)

`16780178047552.jpg` (494.39 KB，下载次数: 1)

[附件原图](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU5NDYyOXxlYzE2YjVlN3wxNzkzNTY5NjY0fDE2ODA4NnwxNzUzNTg1&nothumb=yes)

2023-3-5 20:04 上传
