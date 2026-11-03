
第三个是直接找到了`SetCollisionEnabled`的地址然后构建函数。

```
function SetCollisionEnable(actor_addr,dr=3) {
  var f_addr=moduleBase.add(0x933b300);
  let CallFunc=new NativeFunction(f_addr,'void',['pointer','int']);
  CallFunc(ptr(actor_addr).add(0x130).readPointer(),dr);
}
```

第一个无法实现，第二个和第三个本质上是一样的，可以实现功能。

## Section3：黄色球体

这个类的方法直接猜了下和`getflag`这种字符串有关，直接在`sdkdump`中搜到了。但是正解我还需要再学习一下。

找到该函数后直接到目标地址。

![](https://attach.52pojie.cn/forum/202406/03/162241r4v4cl1xklkxzlx6.png)

`dlopen`是一个`UE4.so`中专门的函数。

调用了`libplay`中的`get_last_flag`函数。

再次进入后发现一堆异或。

![](https://attach.52pojie.cn/forum/202406/03/162257fqllfvtf8wqhcvtc.png)

这段异或尝试后发现是base64的码表。

打算通过动调理清这段加密流程。

在手机端启动`idaserver`。

然后以该命令以调试状态启动目标app

```
adb shell am start -D -n com.tencent.ace.match2024/com.epicgames.ue4.SplashActivity
```

可以Attach到进程，但是F9之后没有任何反应，手机显示`wait for debug`，电脑直接跑飞。

看来动调并不太行，于是转而去读汇编。

![](https://attach.52pojie.cn/forum/202406/03/162316q7vkvkygffff118z.png)

`E4C`这一段下面的`BR`会根据一些比较来决定跳转到下面的`EA8`还是`ED8`，这个让gpt就可以分析出来。

大致分析`EA8`是主要加密段，进行一个异或操作，下面的`ED8`类似`RET`，进行一个结束处理。(分析不一定准确，因为不会ARM汇编，只能结合AI)

再分析下面的函数，进入后发现`EEC`

![](https://attach.52pojie.cn/forum/202406/03/162333lahebdda9wtk6b69.png)

这个数组异或完后猜测是base64变表，那这个函数大抵就是base64了。

再往下的部分大概就是比较了。

![](https://attach.52pojie.cn/forum/202406/03/162353yt1cxwzjhjkxtmmh.png)

汇编喂给GPT后分析出大致是24个字节的比较。

这里就能推断出上文其中一个进行异或的数组是密文

异或后：

![](https://attach.52pojie.cn/forum/202406/03/162413u505rscfniffc85z.png)

![](https://attach.52pojie.cn/forum/202406/03/162435nnmttt5q97pnstwb.png)

这段数组应该就是作为`EA8`段异或操作的`key`数组。

根据这些写出脚本：

![](https://attach.52pojie.cn/forum/202406/03/162501v0afnb3qfofoaaf4.png)

```
#include
int main()
{
        char encode[] = { 0x55,0x4d,0x60,0x74,0x38,0x49,0x64,0x50,0x2c,0x7b,0x4f,0x03,0x68,0x36,0x1f,0x9f,0x8e,0x8a,0,0};
        char key[] = { 0x0A, 0x0C, 0x0E, 0x00, 0x51, 0x16, 0x27, 0x38, 0x49, 0x1A, 0x3B, 0x5C, 0x2D, 0x4E, 0x6F, 0xFA,
        0xFC, 0xFE};
        for (int i = 0; i 
```

拿到最后一部分flag：`_Anti_Cheat_Expert`

![](https://cdn.nlark.com/yuque/0/2024/png/39289695/1717319166251-f43373f7-cd92-4c40-854c-6573c76f7c80.png#averageHue=%23312f2f&clientId=u2cee4990-062f-4&from=paste&height=218&id=u9644edf2&originHeight=272&originWidth=843&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=66438&status=done&style=none&taskId=u033d5f43-5eaf-4586-8bd6-817dfe2ee60&title=&width=674.4)

## 最终结果

flag：`FLAG{8939008_Anti_Cheat_Expert}`

在复习的过程中还是学了不少东西的，也多亏了有两位师傅的无私解疑和指导，写的文章还很粗糙，需要时间打磨。

最后在这里贴上`frida`的脚本供于参考。

```
var GWorld_Offset=0x0B32D8A8

var GName_Offset=0x0B171CC0

var GUObjectArray_Offset=0xB1B5F98

var playerName="FirstPersonCharacter_C"

var moduleBase

var GWorld

var GName

var GUObjectArray

//Class: UObject

//对象的内部索引，用于唯一标识对象。

 var offset_UObject_InternalIndex = 0xC;

 //指向描述对象类的 UClass 对象

 var offset_UObject_ClassPrivate = 0x10;

 //对象名称在 FName 表中的索引

 var offset_UObject_FNameIndex = 0x18;

 //指向包含该对象的外部对象，表示层次关系。

 var offset_UObject_OuterPrivate = 0x20;

 var UObject={

  getClass: function(obj){

    var classPrivate=ptr(obj).add(offset_UObject_ClassPrivate).readPointer();//读取指针

    //console.log(`classPrivate: ${classPrivate}`);

    return classPrivate;

  },

getNameId: function(obj){

  //console.log(`obj:${obj}`);

  try{

    var nameId=ptr(obj).add(offset_UObject_FNameIndex).readU32();//读取4字节。

    //console.log(`nameId:${nameId}`);

    return nameId;

  }catch(e){

    console.log("error")

    return 0;

  }

},

getName: function(obj) {

  if (this.isValid(obj)){

      return getFNameFromID(this.getNameId(obj));

  } else {

      return "None";

  }

},

getClassName: function(obj) {

  if (this.isValid(obj)) {

      var classPrivate = this.getClass(obj);

      return this.getName(classPrivate);

  } else {

      return "None";

  }

},

isValid: function(obj) {

  var isValid = (ptr(obj) > 0 && this.getNameId(obj) > 0 && this.getClass(obj) > 0);

  // console.log(`isValid: ${isValid}`);

  return isValid;

}

 }

 function getFNameFromID(index) {

  // FNamePool相关偏移量和步长

  var FNameStride = 0x2;                   // FNameEntry 的步长，每个FNameEntry占用2字节

  var offset_GName_FNamePool = 0x30;       // GName 到 FNamePool 的偏移量

  var offset_FNamePool_Blocks = 0x10;      // FNamePool 到 Blocks 的偏移量

  // FNameEntry相关偏移量和位

  var offset_FNameEntry_Info = 0;          // FNameEntry 到 Info 的偏移量

  var FNameEntry_LenBit = 6;               // FNameEntry 长度位

  var offset_FNameEntry_String = 0x2;      // FNameEntry 到字符串部分的偏移量

  // 计算块和偏移量

  var Block = index >> 16;                 // 块索引

  var Offset = index & 65535;              // 块内偏移量

  // 获取FNamePool的起始地址

  var FNamePool = GName.add(offset_GName_FNamePool);

  // console.log(`FNamePool: ${FNamePool}`);

  // console.log(`Block: ${Block}`);

  // 获取特定块的地址

  var NamePoolChunk = FNamePool.add(offset_FNamePool_Blocks + Block * 8).readPointer();

  // console.log(`NamePoolChunk: ${NamePoolChunk}`);

  // 计算FNameEntry的地址

  var FNameEntry = NamePoolChunk.add(FNameStride * Offset);

  // console.log(`FNameEntry: ${FNameEntry}`);

  try {

      // 读取FNameEntry的Header

      if (offset_FNameEntry_Info !== 0) {

          var FNameEntryHeader = FNameEntry.add(offset_FNameEntry_Info).readU16();     

      } else {

          var FNameEntryHeader = FNameEntry.readU16();

      }

  } catch(e) {

      // 捕捉读取异常并返回空字符串

      // console.log(e);

      return "";

  }

  // console.log(`FNameEntryHeader: ${FNameEntryHeader}`);

  // 获取字符串地址

  var str_addr = FNameEntry.add(offset_FNameEntry_String);

  // console.log(`str_addr: ${str_addr}`);

  // 计算字符串长度和宽度

  var str_length = FNameEntryHeader >> FNameEntry_LenBit; // 计算字符串长度

  var wide = FNameEntryHeader & 1;                       // 判断字符串是否为宽字符

  // 如果是宽字符，返回 "widestr"

  if (wide) return "widestr";

  // 如果字符串长度合理，读取并返回UTF-8字符串

  if (str_length > 0 && str_length  {

  await GetActorVrtualAdress();

})();

//主要实现作弊的hanshu

function dumpActorInstances(){

  GWorld = moduleBase.add(GWorld_Offset).readPointer();//找到GWorld这个类并储存它的地址作为指针

  var Level_Offset = 0x30

  var Actors_Offset = 0x98

  var Level = GWorld.add(Level_Offset).readPointer()//关卡的指针

  var Actors = Level.add(Actors_Offset).readPointer()//Actor的指针

  var Actors_Num = Level.add(Actors_Offset).add(8).readU32()//Actors的数量

  var actorsInstances = {};

  for(var index =0; index 3000)//高度大于3000则执行

    {              

      try{

          //setActorHidden(actor_addr)

          //setActorCollisionEnabled(actor_addr,1)

          SetVisibility(actor_addr,1,0);

      }

      catch(e){}

    }

  }

}

set("libUE4.so")

setPlayerHP();

//GetActorVrtualAdress();

var b=getActorAddr(playerName);

//getActorLocation(b);

//getActorLocation(b);

dumpActorInstances();

//(-1569.0198974609375, -415.2847595214844, 268.3680725097656)

//setActorLocation(b,-1500,-400,3000);

  //下面这一句代码是指定要Hook的so文件名和要Hook的函数名，函数名就是上面IDA导出表中显示的那个函数
```

以及我学习中有看的一些文章

不错文章：[//www.yxfzedu.com/article/670](http://www.yxfzedu.com/article/670:http)

不错文章：[//iosre.com/t/%E8%AE%B0%E5%BD%95%E4%B8%80%E6%AC%A1%E8%99%9A%E5%B9%BB4ue4%E6%89%8B%E6%B8%B8%E9%80%86%E5%90%91/19808](https://iosre.com/t/%E8%AE%B0%E5%BD%95%E4%B8%80%E6%AC%A1%E8%99%9A%E5%B9%BB4ue4%E6%89%8B%E6%B8%B8%E9%80%86%E5%90%91/19808:https)

好文章：[//www.yxfzedu.com/article/10613](http://www.yxfzedu.com/article/10613:http)
