
# [Web逆向] 西瓜视频最新解密链接

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=1962421](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1962421)
- 抓取时间：2026-11-01T22:24:07

## 解密要点

上一个吾友的帖子：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=1917707&highlight=%CE%F7%B9%CF](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1917707&highlight=%CE%F7%B9%CF)

差不多意思，`key` 还是 `ptk`。

变动的是 `iv`。

`iv` 构造方法为 `{ptk}_{video_id}`，例如 `ef84a15b-ffd0-4c4d-9d56-b652532f_v03043740000bkdcm2d8kgepf3ld6a0g`，再进行 `md5()` 取 16 位。

PHP 可以用：

`md5('ef84a15b-ffd0-4c4d-9d56-b652532f_v03043740000bkdcm2d8kgepf3ld6a0g',1)`

解密还是有效，最新软件链接已顶置。

## 完整示例

完整例子如下

[PHP] 纯文本查看 复制代码

```php
<?php
$data = "VrrQyJKydr29PXzidAN+p1w7w9YSn8XyXoRc5R3+cTNp6CDQJup4kAYicBD/ijLZRBedZTqeWho5AWkSygwFaHoLO2LQwsTsY9SpcHx5OX3M9NSwZGx5jj9GLcV+74mRqQrzzpAD0iZ+3Jxksp7kzkyzDl6ZoakXX00LpASiBHLCPMNpsgm96hT5gUOeR5z45/b/83G+THOClQYYcYTwcRM91wYRP52JayjGeD5DAclRl6A8DTezs9fyPOGx7dVSmXr67xfovBUWUvzFX7tzlAsV3DlXt09xmM7dCOuaEDd592SAOCsUIwGiQ1eTcebteUQohDFmmLY6BytGnV5m2suHxddPMZbkcQ0fJS0zi6/3OCZBRd3OhwNOfU/3m0vz87GEHxVxOaLUWhIt4c/TJPkTqwbytWEMCeguLAwodhM1wzoLxADh7zguqb/i+nj/1rAKCbGMH21c59rYKCiQb4fydyCgOlafeMjyp8xOs07mTAkjAsUz7Cr1gzJ9jXZu0tsK8/dzNdDFh+TAfK1ec75NKkb9HvOWvOb/DhBBFfuiSz6pY2CLDzODR5QQj4NJc1agLbTETO8dvGsfICOIFUiqt+jjhn6bv5x+EEkjBaKm1aAXzHcm6BHS+SLqqDg+s6N0SoA2Uy80oL+w1FAFeA==";
$key = "ef84a15b-ffd0-4c4d-9d56-b652532f";
$iv = md5("ef84a15b-ffd0-4c4d-9d56-b652532f_v03043740000bkdcm2d8kgepf3ld6a0g",1);
print_r(aes_decrypt($data, $key, $iv));
function aes_decrypt($data, $key, $iv) {
    $data = base64_decode($data);
    $decrypted = openssl_decrypt(  
        $data,  
        'aes-256-cbc',  
        $key,  
        OPENSSL_RAW_DATA,
        $iv  
    );
    // 返回解密后的字符串  
    return $decrypted;  
} 

```
