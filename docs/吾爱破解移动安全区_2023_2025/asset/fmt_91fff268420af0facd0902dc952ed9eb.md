
# IDA 解密脚本与运行日志

## 脚本代码

```python
import idaapi
import idautils
import idc

def decrypt(cipher, keybox):
    # 计算密文长度的一半作为结果缓冲区大小
    v4 = len(cipher) // 2
    result = [0] * (v4 + 1)  # 预留终止符位置
    result[v4] = 0  # 终止符

    if not cipher:  # 处理空输入
        return bytes(result[:v4])

    n0x3A = cipher[0]
    n8 = 0
    v8_idx = 2  # v8 初始指向 cipher + 2
    v9_idx = 0  # 结果数组索引

    while n0x3A != 0:
        # 获取当前处理的两个字符
        n0x3A_1 = cipher[v8_idx - 1]

        # 计算v11
        if n0x3A >= 0x3A:
            v11 = -112
        else:
            v11 = 0

        # 计算v12
        if n0x3A_1 >= 0x3A:
            v12 = -55
        else:
            v12 = -48

        # n8循环（0-7）
        if n8 == 8:
            n8 = 0

        # 计算解密字节并异或
        decoded_byte = keybox[n8] ^ (v11 + 16 * n0x3A + v12 + n0x3A_1)
        # 确保结果为无符号字节
        result[v9_idx] = decoded_byte & 0xFF

        # 更新索引和变量
        v9_idx += 1
        n0x3A = cipher[v8_idx] if v8_idx  数据地址: {hex(arg_value)} ('{value_name}')"
                # else:
                #     return f"指令: {hex(ea)} ({idc.generate_disasm_line(ea,0)}) => 立即数/地址: {hex(arg_value)}"

            # 情况 B: 从内存加载 (LDR)
            elif mnem == "LDR" and op_type1 == idc.o_displ:
                # LDR X0, [X19, #0x20]
                # 更复杂，需要进一步的数据流分析来追踪 X19 或内存的值
                print(f"指令: {hex(ea)} ({idc.generate_disasm_line(ea,0)}) => 复杂内存加载 (LDR)，需要数据流分析")
                return

            # 其他复杂情况，如 MOV X0, X1
            else:
                print(f"指令: {hex(ea)} ({idc.generate_disasm_line(ea,0)}) => 复杂寄存器/内存操作")
                return

        # 移动到前一条指令
        ea = idc.prev_head(ea)

    return None
# --- 脚本运行设置 ---
KEYBOX_START_EA = 0xB3671  
KEYBOX_END_EA = 0xB3989  

# 运行函数
decfuncs=find_all_decfunc(KEYBOX_START_EA, KEYBOX_END_EA)
#print(decfunc)
#decfuncs=[131420]
for idx,func_start in enumerate(decfuncs):
    for xref in idautils.XrefsTo(func_start):
        ref_from_ea = xref.frm
        cipher=get_aarch64_first_arg(ref_from_ea)
        if not cipher:
            print(f"需要手动解密 : {hex(ref_from_ea)}")
            continue
        #print(cipher)
        keybox=list(idc.get_bytes(START_EA+idx*8,8))
        #print(keybox)
        plain=decrypt(cipher,keybox)
        print(f"[{hex(ref_from_ea)}]: {plain}")
        idc.set_cmt(ref_from_ea, f"{plain.decode('utf-8', errors='ignore')}", True)
        #break

        #print(hex(ref_from_ea))
```

## 运行日志

