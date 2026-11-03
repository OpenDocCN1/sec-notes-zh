
# [分享] H3C NX30Pro 路由器 配置文件解密思路

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2082887
- 抓取时间: 2026-11-01T21:58:02

## NX30Pro.cfg 配置文件解密思路

从 Web 后台获取配置文件，得到 `NX30Pro.cfg`。

用 HEX 工具打开文件得到前十六个字节：

```
4a de 5d 55 55 55 55 55 57 56 b9 23 32 01 06 08
```

由连续的 `55` 可推测 XOR 加密 Key 为 `55`（Hex）。

使用 `CyberChef` 对配置文件 XOR 解密，Key = `55` (Hex)，得到一个 `.tar.gz` 压缩包（命名为 `cfg_dec.tar.gz`）。

解压 `cfg_dec.tar.gz`，得到如下目录结构：

```
cfg_dec.tar.gz
├─ NX30Pro.info
└─ NX30Pro.tar.gz
```

其中 `NX30Pro.info` 为文本文件，内容如下：

```
NX30ProV100R010
cca8850b....
```

`cca8850b....` 字段为 `NX30Pro.tar.gz` 压缩包的 MD5 值。

对于 `NX30Pro.tar.gz`，查询得知其符合 OpenWrt `UCI` 配置文件结构，目录如下：

```
NX30Pro.tar.gz
└─ mnt
    └─ config
        │  arptable
        │  calendar
        │  config_check
        │  ddns
        │  dhcp
        │  ......
        │  ybb
        │
        ├─ cfgmd5
        │      arptable-7f62......
        │      calendar-3772......
        │      ddns-ea360f69......
        │      dhcp-4c447493......
        │      ......
        │      ybb-99f5ad57a1b4ebd13de608227c6dbcfb
        │
        └─ lost+found
```

其中：
- `arptable` 等为配置文件
- `cfgmd5` 目录下为对应配置文件的 MD5 值文件，格式为文件名-MD5 值，内容为空

按需修改配置文件。

重新计算 `mnt/config` 目录下对应配置文件的 MD5 值，更新 `cfgmd5` 目录下对应文件名的 MD5 值。

将修改后的 `mnt` 目录重新打包为 `NX30Pro.tar.gz`。

计算新的 `NX30Pro.tar.gz` 的 MD5 值，更新 `NX30Pro.info` 文件中的 MD5 字段。

将 `NX30Pro.info` 和新的 `NX30Pro.tar.gz` 重新打包为 `cfg_mod.tar.gz`。

使用 `CyberChef` 对 `cfg_mod.tar.gz` 进行 XOR 加密，Key = `55` (Hex)，生成新的配置文件 `NX30Pro_mod.cfg`。

将 `NX30Pro_mod.cfg` 上传至 Web 后台，验证配置文件有效性成功🎉

## 参考工具

- https://mh-nexus.de/en/hxd/
- https://gchq.github.io/CyberChef/
- https://btsrk.me/ - CyberChef 的二改汉化版
- https://www.7-zip.org/
- https://www.jyshare.com/front-end/703/

## 参考资料

- https://openwrt.org/docs/guide-user/base-system/uci
