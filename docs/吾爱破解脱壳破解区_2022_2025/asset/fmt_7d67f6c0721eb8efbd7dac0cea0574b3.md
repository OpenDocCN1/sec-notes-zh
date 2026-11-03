
```
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        TO TOI = new TO();
        System.out.println(TOI.db("cYoiUd2BfEDc/V9e4LdciBz9Mzwzs3yr0kgrLA=="));
        System.out.println(TOI.db("2hyWtSLN69+QWLHQ"));
        System.out.println(TOI.db("hjyaQ8jNSdp+mZic7Kdtyw=="));
    }
}
```

![](https://attach.52pojie.cn/forum/202501/31/163219ydpwudgbfpp1ryuk.png)

image.png (5.63 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1MjQ1M3xmZTc0NDM5OHwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjQ1M3xmZTc0NDM5OHwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes)

2025-1-31 16:32 上传

> 

Flag: flag{xnkl2025!}

# 解题领红包之四 {Android中级题}

《黑神话·虎先锋の猴肉108种烹饪方式》通关失败后，张师傅怒摔手柄，突然瞥见室友王大爷的屏幕金光大作："无限定身术+金刚不坏+暴击999%！"

"年轻人，听说过科学修仙吗？"王大爷反手一个`Alt+F4`，显示器残留着神秘代码：

风灵月影宗弟子认证：输入[宗门秘钥]可解锁修改权限

（温馨提示：秘钥格式请参考——`flag{我是秘钥}`，"我是秘钥"的真实内容需要动态运算得出）

首先运行看看：

![](https://attach.52pojie.cn/forum/202501/29/164035i733b91qtvqtbi89.png)

image.png (53.16 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1MjI4NnwxZDNjMzE0NHwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjI4NnwxZDNjMzE0NHwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes)

2025-1-29 16:40 上传

> 

好家伙.jpg

试玩一下：

![](https://attach.52pojie.cn/forum/202501/29/164140syffwy01fvvissvw.png)

image.png (42.26 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1MjI4N3wzMjc2NmY0YnwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjI4N3wzMjc2NmY0YnwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes)

2025-1-29 16:41 上传

"造成1点伤害......"，诶右上角的是......

(一阵强劲的音乐响起

![](https://attach.52pojie.cn/forum/202501/29/164319nbbgiebq5inesveh.png)

image.png (29.31 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1MjI4OHxiYmRiODMzM3wxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjI4OHxiYmRiODMzM3wxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes)

2025-1-29 16:43 上传

这挂是假的吧......

## jadx静态分析

通过`manifest.xml`，可以看出这个app分为两个Activity: `com.zj.wuaipojie2025_game.MainActivity`以及`com.zj.wuaipojie2025_game.ui.BattleActivity`

分别对应主界面和战斗界面。

进入战斗界面，找到一个类`BattleActivityKt`

![](https://attach.52pojie.cn/forum/202501/29/165012ual9f6j7zaavr9zl.png)

image.png (23.94 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1MjI4OXw3NmEzNmI5OXwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjI4OXw3NmEzNmI5OXwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes)

2025-1-29 16:50 上传

以及它的子类`BattleActivityKt$BattleScreen$1$3$1$1$2`

![](https://attach.52pojie.cn/forum/202501/29/165317aoko9bbbdr5d9q94.png)

image.png (34.38 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1MjI5MHxiZjMxYTZhZnwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjI5MHxiZjMxYTZhZnwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes)

2025-1-29 16:53 上传

这里有个关键的函数`if (!BattleActivityKt.Check(BattleScreen$lambda$21))`

也就是说如果`Check`函数返回的是true，那么就会启动修改模式。

那么，关键就在于这个`Check`函数了，我们点进去......是空的？！

![](https://attach.52pojie.cn/forum/202501/29/165857eq22wpm3x9pxm9ne.png)

image.png (2.11 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1MjI5M3xkYzhiNTkxZnwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjI5M3xkYzhiNTkxZnwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes)

2025-1-29 16:58 上传

问问GPT:

> 

- Native Method Declaration

The Check function is declared as a native method in the Java or Kotlin code. This indicates that the implementation of the method is provided in a native library (e.g., a .so file on Android). The Smali code only contains the declaration of the method, not its implementation.

- Dynamic Loading

The native code for the Check function is likely loaded dynamically at runtime. This is a common practice for security-related functions, performance-critical code, or code that needs to be hidden from decompilation. The native library containing the implementation of Check is usually loaded using `System.loadLibrary` in the application's Application class or in a specific activity.

简而言之，这个函数仅进行了声明，而实现代码在`so`文件中。

【当然用动态加载`dex`也行，参考2024年的中级题】

搜索代码，也找到了加载`so`文件的函数：

```
static {

        System.loadLibrary("wuaipojie2025_game");

    }
```

确定这是`so`(Java Native Interface, `JNI`)层的函数。

`jadx`不能调试也不能反编译`so`文件，所以得使用其他软件；这里用到了`IDA`。

## IDA静态分析

先来给没有接触过`so`逆向的朋友们介绍一下执行流程，这里借用一下[//blog.csdn.net/LittleFishC/article/details/141562885" target="_blank" rel="noopener noreferrer nofollow"](https://blog.csdn.net/LittleFishC/article/details/141562885:rel="nofollow noopener" href="https)中的部分内容：

> 

`System.loadLibrary`的核心功能是获取`.so`库的全路径，加载该`.so`库文件，然后依次调用`init(DT_INIT` 区块)、`init_array(DT_INIT_ARRAY` 区块，函数指针)和`JNI_OnLoad`函数。

查找`init`:

![](https://attach.52pojie.cn/forum/202501/30/190346chljwi44am4hnh66.png)

image.png (6.82 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1MjM4MnxlOWJkZDlmOHwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjM4MnxlOWJkZDlmOHwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes)

2025-1-30 19:03 上传

`crtls` 跳`PE`区段，查找`init_array`:

![](https://attach.52pojie.cn/forum/202501/30/190513gc3kkyt0vvnyaz82.png)

image.png (15.04 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1MjM4NXwxZjA1MDViMHwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjM4NXwxZjA1MDViMHwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes)

2025-1-30 19:05 上传

都没有，那就直接来看`JNI_Onload`函数吧（以下所有反编译代码均对数据类型和函数名称有所调整，看起来清清楚楚的）。

```
int __cdecl JNI_OnLoad(JavaVM *a1)
{
  int v1; // esi
  jclass v2; // eax
  JNIEnv *v4[4]; // [esp+Ch] [ebp-10h] BYREF

  v4[1] = (JNIEnv *)__readgsdword(0x14u);       // Read memory from a location specified by an offset relative to the beginning of the GS segment.
  v1 = -1;
  if ( !(*a1)->GetEnv(a1, (void **)v4, 65542) )
  {
    v2 = (*v4[0])->FindClass(v4[0], "com/zj/wuaipojie2025_game/ui/BattleActivityKt");// 定位类"com/zj/wuaipojie2025_game/ui/BattleActivityKt"
    if ( v2 )
      return ((*v4[0])->RegisterNatives(v4[0], v2, (const JNINativeMethod *)strCheck, 1) >> 31) | 0x10006;// 注册Native方法"check"，返回方法JNI版本
  }
  return v1;
}
```

嗯，这个函数里面定义了前面说到的`Check`方法，双击`strCheck`，可以看到一个`JNINativeMethod`结构体

```
.data:0015D1B8 strCheck        dd offset aCheck        ; DATA XREF: LOAD:0000009C↑o
.data:0015D1B8                                         ; JNI_OnLoad+60↑o
.data:0015D1B8                                         ; "Check"
.data:0015D1BC                 dd offset aLjavaLangStrin ; "(Ljava/lang/String;)Z"
.data:0015D1C0                 dd offset sub_E4340
```

根据结构体定义：

```
typedef struct {
    const char* name;       // 方法名
    const char* signature;  // 方法签名
    void* fnPtr;            // 对应的本地函数指针
} JNINativeMethod;
```

对应一下可知：方法名为`Check`，方法签名为`(Ljava/lang/String;)Z`【翻译过来输入类型是`string`，返回类型为`bool`，对应关系参考[//www.52pojie.cn/thread-1701353-1-1.html#44383431_%E4%B8%89%E3%80%81%E8%AF%BE%E7%A8%8B%E5%86%85%E5%AE%B9" target="_blank" rel="noopener noreferrer nofollow"](https://www.52pojie.cn/thread-1701353-1-1.html#44383431_%E4%B8%89%E3%80%81%E8%AF%BE%E7%A8%8B%E5%86%85%E5%AE%B9:href="https)】，本地函数指针为`sub_E4340`

打开对应函数：

```
bool __cdecl CheckImplement(JNIEnv *a1, jclass a2, char *a3)

{

  const char *v3; // esi

  char *v4; // edi

  unsigned __int8 v6; // [esp+Fh] [ebp-4Dh]

  __m128i v7; // [esp+10h] [ebp-4Ch] BYREF

  __m128i v8; // [esp+20h] [ebp-3Ch] BYREF

  __int128 v9; // [esp+30h] [ebp-2Ch] BYREF

  __int64 v10; // [esp+40h] [ebp-1Ch]

  unsigned int v11; // [esp+48h] [ebp-14h]

  v11 = __readgsdword(0x14u);

  v3 = (*a1)->GetStringUTFChars(a1, a3, 0);

  A();

  v7.m128i_i8[0] = CNJAK();

  v6 = v7.m128i_i8[0] & jgbjkb() ^ (*v3 == 72);

  v10 = *(_QWORD *)off_158184;

  v9 = 0LL;
```