```text
--- 正在查找范围 0xb3671 到 0xb3989 内的数据引用 ---

--- 查找结束 ---

[0xab88]: b'unknown\x00'

[0xacd4]: b'bdp.\x00\x00\x00\x00'

[0xad9c]: b'FeatureMetaData'

[0xb4b0]: b'/sbin/.magisk/'

[0xb8b0]: b'ro.build.version.release'

[0xbacc]: b'FeatCheckRoot'

[0xbb6c]: b'/etc/mumu-configs/device-prop-configs/mumu.config'

[0xc07c]: b'FeatureAntiMthook'

[0xc194]: b'b0f957fc8d26239fe1d7d0fd0851ef3dcf86b8c2'

[0xc3b0]: b'FeatureSharedPrefEncryption'

[0xc4ec]: b' 复杂寄存器/内存操作

需要手动解密 : 0x1465c   -> b'%s/.bdlock'

[0x16724]: b'_exit\x00\x00\x00'

[0x1ea4c]: b'\x00\x00\x00\x00\x00\x00\x00\x00'

[0x26f9c]: b'%s/.1\x00\x00\x00'

[0x2dbb0]: b'/proc/self/maps'

指令: 0x34fa4 (ADD             X0, X9, #a5d21d4fc2523c1@PAGEOFF; "5D21D4FC2523C15E582CE9EF3E29C7585C2D98E"...) => 复杂寄存器/内存操作

需要手动解密 : 0x34fac libandroid_runtime.so

指令: 0x47834 (MOV             X0, X28) => 复杂寄存器/内存操作

需要手动解密 : 0x47838  %n%*s%n %n%*s%n %n%*s%n %n%*s%n %d %d

[0x47a30]: b' %n%*s%n %n%*s%n %n%*s%n %n%*s%n %d %d'

[0x4e450]: b'java/lang/Throwable'

指令: 0x4ed4c (ADD             X0, X8, #a5026d2ef242ed7@PAGEOFF; "5026D2EF242ED71E5038C6B20A24C7584721C2E"...) => 复杂寄存器/内存操作

需要手动解密 : 0x4ed50 android/app/ActivityThread

[0x4f488]: b'/proc/self/mem'

[0x4fdf0]: b'gdbserver'

[0x50cbc]: b'/proc/self/fd/%d'

[0x78364]: b'.db\x00\x00\x00\x00\x00'

[0x7c850]: b'/\x00\x00\x00\x00\x00\x00\x00'

[0xacf0]: b'stamp-cert-sha256'

[0xb4cc]: b'/sbin/.core/mirror'

[0xb8c4]: b'ro.build.version.incremental'

[0xbb88]: b'/data/local/cfg-gdaeg'

[0xc410]: b'FeatureFileEncryption'

[0xd044]: b'\x00\x00\x00\x00\x00\x00\x00\x00'

[0x11a4c]: b'/proc/self/maps'

指令: 0x14404 (MOV             X0, X8) => 复杂寄存器/内存操作

需要手动解密 : 0x14408 n1\x00\x00\x00\x00\x00\x00

[0x16744]: b'exit\x00\x00\x00\x00'

[0x1d864]: b'assets/%s%d.i.dex'

[0x27000]: b'%s\x00\x00\x00\x00\x00\x00'

[0x2dbc0]: b'r\x00\x00\x00\x00\x00\x00\x00'

指令: 0x34fdc (ADD             X0, X8, #a3ee74c063d4b3d@PAGEOFF; "3EE74C063D4B3D0405E741073E66301807E75A1"...) => 复杂寄存器/内存操作

需要手动解密 : 0x34fe0  __android_log_buf_write

[0x4774c]: b'/proc/mounts'

[0x4e474]: b'getStackTrace'

指令: 0x4eda8 (ADD             X0, X8, #a12e84c0b325835@PAGEOFF; "12E84C0B325835082CD943093E5C20") => 复杂寄存器/内存操作

需要手动解密 : 0x4edac sPackageManager

[0x4f4a8]: b'/proc/self/pagemap'

[0x4fe00]: b'gdb\x00\x00\x00\x00\x00'

[0x50dd8]: b'bin/mt/apksignaturekillerplus/HookApplication'

[0x784b0]: b'-journal'

[0x79f38]: b'/proc/self/fd/%d'

[0x7c9a8]: b'/%s/%s/%s'

[0xaff4]: b'FeatLogProtection'

[0xb4dc]: b'/sbin/.core/img'

[0xb8d8]: b'ro.build.version.codename'

[0xbb98]: b'/data/local/cfg-yzi'

[0xc468]: b'FeatureSQLiteEncryption'

[0xd0b4]: b'\x00\x00\x00\x00\x00\x00\x00\x00'

[0x11a5c]: b'r\x00\x00\x00\x00\x00\x00\x00'

指令: 0x14418 (MOV             X0, X8) => 复杂寄存器/内存操作

需要手动解密 : 0x1441cb'(Landroid/content/Context;Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;I)V

[0x16764]: b'pthread_create'

[0x1d8ec]: b'assets/%s%d.jar'

[0x27044]: b'%s/1\x00\x00\x00\x00'

[0x2a3d0]: b'classes\x00'

[0x2dbfc]: b'r--p\x00\x00\x00\x00'

[0x4775c]: b'r\x00\x00\x00\x00\x00\x00\x00'

[0x4e484]: b'()[Ljava/lang/StackTraceElement;'

指令: 0x4edb8 (ADD             X0, X8, #a2b08d38363be3d@PAGEOFF; "2B08D38363BE3D58480AD28965B43A484819D0C"...) => 复杂寄存器/内存操作

需要手动解密 : 0x4edbc Landroid/content/pm/IPackageManager;

[0x4f50c]: b'/proc/%d/mem'

[0x4fe10]: b'android_server'

[0x50e44]: b'libmthook.so'

[0x78244]: b'libsqlite.so'

[0x78728]: b'libsqlite.so'

[0x7a060]: b'/shared_prefs/'

[0x7cdbc]: b'/proc/self/fd/%d'

[0xb4ec]: b'/sbin/.core/db-0/magisk.db'

[0xb8ec]: b'ro.build.version.sdk'

[0xbba8]: b'/data/local/cfg-hzg'

[0xbf40]: b'FeatureNativeAntiDebug'

[0xd124]: b'\x00\x00\x00\x00\x00\x00\x00\x00'

指令: 0x14434 (MOV             X0, X8) => 复杂寄存器/内存操作

需要手动解密 : 0x14438 b'n2\x00\x00\x00\x00\x00\x00'

[0x16784]: b'pthread_join'

[0x1d974]: b'assets/%s%d.d.jar'

[0x1ed80]: b'.\x00\x00\x00\x00\x00\x00\x00'

指令: 0x1ef7c (MOV             X0, X22) => 复杂寄存器/内存操作

需要手动解密 : 0x1ef80  b'.\x00\x00\x00\x00\x00\x00\x00'

[0x27148]: b'/%d.jar\x00'

[0x2a3ec]: b'.dex\x00\x00\x00\x00'

[0x2dc0c]: b'%s %s %*s %*s %*s %s'

[0x478f0]: b'rw\x00\x00\x00\x00\x00\x00'

[0x4e4e4]: b'java/lang/StackTraceElement'

指令: 0x4ee60 (ADD             X0, X8, #aCe8573fb3fb053_0@PAGEOFF; "CE8573FB3FB053ADC3CB77FF76B057A0D0CB55E"...) => 复杂寄存器/内存操作

需要手动解密 : 0x4ee64   java/lang/reflect/Proxy

[0x4f540]: b'/proc/%d/pagemap'

[0x50eac]: b'/data/data/'

[0x52e88]: b'/proc/%d/cmdline'

[0x79a20]: b'SQLite\x00\x00'

[0x7a15c]: b'.\x00\x00\x00\x00\x00\x00\x00'

[0x7cee8]: b'android/os/Environment'

[0xb4fc]: b'/dev/.magisk.unblock'

[0xb900]: b'ro.product.model'

[0xbbb8]: b'/data/local/cfg-znz'

[0xbd74]: b'FeatCheckPmsHook'

[0xbe10]: b'FeatureMemoryChecker'

指令: 0x14448 (MOV             X0, X8) => 复杂寄存器/内存操作

需要手动解密 : 0x1444c b'(Landroid/content/Context;)V'

[0x167a4]: b'memcpy\x00\x00'

[0x1da4c]: b'%s/lib\x00\x00'

[0x1ed9c]: b'..\x00\x00\x00\x00\x00\x00'

指令: 0x1ef94 (MOV             X0, X23) => 复杂寄存器/内存操作

需要手动解密 : 0x1ef98  b'..\x00\x00\x00\x00\x00\x00'

指令: 0x271ac (MOV             X0, X28) => 复杂寄存器/内存操作

需要手动解密 : 0x271b0 b'/%d.odex'

指令: 0x29ae0 (MOV             X0, X24) => 复杂寄存器/内存操作

需要手动解密 : 0x29ae4   b'META-INF/'

[0x2dc1c]: b'r-xp\x00\x00\x00\x00'

[0x47af4]: b'1\x00\x00\x00\x00\x00\x00\x00'

[0x4e50c]: b'toString'

[0x50a00]: b'android/os/Debug'

[0x50fc0]: b'/data/user/0'

[0x78260]: b'\x00\x00\x00\x00\x00\x00\x00\x00'

[0x7876c]: b'\x00\x00\x00\x00\x00\x00\x00\x00'

[0x7a224]: b'.xml\x00\x00\x00\x00'

[0x7cf10]: b'getExternalStorageDirectory'

[0xb518]: b'/sbin/magiskinit'
```
