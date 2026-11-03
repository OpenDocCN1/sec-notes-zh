
```java
public String getBuy_vip_tips() {

        return this.buy_vip_tips;

    }

    public void setBuy_vip_tips(String buy_vip_tips) {

        this.buy_vip_tips = buy_vip_tips;

    }

    public UserInfo getUserInfo() {

        return this.userinfo;

    }

    public void setUserInfo(UserInfo userInfo) {

        this.userinfo = userInfo;

    }

    public InitUpdate getUpdate() {

        return this.update;

    }

    public void setUpdate(InitUpdate update) {

        this.update = update;

    }

    public InitShare getShare() {

        return this.share;

    }

    public void setShare(InitShare share) {

        this.share = share;

    }

}
```
OK，应该是找到正主了，继续跳转`UserInfo`，看看有什么属性

```java
//
// Decompiled by Jadx - 503ms
//
package net.pengtu.sxscq.bean;

public class UserInfo {
    private String app_account_token;
    private String avatar;
    private long expire_time;
    private String expire_time_str;
    private String nickname;
    private String uid;
    private int user_type;
    private String user_type_str;

    public long getExpire_time() {
        return this.expire_time;
    }

    public void setExpire_time(long expire_time) {
        this.expire_time = expire_time;
    }

    public String getUid() {
        return this.uid;
    }

    public void setUid(String uid) {
        this.uid = uid;
    }

    public String getApp_account_token() {
        return this.app_account_token;
    }

    public void setApp_account_token(String app_account_token) {
        this.app_account_token = app_account_token;
    }

    public String getAvatar() {
        return this.avatar;
    }

    public void setAvatar(String avatar) {
        this.avatar = avatar;
    }

    public String getNickname() {
        return this.nickname;
    }

    public void setNickname(String nickname) {
        this.nickname = nickname;
    }

    public int getUser_type() {
        return this.user_type;
    }

    public void setUser_type(int user_type) {
        this.user_type = user_type;
    }

    public String getUser_type_str() {
        return this.user_type_str;
    }

    public void setUser_type_str(String user_type_str) {
        this.user_type_str = user_type_str;
    }

    public String getExpire_time_str() {
        return this.expire_time_str;
    }

    public void setExpire_time_str(String expire_time_str) {
        this.expire_time_str = expire_time_str;
    }
}
```
再次回顾我们的`InitData`方法，参考`initBean.getUserInfo) ≠ null`以下，我删除无用的放到下面给大家看看，

```
this.vip.postValue(Boolean.valueOf(initBean.getUserInfo().getUser_type() == 1));

SPUtils.getInstance().put("VIP", initBean.getUserInfo().getUser_type() == 1);

this.isVip.set(initBean.getUserInfo().getUser_type() == 1);

this.user_type_str.set(initBean.getUserInfo().getUser_type_str());

this.expire_time_str.set(initBean.getUserInfo().getExpire_time_str());
```
所以我们只需要修改`getUser_type`、`getUser_type_str`、`getExpire_time_str`这三个方法。

修改如下：

```
.method public getExpire_time_str()Ljava/lang/String;
    .registers 2

    .line 81
    iget-object v0, p0, Lnet/pengtu/sxscq/bean/UserInfo;->expire_time_str:Ljava/lang/String;
const-string v0,"2099-12-32"
    return-object v0
.end method

.method public getUser_type()I
    .registers 2

    .line 65
    iget v0, p0, Lnet/pengtu/sxscq/bean/UserInfo;->user_type:I
const/4 v0,0x1
    return v0
.end method

.method public getUser_type_str()Ljava/lang/String;
    .registers 2

    .line 73
    iget-object v0, p0, Lnet/pengtu/sxscq/bean/UserInfo;->user_type_str:Ljava/lang/String;
const-string v0,"VIP"
    return-object v0
.end method
```
### 效果图

![](https://attach.52pojie.cn/forum/202407/03/154543ol7073l10550fdq6.png)

### 使用体验

![](https://attach.52pojie.cn/forum/202407/03/154541mauahq8uddrussus.png)
