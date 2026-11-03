
# [Android CTF] 2025游戏安全技术竞赛-Android-初赛WriteUp

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2024788
- 抓取时间: 2026-11-01T15:52:52.636Z

## 附件说明

| 文件名称 | 功能 |
|---|---|
| `decode.py` | 处理字符串解密 |
| `hack.js` | 实现修复功能的Frida脚本 |
| `SDK.txt` | Dump得到的游戏SDK |
| `Actor.txt` | Dump得到的Actorlist |

`hack.js`的用法为：终端输入

```bash
frida -U -f com.ACE2025.Game -l hack.js
```

等待游戏启动之后继续在终端输入：`my_hack()`调用`my_hack`函数

## 获取UE4三件套

用`frida-ue4dumper`得到`base`: `0x764bd2f000`, `GUObjectArray`: `undefined`, `GName`: `0x7656b1f7c0`

得到`GName`偏移：`0xADF07C0`

`GWorld`: `0xAFAC398` (通过字符串`SeamlessTravel FlushLevelStreaming`定位)

`GUObject`: `0xAE34A98` (通过字符串`Max UObject count is invalid. It must be a number that is greater than 0.`定位)

`dumpSDK`

```bash
./ue4dumper64 --package com.ACE2025.Game --ptrdec --sdku --gname 0xADF07C0 --guobj 0xAE34A98 --output /data/local/tmp --newue+
```

`dumpActors`

```bash
./ue4dumper64 --package com.ACE2025.Game --ptrdec --actors --gname 0xADF07C0 --gworld 0xAFAC398 --output /data/local/tmp/actors.txt --newue+
```

`dump libUE4.so`

```
[INFO] RootService: Service connected

==========================
Process : com.ACE2025.Game
PID: 15801
FILE: libUE4.so
Start Address: 764bd2f000
End Address: 7656a3d000
Size Memory: 173MB (181460992)
[INFO] Fixing...
[INFO] Fixer output :
Rebuilding Elf(So)
warning load size [185078944] is bigger than so size [181460992], dump maybe incomplete!!!
warning DT_HASH not found,try to detect dynsym size...
fixed so has write to /sdcard/PADumper/com.ACE2025.Game/764bd2f000-7656a3d000-libUE4_fix.so
Rebuilding Complete
Output: /sdcard/PADumper/com.ACE2025.Game
[INFO] Dump Success
==========================
```

## 分析外挂加载逻辑

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034574.png)

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034575.png)

直接删掉`libGame.so`再启动游戏会crash，查看log可以定位到`libGame.so`是在`libUE4.so`加载的时候加载的，并且发现`libUE4.so`的依赖中包含了`libGame.so`，可以确认是ELF感染注入

通过`readelf`可以看到`libGame.so`是`libUE4.so`的依赖，具体的实现逻辑应该是通过修改`DT_NEEDED`标签来加载这个so.

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034576.png)

检查`PHT`数组中`p_tag`为`PT_DYNAMIC`的元素，并找到其中`d_tag`为`DT_STRTAB`的元素，其值就是字符串表在文件中的偏移，`d_tag`为`DT_STRSZ`的元素的值是字符串表的长度，将两者相加即为字符串表末尾的地址，发现字符串表中有`libGame.so`，并且`DT_NEEDED`条目中包含了`libGame.so`，当linker加载`libUE4.so`时，会解析`libUE4.so`的dynamic段，并遍历`DT_NEEDED`条目，生成新的`LoadTask`，递归加载所有依赖库

## 外挂逻辑实现

`libGame.so`的函数都被`CFF`混淆过了，用`D810`插件可以有比较不错的去混淆效果，起码算是能看了

`libGame.so`的`init_array`段中会调用一个函数，追踪这个函数的调用链可以发现`sub_27F0`会创建一个线程，那么这个线程执行的函数就是外挂逻辑的实现了

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034577.png)

## 字符串解密

`0x1B9C`函数是主要执行外挂逻辑的函数

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034578.png)

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034579.png)

在`libGame.so`里面还有几个字符串解密函数，由于数量不是很多所以我并没有写一个一次性解密所有字符串的函数，如果需要解密字符串可以通过特征匹配的思路来解决，字符串解密函数的第一个参数存放解密后的字符串，第二个参数是密文，这种字符串解密函数的特征就是前n个字节做密钥，后面的则是密文，字符串的长度就是if条件中的值，如果使用特征匹配的话思路就是匹配^,%, if == n这样的式子来获取字符串长度和密钥长度，然后查找字符串函数的交叉引用，提取参数中的地址，并用IDA的api读取指定地址的数据，然后复现解密函数进行批量解密

我这里为了不复杂化就写一个解密函数来手动修改长度和数据

```python
data = [0xA5, 0x05, 0x5E, 0x29, 0xE2, 0x9A, 0xBB, 0x6E, 0x08, 0x42, 0xC3, 0x55, 0xEC, 0x01, 0x7C, 0xA9, 0x96, 0xD3, 0x59, 0xA8, 0x91, 0xCF, 0x89, 0x11, 0x24, 0xD6, 0xC9, 0x6C, 0x3C, 0x7C, 0xA7, 0xAE, 0x95, 0x1D, 0x67, 0x42]
count = 0
for i in range(len(data)):
    print(chr(data[i + 0x1A] ^ data[i % 0x1A]), end='')
    count += 1
    if count == 0x9:
        break
```

