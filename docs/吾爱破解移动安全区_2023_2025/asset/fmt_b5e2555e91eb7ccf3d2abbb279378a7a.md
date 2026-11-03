
# [Android 原创] 《安卓逆向这档事》十八、表哥，你也不想你的Frida被检测吧!(上)

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1921073
- 抓取时间: 2026-11-02T01:14:56

![](https://pic.rmb.bdstatic.com/bjh/240506/4dc9b4a36e937c443bbce777ada6b57e4327.png)

## 一、课程目标

1. 了解常见 `frida` 检测

## 二、工具

1. 教程Demo(更新)
2. `jadx-gui`
3. VS Code
4. `jeb`

## 三、课程内容

### 1. 检测文件名、端口名、双进程保护、失效的检测点

1. 检测 `/data/local/tmp` 路径下的是否有 `frida` 特征文件，server端改名，例如:`fr`

2. 指定端口转发

```
./fs1 -l 0.0.0.0:6666
adb forward tcp:6666 tcp:6666
frida -H 127.0.0.1:6666 wuaipojie -l hook.js
```

3. `spawn` 启动过双进程保护

```
frida -U -f 进程名 -l hook.js
```

PS:学会看注入报错的日志，比如说当app主动附加自身进程时，这时候再注入就会提示 `run frida as root`（以 `spawn` 的方式启动进程即可）

4. 借助脚本定位检测 `frida` 的so

```
function hook_dlopen() {
    Interceptor.attach(Module.findExportByName(null, "android_dlopen_ext"),
        {
            onEnter: function (args) {
                var pathptr = args[0];
                if (pathptr !== undefined && pathptr != null) {
                    var path = ptr(pathptr).readCString();
                    console.log("load " + path);
                }
            }
        }
    );
}
```

5. 随着 `firda` 的版本迭代，以前诸多检测点以失效

#### (1.) 例如检测 D-Bus

D-Bus是一种进程间通信(IPC)和远程过程调用(RPC)机制,最初是为Linux开发的,目的是用一个统一的协议替代现有的和竞争的IPC解决方案。

```
bool check_dbus() {

    // 定义一个socket地址结构体变量sa

    struct sockaddr_in sa;

    // 创建一个socket文件描述符

    int sock;

    // 定义一个字符数组res，用于存储接收到的数据

    char res[7];

    // 循环遍历所有可能的端口号，从0到65535

    for(int i = 0; i
```

#### (2) 检测 fd

`/proc/pid/fd` 目录的作用在于提供了一种方便的方式来查看进程的文件描述符信息，这对于调试和监控进程非常有用。通过查看文件描述符信息，可以了解进程打开了哪些文件、网络连接等，帮助开发者和系统管理员进行问题排查和分析工作。

```
bool check_fd() {
    DIR *dir = NULL;
    struct dirent *entry;
    char link_name[100];
    char buf[100];
    bool ret = false;
    if ((dir = opendir("/proc/self/fd/")) == NULL) {
        LOGI(" %s - %d  error:%s", __FILE__, __LINE__, strerror(errno));
    } else {
        entry = readdir(dir);
        while (entry) {
            switch (entry->d_type) {
                case DT_LNK:
                    sprintf(link_name, "%s/%s", "/proc/self/fd/", entry->d_name);
                    readlink(link_name, buf, sizeof(buf));
                    if (strstr(buf, "frida") || strstr(buf, "gum-js-loop") ||
                        strstr(buf, "gmain") ||
                        strstr(buf, "-gadget") || strstr(buf, "linjector")) {
                        LOGI("check_fd -> find frida:%s", buf);
                        ret = true;
                    }
                    break;
                default:
                    break;
            }
            entry = readdir(dir);
        }
    }
    closedir(dir);
    return ret;
}
```

#### (3) 检测文件

众所周知 `frida` 我们一般都会放在 `data/local/tmp` 目录下，旧版 `fridaserver` 端运行时都会释放到 `re.frida.server`，所以这里在旧版也会被当做一个检测点，而新版已不再释放

##### 3> 2检测map

```
adb shell ps | findstr com.zj.wuaipojie
cat /proc/12186/maps|grep frida
```

![](https://pic.rmb.bdstatic.com/bjh/240506/8d434c16d1a2af86d824723e4525de9e8343.png)

| 字段 | 描述 |
| --- | --- |
| u0_a504 |  |
