
# [Android CTF] 【2025春节】解题领红包之四 Android 中级题WriteUp

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2006138
- 抓取时间: 2026-11-01T16:07:24.957Z

## 【2025春节】解题领红包之四 Android 中级题WriteUp

## 0x00 初探虎先锋

题目题干如下：

出题老师：正己

题目简介：《黑神话·虎先锋の猴肉108种烹饪方式》通关失败后，张师傅怒摔手柄，突然瞥见室友王大爷的屏幕金光大作："无限定身术+金刚不坏+暴击999%！"

"年轻人，听说过科学修仙吗？"王大爷反手一个Alt+F4，显示器残留着神秘代码：

风灵月影宗弟子认证：输入[宗门秘钥]可解锁修改权限

（温馨提示：秘钥格式请参考——`flag{我是秘钥}`，"我是秘钥"的真实内容需要动态运算得出）

![](https://attach.52pojie.cn/forum/202502/14/024927scbotpci8hqru7xb.jpg)

第1步：直接进入战斗！然后败了😭

第2步：根据提示点击右上角风灵月影

第3步：输入密钥查看交互信息

第4步：`jadx` 查找字符串“密钥错误，请重试！”

![](https://attach.52pojie.cn/forum/202502/14/024929q3bzssb5fbtwwww3.jpg)

第5步：找到实现代码如下：

```
    /* renamed from: invoke */
    public final void m5invoke() {
        String BattleScreen$lambda$21;
        BattleScreen$lambda$21 = BattleActivityKt.BattleScreen$lambda$21(this.$activationCode$delegate);
        //只要Check函数返回True就能激活风灵月影
        if (!BattleActivityKt.Check(BattleScreen$lambda$21)) {
            Toast.makeText(this.$context, "秘钥错误，请重试！", 0).show();
            return;
        }
        BattleActivityKt.BattleScreen$lambda$7(this.$playerHp$delegate, BattleActivityKt.BattleScreen$lambda$3(this.$maxHp$delegate));
        BattleActivityKt.BattleScreen$lambda$10(this.$enemyHp$delegate, BattleActivityKt.BattleScreen$lambda$3(this.$maxHp$delegate));
        this.$battleResult$delegate.setValue("");
        this.$context.clearBattleLog();
        BattleActivityKt.BattleScreen$lambda$25(this.$playerAttackPower$delegate, 9999);
        this.$playerDefense.f5315i = 999;
        BattleActivityKt.updateLog(this.$context, "════════════════════");
        BattleActivityKt.updateLog(this.$context, "★ 风灵月影已激活 ★");
        BattleActivityKt.updateLog(this.$context, "➤ 攻击力提升至9999");
        BattleActivityKt.updateLog(this.$context, "➤ 防御力提升至999");
        BattleActivityKt.updateLog(this.$context, "➤ 生命值已重置");
        BattleActivityKt.updateLog(this.$context, "════════════════════");
        BattleActivityKt.BattleScreen$lambda$19(this.$showActivationDialog$delegate, false);
    }
}
```

第6步：算法助手直接hook这个`Check`函数，直接拿下虎先锋！

![](https://attach.52pojie.cn/forum/202502/14/024940nzvh5isv13zzxpzb.jpg)

第7步：提交`flag`！欸不对我的`flag`呢？

## 0x01 攻克`Check`函数

`jadx` 中查找 `Check` 的声明，发现其只有声明，调用了SO层实现。

```
//声明函数
public static final native boolean Check(String str);

//调用SO
static {
    System.loadLibrary("wuaipojie2025_game");
}
```

解压缩APK拿到`libwuaipojie2025_game.so`文件，直接上IDA。

分析了`arm64-v8a`、`X86`、`armeabi-v7a`感觉`armeabi-v7a`的难度较低，推荐`armeabi-v7a`。

## 0x02 IDA分析`libwuaipojie2025_game.so`

第1步：找到函数实现

在`export`中搜索`java`没有找到相关函数，初步判定为JNI动态注册。

看到有`JNI_OnLoad`函数从他入手

```
jint JNI_OnLoad(JavaVM *vm, void *reserved)
{
  jint v2; // r5
  int v3; // r0
  int v5; // [sp+0h] [bp-10h] BYREF
//此处v3、v5应该都是JNIEnv *类型；off_134D10应该就是g_methods函数
  if ( (*vm)->GetEnv(vm, (void **)&v5, 65542) )
    return -1;
  v3 = (*(int (__fastcall **)(int, const char *))(*(_DWORD *)v5 + 24))(
         v5,
         "com/zj/wuaipojie2025_game/ui/BattleActivityKt");
  v2 = -1;
  if ( v3 && (*(int (__fastcall **)(int, int, char **, int))(*(_DWORD *)v5 + 860))(v5, v3, off_134D10, 1) > -1 )
    return 65542;
  return v2;
}
```

根据JNI实现经验修改部分变量类型：

```
jint JNI_OnLoad(JavaVM *vm, void *reserved)
{
  jint v2; // r5
  JNIEnv *env; // r0
  JNIEnv *v5; // [sp+0h] [bp-10h] BYREF

  //此处v3、v5应该都是JNIEnv *类型；off_134D10应该就是g_methods函数
  if ( (*vm)->GetEnv(vm, (void **)&v5, 65542) )
    return -1;
  env = (JNIEnv *)(*v5)->FindClass(v5, "com/zj/wuaipojie2025_game/ui/BattleActivityKt");
  v2 = -1;
  if ( env && (*v5)->RegisterNatives(v5, env, (const JNINativeMethod *)g_methods, 1) > -1 )
    return 65542;
  return v2;
}
```

转到`g_methods`的汇编地址看到如下内容：

```
.data:00134D10 off_134D10      DCD aCheck              ; DATA XREF: LOAD:0000009C↑o
.data:00134D10                                         ; JNI_OnLoad+46↑o ...
.data:00134D10                                         ; "Check"
.data:00134D14                 DCD aLjavaLangStrin     ; "(Ljava/lang/String;)Z"
.data:00134D18                 DCD sub_BE440+1
.data:00134D1C unk_134D1C      DCB 0x73 ; s            ; DATA XREF: A(void)+20↑o
.data:00134D1C                                         ; A(void)+2E↑o ...
```

可知`sub_BE440`函数就是`Check`的实现位置，反编译结果如下：

```
bool __fastcall sub_BE440(int a1, int a2, int a3)
{
  int v5; // r4
  int v6; // r0
  int v7; // r9
  int v8; // r4
  int v9; // r6
  int v10; // r1
  unsigned int v11; // r6
  char *v12; // r4
  _BOOL4 v13; // r8
  int v14; // r0
  void (__fastcall *v15)(_BYTE *, int, int, void *); // r8
  void *v16; // r5
  int v17; // r4
  const std::nothrow_t *v18; // r1
  unsigned __int64 v20; // [sp+0h] [bp-58h]
  _BYTE v21[16]; // [sp+18h] [bp-40h] BYREF
  _QWORD v22[2]; // [sp+28h] [bp-30h] BYREF
  v5 = 0;
  v6 = (*(int (__fastcall **)(int, int, _DWORD))(*(_DWORD *)a1 + 676))(a1, a3, 0);
  if ( v6 )
  {
    v7 = v6;
    HIDWORD(v20) = a3;
    v8 = A();
    v9 = CNJAK();
    if ( !byte_134E49 )
    {
      afdm::decrypt_buffer((afdm *)byte_134D7E, &byte_4, 0xA8FC3415, v20);
      byte_134E49 = 1;
    }
    v10 = -1;
    if ( v8 )
      v10 = 1;
    v11 = v9 + v10;
    v12 = getenv(byte_134D7E);
    v13 = v12 == 0 || v11 > 24) & 1))));
    dword_134D90 = -559038669;
    memset(v21, 0, sizeof(v21));
    v16 = (void *)operator new[](0x13u);
    v15(v21, v7, 19, v16);
    v17 = memcmp(v16, &unk_3A0FC, 0x13u);
    operator delete[](v16, v18);
    (*(void (__fastcall **)(int, _DWORD, int))(*(_DWORD *)a1 + 680))(a1, HIDWORD(v20), v7);
    return v17 == 0;
  }
  return v5;
}
```

根据已知内容更改变量类型与变量名：

```
jboolean __fastcall check(JNIEnv *env, jobject obj, jstring jkey) //此处根据JNI实现和Check声明修改变量类型
{
  jboolean v5; // r4
  const char *key; // r0
  const char *key1; // r9
  int v8; // r4
  int v9; // r6
  int v10; // r1
  unsigned int v11; // r6
  char *v12; // r4
  _BOOL4 v13; // r8
  int v14; // r0
  void (__fastcall *fun_enc)(_BYTE *, const char *, int, void *); // r8
  void *v16; // r5
  int v17; // r4
  const std::nothrow_t *v18; // r1
  unsigned __int64 v20; // [sp+0h] [bp-58h]
  _BYTE v21[16]; // [sp+18h] [bp-40h] BYREF
  _QWORD v22[2]; // [sp+28h] [bp-30h] BYREF

  v5 = 0;
  key = (*env)->GetStringUTFChars(env, jkey, 0);
  if ( key )
  {
    key1 = key;
    HIDWORD(v20) = jkey;                        // 让v20变量的前8位赋值位jkey
    v8 = A();
    v9 = CNJAK();
    if ( !byte_CA839E49 )
    {
      afdm::decrypt_buffer((afdm *)byte_CA839D7E, (char *)4, 0xA8FC3415, v20);
      byte_CA839E49 = 1;
    }
    v10 = -1;
    if ( v8 )
      v10 = 1;
    v11 = v9 + v10;
    v12 = getenv(byte_CA839D7E);
    v13 = v12 == 0 || v11 > 24) & 1))));
    //fun_enc = (void (__fastcall *)(...)) nullsub_9(*(_DWORD *)( (unsigned int)v22 | (4 * ( ... )) ));
    //关键逻辑：表达式 (unsigned int)v22 | (4 * (...)) 用于计算 v22 数组的索引偏移
        //v22 是数组基址，4 * (...) 是索引偏移（QWORD 类型占8字节，但此处可能因对齐或混淆设计为4字节步进）
        //实际偏移由 (v14 | v13) ^ ... 的位运算结果决定，最终目标为 0（选a）或 1（选ao）

    dword_CA839D90 = -559038669;
    memset(v21, 0, sizeof(v21));        //把v21的值全部赋0
    v16 = (void *)operator new[](0x13u);        //调用加密函数，这里新申请了一段0x13u的内存v16应该是把结果返回到v16
    fun_enc(v21, key1, 19, v16);        //这里判断v16和&unk_CA73F0FC，共0x13（十进制19）位应该就是校验flag是否正确，如果两值前19位相等的话的话v17赋值0
    v17 = memcmp(v16, &unk_CA73F0FC, 0x13u);
    operator delete[](v16, v18);
    (*env)->ReleaseStringUTFChars(env, (jstring)HIDWORD(v20), key1);
    //当v17的值0时返回true
    return v17 == 0;
  }
  return v5;
}
```

### 反调试分析

```
//根据下面这个表达式返回的值确定fun_enc指向的函数地址

(unsigned int)v22 | (4 * (

    (v14 | v13) ^ 

    (sub_CA7C36CC & 1) ^ 

    (((ao ^ a) >> 24) & 1)

))

//返回只有两种可能v22[0]或v22[1]，其中v22[0] = *(_QWORD *)&off_C16B1CE8;（指向函数a的指针）v22[1] = *(_QWORD *)&off_C16B1CF0;（指向函数ao的指针）

//下面这个表达式返回的就决定了是v22指向的是[0]还是[1]

    (v14 | v13) ^         //其中v13、v14在上面进行了赋值；v13=（getenv()==0）,v14=jgbjkb()

    (sub_CA7C36CC & 1) ^         //sub_CA7C36CC只return 0，所以&1结果为0

    (((ao ^ a) >> 24) & 1)        //ao ^ a默认没改的话为0，“>>24”取高8位也为0，所以&1结果为0
```

所有这里的反调试关键点在于上面需要满足以下两个条件：

- `getenv()!=0`
