
```java
String vd2 = SecurityTool.getVD(HeyBoxApplication.C(), SecurityTool.getVX(HeyBoxApplication.C(), "HPPDCEAENEHBFHPASRDCAMNHJLAAPF"));
HeyBoxApplication.C(), SecurityTool.getVX(HeyBoxApplication.C(), "HPPDCEAENEHBFHPASRDCAMNHJLAAPF")
SecurityTool.getVX(HeyBoxApplication.C(), "HPPDCEAENEHBFHPASRDCAMNHJLAAPF")
```

这里有两个参数，一个是 `HeyBoxApplication.C()`，一个是 `SecurityTool.getVX(HeyBoxApplication.C(), "HPPDCEAENEHBFHPASRDCAMNHJLAAPF")` 函数调用。

```text
SecurityTool.getVD is called: context=com.max.xiaoheihe.app.HeyBoxApplication@c790643, str=O2eqUFZC94Ix3NCBI6lc77lsPh1qWayE
SecurityTool.getVD result=NqY5fY1BmjOy4AjzT90Of3wcJA1KMyCw
```

从日志可以看出第一个参数是一个设备的上下文信息，第二个参数是 `getVX()` 函数生成的 32 位字符串。

![](https://attach.52pojie.cn/forum/202510/12/174554uaogjjquhaguyyhq.png)

跟进去就可以看到一堆 so 层的 native 方法，我们找一下 so 文件的加载，然后去分析 so 层的逻辑。

![](https://attach.52pojie.cn/forum/202510/12/174623t3bcvtv3ta8cvauu.png)

找到了，开干。

## 四、so层分析

### 1、getVX

在文件里定位到 so 文件，拖进 IDA 进行分析。

![](https://attach.52pojie.cn/forum/202510/12/174726t2d82lgze8w223vx.png)

现在导出函数表搜索 Java，发现不是静态注册函数，我们 hook 一下 `RegisterNatives()` 函数，找到动态注册函数的偏移。

```javascript
function hook_RegisterNatives() {

  var addrRegisterNatives = null;

  var symbols = Module.enumerateSymbolsSync("libart.so");

  for (var i = 0; i = 0 &&

        symbol.name.indexOf("JNI") >= 0 &&

        symbol.name.indexOf("RegisterNatives") >= 0 &&

        symbol.name.indexOf("CheckJNI") ')

        var env = args[0];        

        var java_class = args[1]; 

        var class_name = Java.vm.tryGetEnv().getClassName(java_class);

        var taget_class = "com.max.security.SecurityTool";   

        if (class_name === taget_class) {

          console.log("\n[RegisterNatives] method_count:", args[3]);

          var methods_ptr = ptr(args[2]);

          var method_count = parseInt(args[3]);

          for (var i = 0; i 
```

需要注意的是这里注入的时间，需要跟着 Frida 检测处理的同时注入，也就是 so 加载的时机。

```text
setKA sig: (Ljava/lang/String;)V module_name: libhbsecurity.so offset: 0xa35c0 8
setKB sig: (Ljava/lang/String;Ljava/lang/String;)V module_name: libhbsecurity.so offset: 0xa3c34 8
setKM sig: (Ljava/lang/String;Ljava/lang/String;)V module_name: libhbsecurity.so offset: 0xa3d80 8
setKT sig: (Ljava/lang/String;Ljava/lang/String;)V module_name: libhbsecurity.so offset: 0xa3ecc 8
setKN sig: (Ljava/lang/String;Ljava/lang/String;)V module_name: libhbsecurity.so offset: 0xa3fd8 8
setKD sig: (Ljava/lang/String;Ljava/lang/String;)V module_name: libhbsecurity.so offset: 0xa40e0 8
setKC sig: (Ljava/lang/String;Ljava/lang/String;)V module_name: libhbsecurity.so offset: 0xa42d8 8
getVX sig: (Landroid/content/Context;Ljava/lang/String;)Ljava/lang/String; module_name: libhbsecurity.so offset: 0xa45d0 8
getVA sig: (Landroid/content/Context;Ljava/lang/String;)Ljava/lang/String; module_name: libhbsecurity.so offset: 0xa4834 8
getVB sig: (I)I module_name: libhbsecurity.so offset: 0xa5954 8
getVC sig: (Landroid/content/Context;Ljava/lang/String;)Ljava/lang/String; module_name: libhbsecurity.so offset: 0xa5a58 8
getVD sig: (Landroid/content/Context;Ljava/lang/String;)Ljava/lang/String; module_name: libhbsecurity.so offset: 0xa5e44 8
resetVA sig: ()V module_name: libhbsecurity.so offset: 0xa6428 8
```

根据 `getVX()` 函数偏移 `0xa45d0`，在 IDA 中利用快捷键 `G` 定位。

![](https://attach.52pojie.cn/forum/202510/12/174948s0qifx0qa7w00qqq.png)

修改 IDA 中函数和入参的名称，方便再定位修改。

![](https://attach.52pojie.cn/forum/202510/12/175037v2ayy525fvytxaua.png)

这里的话只有 `sub_A2B50()` 对入参 `str` 进行了一个处理，hook 一下看看情况。

```javascript
function print_arg(addr) {

    var module = Process.findRangeByAddress(addr);

    if (module != null) return hexdump(addr) + "\n";

    return ptr(addr) + "\n";

}

function hook_native_addr1(funcPtr, paramsNum) {

  var module = Process.findModuleByAddress(funcPtr);

  Interceptor.attach(funcPtr, {

    onEnter: function (args) {

      this.logs = [];

      this.params = [];

      this.logs.push("call " + module.name + "!" + ptr(funcPtr).sub(module.base) + "\n");

      for (let i = 0; i 
```

从日志内容（后续会上传日志文本，太多了）可以看出，没有明显的加密字符串返回，说明这里不是一个加密逻辑的点。然后 `v6` 对 `v5` 处理后的结果，看着像是地址的偏移，也可以来 hook 一下。

```javascript
function print_arg(addr) {
  var module = Process.findRangeByAddress(addr);
  if (module != null) return hexdump(addr) + "\n";
  return ptr(addr) + "\n";
}
function hook_native_addr1(funcPtr, paramsNum) {
  var module = Process.findModuleByAddress(funcPtr);
  Interceptor.attach(funcPtr, {
    onEnter: function (args) {
      this.logs = [];
      this.params = [];
      this.logs.push("call " + module.name + "!" + ptr(funcPtr).sub(module.base) + "\n");
      for (let i = 0; i 
```

（日志分析）都是一些空值，可能是指针的一些信息，不能直接用 `hexdump` 打印出来，我们接下来看的 `v6` 又进行了哪些处理。

![](https://attach.52pojie.cn/forum/202510/12/175314kbyyly2vhontoxft.png)

`v8` 是由 `v6` 生成的，我们直接 hook 看看。

![](https://attach.52pojie.cn/forum/202510/12/175340tqg5td0pbjj6qb5p.png)

这里就会发现很像我们最后生成的结果，我们跟进去看看怎么去使用 `v6`。

```c
__int64 sub_A6620(){
    __int64 v0; // x19
    unsigned __int8 v1; // w8
    __int64 result; // x0
    v0 = sub_72AE4(0x21);
    *(_BYTE *)v0 = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 1) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 2) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 3) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 4) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 5) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 6) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 7) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 8) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 9) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 10) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 11) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 12) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 13) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 14) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 15) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 16) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 17) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 18) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 19) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 20) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 21) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 22) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 23) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 24) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 25) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 26) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 27) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 28) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 29) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    *(_BYTE *)(v0 + 30) = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    v1 = byte_230F4[(unsigned int)sub_4AC1C(0x3Eu)];
    result = v0;
    *(_WORD *)(v0 + 31) = v1;
    return result;
}
```

我们在日志中发现他不停地去调用生成新的数字。可以看到 `sub_4AC1C()` 不断去处理 `0x3Eu` 来填充 `v0` 的字段长度，我们去 hook `sub_4AC1C()` 处理情况。

![](https://attach.52pojie.cn/forum/202510/12/175444ee8ix9hw9h8pa7mz.png)

`byte_230F4` 可能是一个数组，用来存储数据映射后面产生的随机数，填充生成我们的加密字符串，我们需要根据汇编代码去找 `byte_230F4` 的值。

![](https://attach.52pojie.cn/forum/202510/12/175554y3lpltqq3ztqauaq.png)

ARM64 的 `LDRB` 指令支持多种寻址方式，最常见的格式为：
