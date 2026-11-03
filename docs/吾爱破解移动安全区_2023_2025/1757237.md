
# [Android 原创] 第十课、frida环境搭建与使用

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1757237
- 抓取时间: 2026-11-01T21:47:19

## 第十课、frida环境搭建与使用

### frida检测名字与端口

#### 修改名字与端口
`mv frida-server-12.8.0-android-arm64 fs1280a64`

#### 查看端口的命令
`netstat -tuulp |grep fs`
默认的是`27042`

#### 开启自定义端口
`./fs1280a64 -v -l 0.0.0.0:8787 listen`

在 `https://www.52pojie.cn/thread-661779-1-1.html` 上命令

`frida-ps -H 192.168.31.97:8787` 查看进程

`frida -H 192.168.31.97:8787 -F` attach前台程序

#### objection使用
`objection -N -h 192.168.31.97 -p 8787 -g com.android.settings explore`

`dpkg -i dwada.deb` 安装deb格式软件

## frida脚本开发环境正确操作

之前建议的这种方法 `frida-gum.d.ts` 已经被大胡子删掉了，他现在不赞成使用这种方式。

目前最新正确的Frida环境搭建方法：
1. `git clone https://github.com/oleavr/frida-agent-example.git`
2. `cd frida-agent-example/`
3. `npm install`
4. 使用VSCode等IDE打开此工程，在 `agent` 下编写 `typescript`，会有智能提示。
5. `npm run watch` 会监控代码修改自动编译生成 `js` 文件
6. `frida -U -f com.example.android --no-pause -l _agent.js`

### vscode使用脚本
`frida -H 192.168.31.97:8787 -F -l _agent.js` 这个ip地址是要写手机的ip地址

- `r`
- `cn.soulapp.android`

### RPC调用

```
function hook_KeyStore_load() {
Java.perform(function () {
var StringClass = Java.use("java.lang.String");
var KeyStore = Java.use("java.security.KeyStore");
KeyStore.load.overload('java.security.KeyStore$LoadStoreParameter').implementation = function (arg0) {
console.log(Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new()));
console.log("KeyStore.load1:", arg0);
this.load(arg0);
};
KeyStore.load.overload('java.io.InputStream', '[C').implementation = function (arg0, arg1) {
console.log(Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new()));
console.log("KeyStore.load2:", arg0, arg1 ? StringClass.$new(arg1) : null);
this.load(arg0, arg1);
};

console.log("hook_KeyStore_load...");
});
}
```

`setImmediate(hook_KeyStore_load)`
`debug_print('Resuming PID test {pid}'.format(pid=self.spawned_pid))`
`self.device.resume(self.spawned_pid)`
