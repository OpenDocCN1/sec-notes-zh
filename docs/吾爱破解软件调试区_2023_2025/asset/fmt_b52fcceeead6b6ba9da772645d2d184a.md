
# [游戏安全] 如何制作 Unity 游戏的 CE-table（以 CYANBRAIN 为例）

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2061705
- 抓取时间: 2026-11-01T21:51:23

CYANBRAIN 太难了，然后 CE 搜出来的指针经常性不能用，所以想自己做一个。

## 前言

本文介绍如何为 Unity 游戏 CYANBRAIN 制作 Cheat Engine (CE) table，实现数据修改和调试。

主要难点在于基址的查找。

- 使用 `Pointer scan for this address` 或 `Find out what writes to this address`
  - 扫的结果过多
  - 结果不一定准确
  - 速度过慢
- 部分通过 `Assembly-CSharp.dll`+定位也不准确
- 更便捷的方式
  - 打开 `Mono`
  - `Mono -> .NetInfo`
  - `.NetInfo -> Assembly-CSharp -> 搜索 player/Inventory/gameManager`
  - 找到 `player` 指针，通过指针修改
  - 必要的时候可以用

先说 `Assembly-CSharp.dll`。它就是游戏的脚本，游戏的整个运行逻辑和效果都在这里头。我们要修改 Mono 游戏，总是在和这个文件打交道。

## 先看下 .Net 代码

一般搜 `player`/`Inventory`/`gameManager` 等等的类，找找看里面有什么东西，CYANBRAIN 的相关代码在 `Player` 这个类里面。

