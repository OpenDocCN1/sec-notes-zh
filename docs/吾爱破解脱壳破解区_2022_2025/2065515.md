
# [原创] 某电商平台客户联系方式加密算法

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2065515
- 抓取时间: 2026-11-01T22:01:43

## 起因

原因呢是像查看电商平台的财务报表，导出的时候勾选可客户资料。但可能实际情况是不支持导出客户资料，所以导出都是空白。

而后台页面呢是有客户姓名和联系方式的，但是每次都要点击，而且大概率是由频次风控。所以已放弃获取了。

顺手抓了个获取的包，发现包还是加密的。所幸比较简单，就记录了下。

## 经过

来包内容：

```json
{"success":true,"error_code":0,"error_msg":"成功","result":{"order_sn":"251011-076934049282793","virtual_mobile":"NjAzMTYwMzc2MD脱敏处理wMzk2MDM5NjAzNzYwMzA=","show_virtual_report_button":true,"additional_report_reason_list":[],"show_virtual_tip_after_mobile":true,"show_virtual_risk_tip":false,"can_virtual_extend":false,"ext_number":"3482","cannot_extend_reason_type":1,"virtual_expired_timestamp":1770527802,"virtual_extend_expired_timestamp":1770527802,"show_export_popup":false,"mobile_from_order_print":false,"has_report":false,"valid_appeal":false,"web_spider_rule":"e5089033a9604513896fbb936cabd1faAv4063vwCz0y4a7tZ5rIs/EW4WKL7OvhXvrQowDrMhrSwchrCTtX1XvXo9FXgK4kf5D|$.contact_mobile|$.virtual_mobile|$.mobile"}}
```

`virtual_mobile`典型的`base64`，`base64decode`。得到`603x603x603x603x603x603x603x603x603x603x603x`。

## 结果

把得到的 11 个 `x` 串起来，就是手机号。
