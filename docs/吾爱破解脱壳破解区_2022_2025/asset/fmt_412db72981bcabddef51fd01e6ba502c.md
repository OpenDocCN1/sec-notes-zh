
## 补丁内容

```diff
@@ -0,0 +1,3 @@
+#pragma once
+
+void disable_md5_verify();

diff --git a/src/patch.cpp b/src/patch.cpp
index 2661e82..68f0b1e 100644
--- a/src/patch.cpp
+++ b/src/patch.cpp
@@ -8,6 +8,7 @@
  */
 #include "patch.h"
 #include "MemoryRWGuard.hpp"
+#include "disable_md5_verify.h"
 #include "hook_md5_self_verify.h"
 #include "hook_md5_update.h"
 #include "utils.h"
@@ -33,11 +34,15 @@ inline void patch_machine_code() {
 }
 
 void init_patch() {
+#if DISABLE_MD5_VERIFY
+  disable_md5_verify();
+#else
   // Backup memory
   patch_md5_update();
   // Patch file verification
   patch_md5_self_verify();
+#endif
   // Hardcode machine code
   patch_machine_code();
```

## 编译指令

然后执行下述指令进行编译即可：

```bash
make clean all "OPT_DEF_FLAGS=-DDISABLE_MD5_VERIFY=1"
```

当然，这需要你提前解密并将解密后的文件提换掉软件目录下的 `SxtqCoef.bin` 文件 :D

.

原文使用 Markdown 撰写。

## 原版程序下载

- [蓝奏](https://wws.lanzouq.com/b0ds8obwh) 密码：52pj
- [百度](https://pan.baidu.com/s/13xb5Kk7ugRc9eeXyaI0NXg?pwd=wq6n)

## 文件说明

- 其中 `7z` 压缩格式为安装后提取的绿色版，其中 `sxtq2_2016_收官模式_答案.7z` 为非必需文件（收官模式需要）
- `zip` 压缩格式为原始安装程序，可能会有捆绑。

注：若转载请注明来源（本贴地址）与作者信息。

> 伤心跳× 2.1（2008）及 10.4（2016）注册算法+自校验分析（吾爱破解论坛 - 爱飞的猫）
>
> https://www.52pojie.cn/thread-1658431-1-1.html
