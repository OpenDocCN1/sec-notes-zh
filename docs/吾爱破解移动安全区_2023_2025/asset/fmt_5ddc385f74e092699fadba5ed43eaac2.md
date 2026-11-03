
# [Android 原创] 第一课、环境部署 r0env

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1755913
- 抓取时间：2026-11-01T21:47:35

## 安装设置 r0env

- https://mp.weixin.qq.com/s/umxxDAxRZw3kyCPDV-63RQ
- https://mp.weixin.qq.com/s/gBdcaAx8EInRXPUGeJ5ljQ

### 虚拟机环境

- 下载解压后，打开 `vmware`
- 点击“打开”，选择解压后的文件
- 编辑[设置](https://www.52pojie.cn/thread-661779-1-1.html)

### 系统与工具配置

1. 更新系统源

   ```
   apt update
   ```

2. KaliLinux 默认不带中文，安装中文字体

   ```
   apt install xfonts-inti-chinese
   apt install ttf-wqy-microhei
   ```

3. 简单的防盗版

   ```
   nano /etc/hostname
   ```

   `ctrl + x` 退出

4. 链接手机
   - 数据线
   - `wifi adb`

5. 快速调出 `terminal`

   `ctrl + alt + T`

6. 安装几个必备软件

   ```
   apt install tmux jnettop htop
   jnettop
   ```

   实时看下载的流量和端口

7. `htop`

   看当前系统的内存

8. 查看当前手机架构

   ```
   uname -a
   ```

   x86 32位