于是可以得到字符串信息

| 地址 | 字符串 |
|---|---|
| `0xB658` | `libUE4.so` |
| `0xB650` | `-` |
| `0xB648` | `r` |
| `0xB634` | `/proc/%d/maps` |
| `0xB620` | `/proc/self/maps` |

### sub_B80(获取libUE4基址)

`sub_B80`函数是寻找`libUE4.so`的基址，具体实现流程如下:通过`fopen`函数打开`/proc/pid/maps`文件打开程序的虚拟内存空间，通过`fgets`函数遍历`maps`文件的每一行字符串，通过`strstr`函数筛选包含`libUE4.so`字符串的行，然后通过`strtok`函数以字符`-`分割文本，提取字符串，最后使用`strtoul`函数讲字符串转换成`unsigned long int`类型的整数，通过`Frida` hook可以很容易分析出来这一函数

```javascript
function hook_strstr() {
    const targetModule = "libGame.so";
    const strstrPtr = Module.getExportByName(null, "strstr");
    Interceptor.attach(strstrPtr, {
        onEnter: function(args) {
            const returnAddress = this.returnAddress;
            const module = Process.findModuleByAddress(returnAddress);
            if (!module || module.name !== targetModule) return;
            const mainStr = Memory.readCString(args[0]);
            const subStr = Memory.readCString(args[1]);
            console.log(`[${targetModule}] strstr("${mainStr}", "${subStr}")`);
        }
    });
}

const waitForGameLib = setInterval(() => {
    if (Module.findBaseAddress("libGame.so")) {
        clearInterval(waitForGameLib);
        hook_strstr();
    }
}, 100);
```

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034580.png)

### 获取GWorld

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034581.png)

发现这里和之前找到的`GWorld`偏移一致，可以知道这里获取了`GWorld`的地址

### 获取PersistentLevel

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034582.png)

这里获取了持久关卡的指针，这是`UWorld`类中的一个重要成员，存放了`Actors`等数据，是遍历玩家角色的关键成员变量

### 获取Actors和Actors的数量

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034583.png)

这里获取了`Actor`的数组和长度，用于下面遍历`Actor`获取目标对象

### 遍历Actors

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034584.png)

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034585.png)

此处遍历`Actor`并获取指定对象，这里的`0xA63BE28`是对象的虚标指针，`UE4`中每个类的虚表（`vtable`）在`libUE4.so`中的偏移是固定的。通过比对虚表偏移，判断当前 `Actor` 是否为目标对象

```javascript
function my_hack(){
    getlibue4();
    var actorsAddr = getActorsAddr();
    var base = Module.getBaseAddress("libUE4.so");
    for (var key in actorsAddr) {
        if (actorsAddr[key].readPointer() - base == 0xA63BE28) {
            console.log("[+] Successful Get target Actor : ", key);
        }
    }
}
```

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034586.png)

通过`frida`脚本可以发现程序寻找的`Actor`为: `FirstPersonCharacter_C`

### 修改移速和后坐力

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034587.png)

这里可以看到外挂寻找了几个当前对象的偏移

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034588.png)

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034589.png)

![](https://jeremiah.oss-cn-shenzhen.aliyuncs.com/picture/202504160034590.png)

可以在`SDK`找到这几个偏移对应的变量

| 偏移 | 类 | 对象 |
|---|---|---|
| `0x538` | `MyProjectCharacter.Character.Pawn.Actor.Object` | `RecoilAccumulationRate` |
| `0x288` | `Character.Pawn.Actor.Object` | `CharacterMovementComponent` |
| `0x1A0` | `CharacterMovementComponent.PawnMovementComponent....` | `MaxAcceleration` |

```javascript
function my_hack(){
    getlibue4();
    var actorsAddr = getActorsAddr();
    var base = Module.getBaseAddress("libUE4.so");
    for (var key in actorsAddr) {
        if (actorsAddr[key].readPointer() - base == 0xA63BE28) {
            console.log("[+] Successful Get target Actor : ", key);
            var player_addr = actorsAddr[key];
            var RecoilAccumulationRate = player_addr.add(0x538);
            var CharacterMovementComponent = player_addr.add(0x288).readPointer();
            var MaxAcceleration = CharacterMovementComponent.add(0x1A0);
            Memory.writeFloat(MaxAcceleration, 1000);
            Memory.writeFloat(RecoilAccumulationRate, 5);
        }
    }
}
```

通过修改对应的内存值可以发现`RecoilAccumulationRate`实际上是计算枪口抖动的系数，外挂中把此处修改成了0，改成非0的值就可以发现开枪之后枪口会上调

`MaxAcceleration` 是人物移动速度的变量，程序把这里的值修改成了1000000000，把值改成1000则可以使人物正常行走

### 自瞄

仔细观察游戏会发现，每次开枪的时候准星都会锁定在一个箱子那里，无论是重启游戏还是移动角色或者箱子到不同位置，准星始终在一个箱子身上，于是先寻找出来是哪一个箱子，在之前dump的Actor列表中发现有名为`EditorCube`的对象，一共有14个，数了一下箱子的数量也是14个，那么基本可以确定箱子就是这个对象，然后通过输出所有箱子的`vector`的值获取坐标，再移动目标箱子就可以比对出来是哪一个箱子
