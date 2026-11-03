
```cpp
#include "pch.h"
#include 
#include 
#include 
#include "MinHook.h"
#pragma comment(lib, "libMinHook-x64-v120-mdd.lib")

// 函数指针定义
typedef int(__stdcall* TargetFunction_t)(int, int);
TargetFunction_t originalFunction = nullptr;

uintptr_t __fastcall HookedFunction(int param1, int param2) {
    uintptr_t result = originalFunction(param1, param2);
    LONG64* ptr = reinterpret_cast(result);
    char message[64];
    sprintf_s(message, " Address: 0x%p", (LONG64*)*ptr);
    MessageBoxA(NULL, message, "Hook Info", MB_OK | MB_ICONINFORMATION);
    // 替换为自己的公钥，注意长度392要与程序原公钥保持一致，
    memcpy_s((LONG64*)*ptr, 392, "XXXXXXXXX", 392);
    return result; // 修改返回值
}

void Execute()
{
    MH_Initialize();

    HMODULE hModule = GetModuleHandle(L"libcc.dll");
    if (!hModule) {
        printf("Failed to get module handle\n");
        return;
    }
    uintptr_t targetAddress = (uintptr_t)hModule + 0x1C190A0; // 替换为实际 RVA

    MH_CreateHook((LPVOID)targetAddress, &HookedFunction,
        reinterpret_cast(&originalFunction));
    MH_EnableHook((LPVOID)targetAddress);
}

BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
        Execute();
        break;
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```

使用的公钥跟激活状态类似于有一种绑定关系，也就是说，你用一个公钥激活之后，又使用另外的公钥制作了新的`dll`，注入新`dll`启动程序将仍是未激活状态。替换什么样的公钥激活，以后就用这个注入启动。有条件的就自己编译代码，使用自己的`RSA2048`位密钥对。文末附件是生成好的对应`V17.1.3`版本，注释掉了弹窗信息，对应的密钥对是：

