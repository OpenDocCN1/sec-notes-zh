
[href="forum.php?mod=attachment&aid=MjYwNjY0N3xjYjQwNTBhZnwxNzkzNTczODEyfDE2ODA4NnwxNzcyNTQ0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNjY0N3xjYjQwNTBhZnwxNzkzNTczODEyfDE2ODA4NnwxNzcyNTQ0&nothumb=yes)

2023-4-11 19:21 上传

我们IDA看一下这个地址所在的函数是干嘛的:

```c
__int64 __fastcall sub_1001FBF10(__int64 a1, __int64 a2)
{
  __int64 v2; // rax
  __int64 v3; // r13
  void *v4; // rsp
  void *v5; // rsp
  __int64 v6; // rdi
  __int64 v7; // rax
  unsigned int v8; // ebx
  _QWORD v10[6]; // [rsp-8h] [rbp-30h] BYREF

  v10[0] = v2;
  sub_10000B6A0(&unk_100454320);
  v4 = alloca(sub_10000C800(&unk_100454320, a2));
  v5 = alloca(sub_10000C800(&unk_100454320, a2));
  v6 = *(_QWORD *)(*(_QWORD *)v3 + 16LL);
  swift_retain(v6);
  _s7Combine19CurrentValueSubjectC5valuexvg();
  swift_release(v6);
  sub_1001FC3D0(v10, v10, &unk_100454320);
  v7 = sub_1001F95A0(0LL);
  v8 = (*(__int64 (__fastcall **)(_QWORD *, __int64, __int64))(*(_QWORD *)(v7 - 8) + 48LL))(v10, 1LL, v7);
  if ( v8 != 1 )
    sub_1001FC400(v10, &unk_100454320);
  LOBYTE(v8) = v8 != 1;
  sub_1001FC400(v10, &unk_100454320);
  return v8;
}
```

可以看出`v8`返回值为0或者1，因为他有一个`v8!=1`的判断在这里，那么我们如果Hook `0x1001FBF10`这个函数的返回值是否就ok了呢？

编写Frida代码进行内存注入：

```javascript
import { HookApp, log } from "./Utils.js";

HookApp("Paste", (hook, point, method, baseAddr, tools) => {

  // setTimeout(() => {

  //   hook(point(0x1f9f10), (ths, r) => {

  //     // r.replace(ptr(2));

  //   });

  // }, 3000);

  hook(point(0x1fbf10), (ths, r) => {

    r.replace(ptr(1));

  });

});
```

狠狠的按下`F5`启动，我们看看是否注入成功：

![](https://attach.52pojie.cn/forum/202304/11/192132pfmttr0rtf0r1oom.jpg)

`16812068323518.jpg` (652.54 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYwNjY0OHwzMzFjMDE2MXwxNzkzNTczODEyfDE2ODA4NnwxNzcyNTQ0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNjY0OHwzMzFjMDE2MXwxNzkzNTczODEyfDE2ODA4NnwxNzcyNTQ0&nothumb=yes)

2023-4-11 19:21 上传

这里没有去修改有效订阅的那个函数，因为修改了这个信息之后会因为无法读取订阅数据而导致App崩溃，而且`Paste`只检查了`0x1fbf10`这个函数，所以我们直接返回1即可完成。

### 3> 0x02 总结

文中使用的Frida js全文如下:

```javascript
var appBaseAddr = Module.findBaseAddress("Paste");
var func = appBaseAddr.add(0x1fbf10)
Interceptor.attach(func, {
    onEnter(this1, args) {
    },
    onLeave(retval) {
        retval.replace(0x1)
    },
})
```

安装`python`和Frida后，`frida -p pid -l main.js`启动注入。`pid`是进程列表中的进程id。
