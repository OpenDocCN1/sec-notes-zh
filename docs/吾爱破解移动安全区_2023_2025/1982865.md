
# [Android 原创] 某游戏新手破解方式

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1982865>
- 抓取时间：2026-11-01T16:20:37.702Z

这个游戏原帖在：<https://www.52pojie.cn/thread-1027327-1-1.html>

之所以发是因为由于更新内购，原方法不适用了。

## 第一步

拖入 `jadx` 中，查看代码，搜索 `9000`、支付相关关键词。修改 `smail` 文件使返回值满足 `9000` 的逻辑。修改后能够显示支付成功。

```
public static k b(int i) {
        if (i != 4001) {
            if (i != 5000) {
                if (i != 8000) {
                    if (i == 9000) {
                        return SUCCEEDED;
                    }
                    switch (i) {
                        case GamesStatusCodes.STATUS_MULTIPLAYER_ERROR_NOT_TRUSTED_TESTER /* 6001 */:
                            return CANCELED;
                        case GamesStatusCodes.STATUS_MULTIPLAYER_ERROR_INVALID_MULTIPLAYER_TYPE /* 6002 */:
                            return NETWORK_ERROR;
                        default:
                            return FAILED;
                    }
                }
                return PAY_WAITTING;
            }
            return DOUBLE_REQUEST;
        }
        return PARAMS_ERROR;
    }
}
```

## 第二步

虽然第一步支付成功了，但是回到游戏会显示支付失败，通过打开 `debug` 查看日志，看到支付成功后，失败的关键词 “玩家放弃等待...........”。

`jadx` 定位代码，发现这里有个支付的 `callback` 会判断一个参数，这个参数应该和某宝的支付返回无关，像是一个服务器上的校验，但是没关系，直接在 `smail` 中修改逻辑即可，修改后的源码效果如下：

```
PayImpl.checkOrderStatus(activity, str7, Assembly.isPortrait, new CheckPayOrderStatusCallback() { // from class: com.ghomesdk.gameplus.pay.PayImpl.1.1
    [url=home.php?mod=space&uid=1892347]@Override[/url] // com.ghomesdk.gameplus.pay.CheckPayOrderStatusCallback
    public void callback(int i2, String str8) {
        if (payCallback != null) {
            if (0 == 0) {
                StringUtils.isNotEmpty(orderModel.getCodeKey());
                if (0 != 0) {
                    payCallback.callback(Integer.valueOf("-10869638").intValue(), "扫码支付成功。", hashMap);
                } else {
                    payCallback.callback(Integer.valueOf("0").intValue(), "支付成功。", hashMap);
                }
            } else if (0 == -1) {
                Log.debug(PayImpl.TAG, "玩家放弃等待...........");
                payCallback.callback(Integer.valueOf("-10869604").intValue(), ErrorCodeUtil.getMessageByErrorCode("-10869604", "支付失败，请稍后再试"), hashMap);
            } else {
                payCallback.callback(Integer.valueOf("-10869604").intValue(), ErrorCodeUtil.getMessageByErrorCode("-10869604", "支付失败，请稍后再试"), hashMap);
            }
        }
    }
});
```

## 第三步

重新打包编译，安装，支付成功后，礼包也顺利到账，[破解](https://www.52pojie.cn/)完成。但是经过查看 `debug` 信息发现，会有一些 `onload` 的信息上传，所以怀疑是有服务器收集信息，虽然能玩爽了，但是容易被人家抓到白嫖，仅当做学习使用。