```
-----BEGINPUBLICKEY-----

MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAijqbHNPRBmVNQb9KNZu0bPbRW/M/pFOQWoGh4uaPpIsW/LwjTegEBNklw9tuXXeUN4RNheeWaImcHnBxe6gvoYjMBPh2nMCWKhId3ba9JbIjbcRi/O7y6+F6votbqLT05FI25QV4coLeOQyb0NehOdQbah773Q88cJePgLnsoqthGOXmPWv1I/3JyrEf9OqF7JSNvik/2kmoTHHW63uWR/ExQLhOsHx7lIMUh3EwXraxWX/6aXQgS7On2mq2siWyxfKWS6GO93sEVDqFhB7/P358Lpiu/S+2QZNPm0ayu/m02fWI3kxLh+y9b9icoKKN2tLRWZdrbYIpyX3TbJcZOwIDAQAB

-----ENDPUBLICKEY-----

-----BEGINRSAPRIVATEKEY-----

MIIEogIBAAKCAQEAijqbHNPRBmVNQb9KNZu0bPbRW/M/pFOQWoGh4uaPpIsW/LwjTegEBNklw9tuXXeUN4RNheeWaImcHnBxe6gvoYjMBPh2nMCWKhId3ba9JbIjbcRi/O7y6+F6votbqLT05FI25QV4coLeOQyb0NehOdQbah773Q88cJePgLnsoqthGOXmPWv1I/3JyrEf9OqF7JSNvik/2kmoTHHW63uWR/ExQLhOsHx7lIMUh3EwXraxWX/6aXQgS7On2mq2siWyxfKWS6GO93sEVDqFhB7/P358Lpiu/S+2QZNPm0ayu/m02fWI3kxLh+y9b9icoKKN2tLRWZdrbYIpyX3TbJcZOwIDAQABAoIBACm2j9J+NSjJ7yt1D27oAuWTAhDOchkjwn8deX3gd95+vrwcBkdfSb8wzE2zbTW1fTvXqBFNlr0LMdqZshVP5rfHcVHarOiW0Xn2p56do6y5eimIxxpu/U2jaIgRw+JAYVp5WOl54iQkS+1JWcgHGjc6Q9+dZAIMs/+CLSL6eGeJ/vTUzVzcM85InXsjquEh5jTDTo1ADPfWZWnebrWjGWZc2XUeDw0FAaQM8gYWdtMQnNln11YJ+kMcWsfK5j2qnJHvMvUR8gHPH3bPaInUp0Wux/lqwtdWGyMh/Pgcfq1nY8NT3FsmP5TEUEnIWM6Kgq/dPHKXahjkVYkIqoT/93ECgYEA33pnsraA3EbhvWxNnLb3mjkdMH7H3tD6zQxytRnFr7fbGPenaUHX6iLSRoETUuvJO6uYpP/sNAJcMwY/mmNYmD1Z8n3Ct0+WplOshtVnZbsB9CrU33HMnMD4cFmVmQhHBh2UWFPjx08dEND9a3ILOdHOWi1Y1qCJcrlrT2iSrNMCgYEAnlhF6rTjqJC7idddT+T48sk0wvhIRvR51A/88zFb0hDkYMYqkAmV8gixDL1eBYE5s/D447X2kTWEEF2uHqS28JzevBwyp+uaJ2DCMd/MspYaXa8mtwojf3zWJ+LNyasgwkxj4tlB3kMxpjLrDeJe7Uc96I3v5Y5ukVNG1Fo/APkCgYBCoWhRrIB3LNsZczUgyJzCasz9LI/DK2Pbxx5cSLbcUWEN7KkJhQwvTvqK8ukkDBEPrXXxnCx8xUdgHzjPe1dq7L/owP5RUXWFWeVSvkX52uwWNL8LcA6lnhN0zAECg6lJdYBJZZnATyJHZUhpxLL8JkX0NRkLxQm2HvqTzoYM6QKBgGTQkwNSVDg4ng6u/gwmHpKFpIAWD/6NHRv1afl0jZ97fcxXnzXBxKQrAH65+qjKk/CLFgv1CJ1/UywQWRHNkWQ0nGNsGJxtFuLHahzS/sMdaoAHSkDzxZIf60HN13P07pCGjnoaFSPr6CqG9L0MSaAM4c+0vu6K9r60JL6DDuYRAoGAB2YqHtWf1QKetjmn5pQGv8K0GEnLl/KDC+Ko7yWRM3rjxeVGPBv9ioocO+5DE9XlEIUsQct6D6TVzi1SHAk3tNLw0g/HI6f6pbMn7fw+jFSSsVREhSwkpiD59JeNoXKILFQlDaWuL/Jj78tOqnKlbo5PKIanevzOTZykm6ZrrcE=

-----ENDRSAPRIVATEKEY-----
```

在程序启动后加载制作的`dll`进行`hook`，可以使用`x64dbg`调试的方法，将程序挂起在入口点，打开`ProcessHacker`，找到进程，点击右键，`Miscellaneous->Inject DLL`，选择生成的`dll`即可。注入后继续运行程序，此时应有显示公钥地址的弹窗，说明已找到原公钥，点击“确定”即可成功`hook`替换为自己的`RSA`公钥。

