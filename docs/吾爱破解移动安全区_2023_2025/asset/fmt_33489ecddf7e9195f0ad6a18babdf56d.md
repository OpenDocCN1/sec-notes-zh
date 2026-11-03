
![](https://attach.52pojie.cn/forum/202504/14/094909l992029g229d6842.png)

![](https://attach.52pojie.cn/forum/202504/14/094909feccxzaehhtextnd.png)

但`VR_MuzzleLocation`看名字來說是給VR設備使用的，安卓設備感覺是使用`FP_MuzzleLocation`才對。

因此合理懷疑這也是一個異常點。

```
Class: MyProjectCharacter.Character.Pawn.Actor.Object
    SkeletalMeshComponent* Mesh1P;//[Offset: 0x4b8, Size: 0x8]
    SkeletalMeshComponent* FP_Gun;//[Offset: 0x4c0, Size: 0x8]
    SceneComponent* FP_MuzzleLocation;//[Offset: 0x4c8, Size: 0x8]  // 槍口位置
    SkeletalMeshComponent* VR_Gun;//[Offset: 0x4d0, Size: 0x8]
    SceneComponent* VR_MuzzleLocation;//[Offset: 0x4d8, Size: 0x8]
```

`hook` `process_before_shoot`，從IDA裡可知`args[0].add(0x4D8)`保存著`VR_MuzzleLocation`對象的地址，嘗試將它改為`FP_MuzzleLocation`的地址。

```
function hook_test() {

    Interceptor.attach(base.add(0x670F110), {

        onEnter: function(args) {

            console.log("[process_before_shoot]");

            let VR_MuzzleLocation = args[0].add(0x4D8)

            VR_MuzzleLocation.writePointer(All_Objects["MuzzleLocation"])

            printName(VR_MuzzleLocation.readPointer());

        },

        onLeave: function(retval) {

        }

    })

}
```

結果是子彈終於會隨著槍口的變化而變化，但卻是從槍口往左發射的，正常應該是往前才對。

接下來嘗試修改`FP_MuzzleLocation`裡的一些參數，看看能否改變發射方向。

`FP_MuzzleLocation`屬於`USceneComponent`類，其中有以下這兩個屬性：

```
Class: SceneComponent.ActorComponent.Object
    //...
    Vector RelativeLocation;//[Offset: 0x11c, Size: 0xc]
    Rotator RelativeRotation;//[Offset: 0x128, Size: 0xc]
```

利用`K2_SetRelativeLocation`和`K2_SetRelativeRotation`函數來修改。通過不斷嘗試，發現只需要將`Rotation`設置為`[0, 90, 0]`即可，相當於旋轉了90度。完整修複代碼如下：

注：不能直接通過內存來修改，要用API來修改。

```
function fix_MuzzleLocation() {

    Interceptor.attach(base.add(0x670F110), {

        onEnter: function(args) {

            console.log("[process_before_shoot]");

            /* API */

            let K2_SetRelativeLocation = new NativeFunction(base.add(0x8AE6D70), "void", ["pointer", "float", "float", "float", "bool", "pointer", "bool"]); 

            let K2_SetRelativeRotation = new NativeFunction(base.add(0x8AE6F00), "void", ["pointer", "float", "float", "float", "bool", "pointer", "bool"]);

            // K2_SetRelativeLocation(All_Objects["MuzzleLocation"], 0, 0, 0, 0, ptr(0), 0);

            K2_SetRelativeRotation(All_Objects["MuzzleLocation"], 0, 90, 0, 0, ptr(0), 0);

            /* Prop */

            let RelativeLocation = All_Objects["MuzzleLocation"].add(0x11c);

            let RelativeRotation = All_Objects["MuzzleLocation"].add(0x128);

            console.log("location: ", JSON.stringify(readVector(RelativeLocation)))

            console.log("rotation: ", JSON.stringify(readVector(RelativeRotation)))

            // Replace

            let MuzzleLocation = args[0].add(0x4D8)

            MuzzleLocation.writePointer(All_Objects["MuzzleLocation"])

        },

        onLeave: function(retval) {

        }

    })

}
```

## 透視分析

在`Objects.txt`裡可以看到`FirstPersonCharacter_C`和`ThirdPersonCharacter_C`，它們分別是我控制的人物和假人。

```
[0x3c05]:
Name: FirstPersonCharacter_C
Class: FirstPersonCharacter_C
ObjectPtr: 0x71694840c0
ClassPtr: 0x7169582700

[0x3c53]:
Name: ThirdPersonCharacter
Class: ThirdPersonCharacter_C
ObjectPtr: 0x7168662630
ClassPtr: 0x716958c100
```

### 嘗試一：替換 Material

`// 沒效果`

```
let ThirdPersonCharacter_Mesh = ThirdPersonCharacter_obj.add(Offset.ACharacterToUSkeletalMeshComponent).readPointer();

let SetMaterial = new NativeFunction(ThirdPersonCharacter_Mesh.readPointer().add(0x598).readPointer(), "void", ["pointer", "int", "pointer"]);

SetMaterial(ThirdPersonCharacter_Mesh, 0, All_Objects["BaseMaterial"]);

// SetMaterial(ThirdPersonCharacter_Mesh, 0, All_Objects["DefaultTextMaterialOpaque"]);
// SetMaterial(ThirdPersonCharacter_Mesh, 0, All_Objects["m_SimpleVolumetricCloud"]);
// SetMaterial(ThirdPersonCharacter_Mesh, 0, All_Objects["DefaultSpriteMaterial"]);
// SetMaterial(ThirdPersonCharacter_Mesh, 0, All_Objects["PokeAHoleMaterial"]);
// SetMaterial(ThirdPersonCharacter_Mesh, 0, All_Objects["OculusMR_ChromaKey"]);
// SetMaterial(ThirdPersonCharacter_Mesh, 0, All_Objects["DebugMeshMaterial"]);
// SetMaterial(ThirdPersonCharacter_Mesh, 0, All_Objects["EmissiveMeshMaterial"]);
```

### 嘗試二：設置 `bDisableDepthTest` 為 0

`// 沒效果`

```
let ThirdPersonCharacter_Mesh = ThirdPersonCharacter_obj.add(Offset.ACharacterToUSkeletalMeshComponent).readPointer();

let GetNumMaterials = new NativeFunction(ThirdPersonCharacter_Mesh.readPointer().add(0x6e8).readPointer(), "int", ["pointer"]);
let GetMaterial = new NativeFunction(ThirdPersonCharacter_Mesh.readPointer().add(0x590).readPointer(), "pointer", ["pointer", "int"]);

let NumMaterials = GetNumMaterials(ThirdPersonCharacter_Mesh)
console.log("NumMaterials: ", NumMaterials);
for(let i = 0; i 
```

### 嘗試三：利用 `SetTexture` 隨便設置一個 `Texture`

`// 沒效果`

```
let ThirdPersonCharacter_Mesh = ThirdPersonCharacter_obj.add(Offset.ACharacterToUSkeletalMeshComponent).readPointer();

/* API */

// void SetTexture(Texture* InTexture);// 0x95efb5c

let SetTexture = new NativeFunction(base.add(0x8B2C4CC), "void", ["pointer", "pointer"]);

// Texture* GetTexture();// 0x95efa98

let GetTexture = new NativeFunction(ThirdPersonCharacter_Mesh.readPointer().add(0x1F8).readPointer(), "pointer", ["pointer"]);

/* Prop */

let ThirdPersonCharacter_Mesh = ThirdPersonCharacter_Mesh.add(Offset.USkeletalMeshComponentToSkeletalMesh).readPointer();

SetTexture(ThirdPersonCharacter_Mesh, All_Objects["T_ML_Rubber_Blue_01_N"]);
```

以下部份是賽後的分析。

找資料時發現類似這遊戲的人物透視效果基本上有以下兩種實現思路：

- 通過`Disable Depth Test`（[參考](https://www.bilibili.com/video/BV1Qy4y1e7gS/?vd_source=999a37555f77c5995df6185262c99be3)）。
- 通過`Custom Depth`（[參考](https://blog.csdn.net/u013007305/article/details/126011985)）。

但嘗試後發現遊戲似乎不是用上述方法實現透視效果的（不太確定，也有可能是我修改的地方不對）？

找了很久都沒有什麼思路，最終只好退而求其次，用一種「掩耳盜鈴」的方式來修復，具體思路如下：

- 調用`KismetSystemLibrary`的靜態函數`LineTraceSingle`來獲取`FirstPersonCharacter_C`和`ThirdPersonCharacter_C`之間的`HitResult`。
- 分析`HitResult`，會發現`ThirdPersonCharacter_C`沒有被遮擋時`HitResult.Distance`為0，否則為二者之間的距離。
- 因此可以根據`HitResult.Distance`是否為0來設置`ThirdPersonCharacter_C`是否渲染到`MainPass`。

具體調用`LineTraceSingle`、獲取`HitResult.Distance`的代碼如下：

```
function getFirstPersonThirdPersonDistance() {
    // static bool LineTraceSingle(const Object* WorldContextObject, const Vector Start, const Vector End, byte TraceChannel, bool bTraceComplex, out const Actor*[] ActorsToIgnore, byte DrawDebugType, out HitResult OutHit, bool bIgnoreSelf, LinearColor TraceColor, LinearColor TraceHitColor, float DrawTime);// 0x9471770
    let LineTraceSingle = new NativeFunction(base.add(0x8D1AA78), "bool", ["pointer", "float", "float", "float", "float", "float", "float", "uint8", "bool", "pointer", "uint8", "pointer", "bool", "float", "float", "float", "float", "float", "float", "float", "float", "float"]);

    let buf1 = Memory.alloc(0x1000);
    let HitResult = Memory.alloc(0x1000);

    let start_loc = getActorLocation(FirstPersonCharacter_C_obj);
    let end_loc = getActorLocation(ThirdPersonCharacter_obj);

    let r = LineTraceSingle(FirstPersonCharacter_C_obj, start_loc[0], start_loc[1], start_loc[2], end_loc[0], end_loc[1], end_loc[2], 0, 0, buf1, 1, HitResult, 0, 255, 0, 0, 0, 0, 255, 0, 0, 10000);

    let HitResult_Distance = HitResult.add(0x8).readFloat();

    return HitResult_Distance;
}
```

時機選擇在`Actor`類的`ReceiveTick`函數，在其中判斷是否渲染：

```
function bypassWallhack() {

    let SetRenderInMainPass = new NativeFunction(base.add(0x8AB9E58), "void", ["pointer", "bool"]);

    let ThirdPersonCharacter_Mesh = ThirdPersonCharacter_obj.add(Offset.ACharacterToUSkeletalMeshComponent).readPointer();

        // void ReceiveTick(float DeltaSeconds);// 0x6c50500

    Interceptor.attach(base.add(0x6c50500), {

        onEnter: function(args) {

            if (getFirstPersonThirdPersonDistance() == 0) {

                console.log("can see ThirdPerson")

                SetRenderInMainPass(ThirdPersonCharacter_Mesh, 1); 

            } else {

                console.log("can not see ThirdPerson")

                SetRenderInMainPass(ThirdPersonCharacter_Mesh, 0); 

            }

        }
```
