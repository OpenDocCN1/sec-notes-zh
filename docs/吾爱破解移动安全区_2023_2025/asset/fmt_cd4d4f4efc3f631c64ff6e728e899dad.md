
```
class Vector {
    //设置向量对象
    constructor(x, y, z) {
        this.x = x;
        this.y = y;
        this.z = z;
    }
    // 将向量转换为字符串
    toString() {
        return `(${this.x}, ${this.y}, ${this.z})`;
    }
}
function dumpVector(addr) {
    // dumpAddr('firstPersion_RootComponent',firstPersion_RootComponent_ptr,0x152)
    // 从地址空间中读取三个浮点数
    const values = Memory.readByteArray(addr, 3 * 4); // 3个float共占12个字节
    // 解析浮点数并初始化 Vector 对象
    const vec = new Vector(
        new Float32Array(values, 0, 1)[0], // 读取第一个浮点数
        new Float32Array(values, 4, 1)[0], // 读取第二个浮点数
        new Float32Array(values, 8, 1)[0] // 读取第三个浮点数
    );
    console.log("[+] 坐标：", vec); //打出坐标。
}
function get_box_vec(){
    getlibue4();
    var base = Module.getBaseAddress("libUE4.so");
    var actorsAddr = getActorsAddr();
    for (var key in actorsAddr) {
        if (key.includes("EditorCube")){
            var actor_addr = actorsAddr[key];
            var RootComponent = actor_addr.add(0x130).readPointer();
            var Location = RootComponent.add(0x11C);
            console.log("[+]Actor: ", key);
            dumpVector(Location);
        }
    }
}
```
## Part 1
[+]Actor:  EditorCube8

[+] 坐标： (843.998779296875, -1169.60498046875, 295.23797607421875)

[+]Actor:  EditorCube9

[+] 坐标： (1464.425537109375, -657.312744140625, 245.24537658691406)

[+]Actor:  EditorCube10

[+] 坐标： (1464.425537109375, -46.785343170166016, 245.24537658691406)

[+]Actor:  EditorCube11

[+] 坐标： (860.821533203125, -46.78565216064453, 245.23817443847656)

[+]Actor:  EditorCube12

[+] 坐标： (1307.8231201171875, 714.8047485351563, 245.24351501464844)

[+]Actor:  EditorCube13

[+] 坐标： (1310.8233642578125, 874.9715576171875, 245.2435302734375)

[+]Actor:  EditorCube14

[+] 坐标： (1310.8233642578125, 790.3173828125, 395.24346923828125)

[+]Actor:  EditorCube15

[+] 坐标： (-896.7808227539063, 828.98193359375, 245.21722412109375)

[+]Actor:  EditorCube16

[+] 坐标： (-1034.357666015625, 746.9696655273437, 245.21559143066406)

[+]Actor:  EditorCube17

[+] 坐标： (-961.6449584960938, 790.31689453125, 395.21636962890625)

[+]Actor:  EditorCube18

[+] 坐标： (-1439.8385009765625, -811.4144897460937, 245.21078491210937)

[+]Actor:  EditorCube19

[+] 坐标： (-1439.401123046875, -811.4213256835937, 395.2096862792969)

[+]Actor:  EditorCube20

[+] 坐标： (-1309.3416748046875, -373.11163330078125, 295.2123107910156)

[+]Actor:  EditorCube21

[+] 坐标： (-1123.3922119140625, 153.2134246826172, 245.2145233154297)

## Part 2
[+]Actor:  EditorCube8

[+] 坐标： (859.3866577148438, -1172.3924560546875, 295.2377624511719)

[+]Actor:  EditorCube9

[+] 坐标： (1464.425537109375, -657.312744140625, 245.24537658691406)

[+]Actor:  EditorCube10

[+] 坐标： (1464.425537109375, -46.785343170166016, 245.24537658691406)

[+]Actor:  EditorCube11

[+] 坐标： (860.821533203125, -46.78565216064453, 245.23817443847656)

[+]Actor:  EditorCube12

[+] 坐标： (1307.8231201171875, 714.8047485351563, 245.24351501464844)

[+]Actor:  EditorCube13

[+] 坐标： (1310.8233642578125, 874.9715576171875, 245.2435302734375)

[+]Actor:  EditorCube14

[+] 坐标： (1310.8233642578125, 790.3173828125, 395.24346923828125)

[+]Actor:  EditorCube15

[+] 坐标： (-896.7808227539063, 828.98193359375, 245.21722412109375)

[+]Actor:  EditorCube16

[+] 坐标： (-1034.357666015625, 746.9696655273437, 245.21559143066406)

[+]Actor:  EditorCube17

[+] 坐标： (-961.6449584960938, 790.31689453125, 395.21636962890625)

[+]Actor:  EditorCube18

[+] 坐标： (-1439.8385009765625, -811.4144897460937, 245.21078491210937)

[+]Actor:  EditorCube19

[+] 坐标： (-1439.401123046875, -811.4213256835937, 395.2096862792969)

[+]Actor:  EditorCube20

[+] 坐标： (-1309.3416748046875, -373.11163330078125, 295.2123107910156)

[+]Actor:  EditorCube21

[+] 坐标： (-1123.3922119140625, 153.2134246826172, 245.2145233154297)

通过上述方法比对之后不难发现只有`EditorCube8`的值发生了变化，那么可以确定这个就是我们的目标`Actor`，接下来寻找这个箱子的世界坐标并给坐标下硬件断点打印调用栈

通过`Frida` hook 我们可以获取角色视角的`Vector`的值，通过`PlayerController` + `0x288`的偏移处可以获取到`ControlRotation`的地址，其中后续的12个字节就是角色的相机的位置信息，即`FRotator`结构体，其成员变量分别是`Pitch`、`Yaw`、`Roll`，使用`stackplz`给此处下硬件断点(`w`)并打印调用栈可回溯到写入此内存地址的函数的调用栈

```
function my_hack(){
    getlibue4();
    var actorsAddr = getActorsAddr();
    var base = Module.getBaseAddress("libUE4.so");
    console.log("[+] UE4 base = ", base);
    hook_addr(base.add(0x670f3f8));
    for (var key in actorsAddr) {
        //console.log("[+] current actor is : ", key);
        //console.log("UE4 base = ", base);
        if (actorsAddr[key].readPointer() - base == 0xA63BE28) {
            console.log("[+] Successful Get target Actor : ", key);
            var player_addr = actorsAddr[key];
            var RecoilAccumulationRate = player_addr.add(0x538);
            var CharacterMovementComponent = player_addr.add(0x288).readPointer();
            var MaxAcceleration = CharacterMovementComponent.add(0x1A0);
            Memory.writeFloat(MaxAcceleration, 1000);
            Memory.writeFloat(RecoilAccumulationRate, 0);
        }
        if (key == "PlayerController"){
            console.log("[+] Successful Get target Actor : ", key);
            var ControlRotation = actorsAddr[key].add(0x288);
            var Pitch = ControlRotation.add(0x0).readFloat();
            var Yaw = ControlRotation.add(0x4).readFloat();
            var Roll = ControlRotation.add(0x8).readFloat();
            console.log("[+] Pitch = {}, Yaw = {}, Roll = {}", Pitch, Yaw, Roll);
            console.log("[+] ControlRotation = ", ControlRotation);

        }
    }
}

下断点
```

`./stackplz --brk 0x758eb0f5c8:w --brk-len 4 --stack -o pitch.log`