[imgse.com/i/pV4RTRx](https://imgse.com/i/pV4RTRx)

### 我们先看最简单的血量

`.NetInfo->Assembly-CSharp->Player->methods->get_HP`

[imgse.com/i/pV4RXee](https://imgse.com/i/pV4RXee)

#### 汇编硬分析

双击 `get_HP` 进入汇编。

[imgse.com/i/pV4RjdH](https://imgse.com/i/pV4RjdH)

```
Player:get_HP - 48 83 EC 08           - sub rsp,08
Player:get_HP+4- 48 89 34 24           - mov [rsp],rsi
Player:get_HP+8- 48 8B F1              - mov rsi,rcx
Player:get_HP+b- 48 8B 46 68           - mov rax,[rsi+68]
Player:get_HP+f- F3 0F10 40 10         - movss xmm0,[rax+10]
Player:get_HP+14- F3 0F5A C0            - cvtss2sd xmm0,xmm0
Player:get_HP+18- F2 0F5A C0            - cvtsd2ss xmm0,xmm0
Player:get_HP+1c- 48 8B 34 24           - mov rsi,[rsp]
Player:get_HP+20- 48 83 C4 08           - add rsp,08
Player:get_HP+24- C3                    - ret
```

- 可以直接扔给 AI，他会告诉 `HP` 相对于 `Player` 的偏移是多少。
- 可以自己分析。因为是 `Player:get_HP`，所以 `rsi` 就是 `Player` 的指针。
- `Player:get_HP+b` 偏移 `68`，`Player:get_HP+f` 再偏移 `10`。
- `Player -> +68 -> +10`

[imgse.com/i/pV4Rvod](https://imgse.com/i/pV4Rvod)

#### 使用 dnspy 分析

`Assembly-CSharp.dll` 位于 `*\CYANBRAIN_Data\Managed\Assembly-CSharp.dll`。

`Player` -> 搜索 `hp`。

[imgse.com/i/pV4RzFA](https://imgse.com/i/pV4RzFA)

```
    // Token: 0x17000058 RID: 88
    // (get) Token: 0x06000652 RID: 1618 RVA: 0x00018AF4 File Offset: 0x00016CF4
    public float HP
    {
        get
        {
            return this.status.hp;
        }
    }
```

`Player -> status -> hp`

回到 CE 中，`.NetInfo->Assembly-CSharp->Player->fields->status`。

[imgse.com/i/pV4WpWt](https://imgse.com/i/pV4WpWt)

可以看到是 `Player.status = Player + 68`。

[imgse.com/i/pV4WCSP](https://imgse.com/i/pV4WCSP)

可以看到是 `Player.status.hp = Player.status + 10 = Player + 68 + 10`。

和汇编分析出来的一致。

## 再看下武器相关的

### 汇编硬分析

和上面一样，找到 `.NetInfo->Assembly-CSharp->Player->methods->EquipGun` 或者 `.NetInfo->Assembly-CSharp->Player+Inventory->methods->CostBullet`。

全部汇编扔给 AI 或者自己分析，都可以分析出有个 `Player+128` 的指针（或者自己硬找到 `field` 中的 `currentWeapon` 也可以）。

在 `field` 中看一下：

[imgse.com/i/pV4WeFs](https://imgse.com/i/pV4WeFs)

有一个 `currentWeapon` 是 `WeaponHandle` 的类。

`needPullBolt`（换弹夹）、`clipBulletCount`（当前子弹）、`gunSO` 就是枪的具体东西了。

![](https://img.parap.dpdns.org/ce-cyanbrain-img/weponso.png)

里面有 `shootCD`（射速）、`damage`（伤害）、`gunType`（哪个枪）等等。

### 以修改 `gunType` 为例

（偏移在 `field` 的 `offset` 字段）：

```
Player.currentWeapon= Player + 128 +
Player.currentWeapon.gunSO= Player + 128 + 10
Player.currentWeapon.gunSO.gunType= Player + 128 + 10 +5c
```

![](https://img.parap.dpdns.org/ce-cyanbrain-img/guntype.png)

### 使用 dnspy 分析

![](https://img.parap.dpdns.org/ce-cyanbrain-img/WeaponHandle.png)

也是一样的，更简单一点，还有对应各种常量如 `GunType`。

```
public enum GunType
    {
        // Token: 0x040008E6 RID: 2278
        Pistol,
        // Token: 0x040008E7 RID: 2279
        MP01,
        // Token: 0x040008E8 RID: 2280
        MP104,
        // Token: 0x040008E9 RID: 2281
        M83,
        // Token: 0x040008EA RID: 2282
        STG108,
        // Token: 0x040008EB RID: 2283
        SG99,
        // Token: 0x040008EC RID: 2284
        SG100,
        // Token: 0x040008ED RID: 2285
        MK11,
        // Token: 0x040008EE RID: 2286
        MG106,
        // Token: 0x040008EF RID: 2287
        Kar100k,
        // Token: 0x040008F0 RID: 2288
        GM1,
        // Token: 0x040008F1 RID: 2289
        LFG,
        // Token: 0x040008F2 RID: 2290
        X
    }
}
```

## `Player` 的指针

那么最关键的来了，如何找 `Player` 的指针呢？CE 是有 `Mono` 功能的，我们可以直接 hook DLL 中函数。

hook `Player` 的任意函数 -> 修改里面的内容（因为是 `Player` 的函数，所以可以通过 `this` 获取到 `Player` 的指针）-> 获取到指针。

### hook 函数选择问题

因为 `Player` 的指针是经常变的，比如死亡/回档/重启等等操作都会可能导致 `Player` 的指针的变化，这就是为什么 CE 搜到的值不能用了。

- 因为 Unity 有 `Update` 的函数，即每一帧都会执行一遍，可以 hook 这个函数。
- 也可以 hook 其他的函数，如键盘输入时 `input`，或者其他任意 `Player` 的指针的变化后执行了的函数。
- 更新的越频繁越消耗系统资源（其实也没消耗多少），但是也要保证死亡/回档时能及时更新。
- 这里 hook 的是 `UpdatePlayerState`。

### 如何 hook 函数

以 `UpdatePlayerState` 为例，`this` 一般作为第一个参数传进来，位于 `rcx` 中，不放心扔给 AI 让他分析一下。

![](https://img.parap.dpdns.org/ce-cyanbrain-img/UpdatePlayerState.png)

我们可以看到：

```
Player:UpdatePlayerState+10 - 48 8B F1              - mov rsi,rcx
```

这里其实就是函数中调用了 `this`。

```
private void UpdatePlayerState()
{
    if (!this.actionFlag.Has(ActionFlag.Roll) && !this.actionFlag.Has(ActionFlag.Combat))
    {
        this.status.energy += Time.deltaTime / this.playerSetting.energyChargeTime;
        this.status.energy = Mathf.Min(this.status.energy, 1f);
    }
    this.status.hunger -= Time.deltaTime / this.status.hungerDuration;
    this.status.hunger = Mathf.Max(0f, this.status.hunger);
    float num;
    if (this.status.medicineCount g__MedicineSpawnTime|104_0(out num))
    {
        this.status.medicineProgress += Time.deltaTime / num;
        if (this.status.medicineProgress >= 1f)
        {
            this.status.medicineProgress -= 1f;
            this.status.medicineCount++;
        }
    }
    this.playerUI.UpdatePlayerInfoPanel(this.status.energy, this.status.medicineProgress, -1f, this.status.hunger);
    this.playerUI.UpdateWeaponPanelSingle(-1, -2, this.status.medicineCount);
}
```

我们知道 `rcx` 是传参第一个参数的 `pointer=this=Player` 的指针。

我们可以在任意地方插入 `mov [PlayerPtr],rcx` // 保存 `Player` 指针。

### 如何创建脚本

我打算在 `Player:UpdatePlayerState+10 - 48 8B F1 - mov rsi,rcx` 下面插入 `mov [PlayerPtr],rcx`，这样就可以获取到 `Player` 的指针。

也不一定非要在这里，在开头第一句也可以，在最后也可以，只要不影响原来的逻辑都是可以的。

以 `Player:UpdatePlayerState+10` 为例：

在 CE 的 `Memory Viewer` 窗口 -> 点击 `Player:UpdatePlayerState+10` 那一行 -> `Tools` -> `Auto Assemble` -> `Template` -> `full injection`。

CE 会自动生成一些代码，把注释去掉之后就是这样：

```
define(address,292A75BC110)
define(bytes,48 8B F1 66 0F 57 C0)

[ENABLE]
assert(address,bytes)
alloc(newmem,$1000,292A75BC110)
label(code)
label(return)

newmem:
code:
  mov rsi,rcx
  xorpd xmm0,xmm0
  jmp return

address:
  jmp newmem
  nop 2

return:

[DISABLE]
address:
  db bytes
  // mov rsi,rcx
  // xorpd xmm0,xmm0
dealloc(newmem)
```

### 模板脚本解释

- `define(address,Player:UpdatePlayerState+10)`：修改 `292A75BC110` 为 `Player:UpdatePlayerState+10`，因为 `292A75BC110` 每次启动游戏会变化。
- `define(bytes,48 8B F1 66 0F 57 C0)`：`48 8B F1` 对应 `Player:UpdatePlayerState+10 - 48 8B F1 - mov rsi,rcx`；`66 0F 57 C0` 对应 `Player:UpdatePlayerState+13 - 66 0F 57 C0 - xorpd xmm0,xmm0`。
- 这里为什么要 hook 2 行：`Player:UpdatePlayerState+10` 才 3 字节，`jmp newmem` 就是 5 个字节，一行不行。2 行 7 字节还有 2 个字节用 `nop` 填充。

```
address:
jmp newmem
nop 2
```

- `assert(address,bytes)`：把 `address` 和 `bytes` 做校验，如果不一样就报错，为的是防止游戏版本变化后脚本不能用。
- 分配指针并添加全局变量供 CE 使用，把下面 4 句插入到对应位置：

```
alloc(PlayerPtr,8)
dealloc(PlayerPtr)
registersymbol(PlayerPtr)
unregistersymbol(PlayerPtr)
```

- `code` 中是原始代码，可以删除。

最后结果