![](https://attach.52pojie.cn/forum/202508/13/094502uoiqvdrgd8ypdiw3.png)

Snipaste_2025-08-07_17-20-16.png (38.75 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=Mjc5NjQ2NHw3ZmJhMmI3ZnwxNzkzNTcwNzQyfDE2ODA4NnwyMDUyOTY5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5NjQ2NHw3ZmJhMmI3ZnwxNzkzNTcwNzQyfDE2ODA4NnwyMDUyOTY5&nothumb=yes)

2025-8-13 09:45 上传

![](https://attach.52pojie.cn/forum/202508/13/094504ws9ss9r9bs9z51br.png)

Snipaste_2025-08-07_17-26-21.png (67.24 KB, 下载次数: 6)

[href="forum.php?mod=attachment&aid=Mjc5NjQ2NXw3MWRhZjBhMnwxNzkzNTcwNzQyfDE2ODA4NnwyMDUyOTY5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5NjQ2NXw3MWRhZjBhMnwxNzkzNTcwNzQyfDE2ODA4NnwyMDUyOTY5&nothumb=yes)

2025-8-13 09:45 上传

## 总结

1. 生成一对`2048`位的`RSA`公私钥，制作替换公钥的`hook dll`，需要`hook`的函数是`libcc.dll`中`sub_181C190A0`。

2. 程序启动后中断挂起，注入`dll`后继续运行。或进行第6步操作。

3. 根据文中“许可证密钥生成”一节（或参考教程）选择`8`字节，进行`ECB`模式、无填充的`DES`加密，密钥为 `E9 7F B0 60 77 45 90 AE`，生成`8`字节。将`8`字节追加到以`0x682A`开头的`2`字节，组成新的`10`字节。对`10`字节进行`BASE32`编码，得到`16`个字符即为许可证密钥。

4. 断网状态下进行手动激活流程。选择`RSA-2048`位`PKCS1`填充模式，使用生成的私钥对请求码解密，得到`K`、`DI`、`P`字段的`JSON`格式字符串。

5. 保持原数据内容不变，增加`N`、`O`、`T`字段。提供私钥，选择私钥加密的方式对新字符串加密，得到的密文即为激活码。

6. 若激活成功，但仍提示过期则继续本步骤，否则可忽略。制作程序启动器`launch`，运行程序时以启动器代替原程序。本步骤目的是程序启动时注入替换`RSA`公钥的`hook`代码，能实现此目的的方式皆可。

本篇教程的难点是对各种加密算法的分析识别，熟悉密码学的话，破起来会简单点。教程主要针对版本`17.1.3`，理论上，如果激活逻辑流程不变的话，本文所讲方法是适用的，只不过关键函数地址、密钥值等可能会有变化。

## 最后关于更新

本文不推荐你一直更新，本来这个教程已经结束，但是出于好奇心我点击了“检查更新”，重启后进入软件你会看到如下界面：

![](https://attach.52pojie.cn/forum/202508/13/094520cwgueeszgggnu8zg.png)

Snipaste_2025-08-12_17-09-34.png (37.31 KB, 下载次数: 6)

[href="forum.php?mod=attachment&aid=Mjc5NjQ3MnwyZTczYmIxZHwxNzkzNTcwNzQyfDE2ODA4NnwyMDUyOTY5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5NjQ3MnwyZTczYmIxZHwxNzkzNTcwNzQyfDE2ODA4NnwyMDUyOTY5&nothumb=yes)

2025-8-13 09:45 上传

没办法只能尝试`launch`启动，发现并没有`hook`到公钥的弹窗。仔细看软件目录，`libcc.dll`的修改日期发生变化，是连同软件一起更新了，加载公钥的函数地址变化导致不能`hook`。那么这个时候屏幕前的老铁不要慌，结合之前的分析，对比前后文件，可以找一个公钥碎片字符，在`.rdata`中搜索，找到后下硬件断点，重新加载程序运行，断点命中后，回溯调用堆栈，可以找出新的函数地址。然后修改`hook`代码中的函数`RVA`（`V17.1.3`对应`0x1C190A0`，`V17.3.2`对应`0x215B240`），制作对应版本的`dll`。本来以为再次`launch`启动后还要重新激活，但是不用，现在最新版本已经可用了。还好成功，万一这方法不可行，那么连我都要慌了... 不过这次也证明了教程方法的可用性。如果以后想继续更新软件，那么就要冒此方法失效的风险。若不想更新，也可直接修改本机的`hosts`文件。

![](https://attach.52pojie.cn/forum/202508/13/094523rxijgjeigzaxznsx.png)

Snipaste_2025-08-12_17-34-39.png (87.08 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=Mjc5NjQ3M3w2ODRhMThlMnwxNzkzNTcwNzQyfDE2ODA4NnwyMDUyOTY5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5NjQ3M3w2ODRhMThlMnwxNzkzNTcwNzQyfDE2ODA4NnwyMDUyOTY5&nothumb=yes)

2025-8-13 09:45 上传

教程仅作技术交流，最后支持使用正版软件。

## 参考

[//www.52pojie.cn/thread-688820-1-1.html" target="_blank" rel="noopener noreferrer nofollow"](https://www.52pojie.cn/thread-688820-1-1.html:href="https)

[//github.com/TsudaKageyu/minhook" target="_blank" rel="noopener noreferrer nofollow"](https://github.com/TsudaKageyu/minhook:rel="nofollow noopener" href="https)

[//blog.csdn.net/qq_42468226/article/details/128407995" target="_blank" rel="noopener noreferrer nofollow"](https://blog.csdn.net/qq_42468226/article/details/128407995:rel="nofollow noopener" href="https) (MinHook介绍|文档|例子)

