
```java
String retValFormatted = String.format("0x%x", retValLong);

UnidbgPointer pointer = UnidbgPointer.pointer(emulator, retValLong);

if (pointer != null) {
    String cstring = safeReadCString(pointer);

    // 如果是一个可打印的字符串，则附加到日志中

    if (isPrintable(cstring)) {
        retValFormatted += String.format(" -> \"%s\"", cstring);
    }
}

traceStream.printf("%sRET
```

完整代码：

```java
package com.example.ndkdemo;

import com.github.unidbg.*;
import com.github.unidbg.arm.backend.Backend;
import com.github.unidbg.arm.backend.CodeHook;
import com.github.unidbg.arm.backend.UnHook;
import com.github.unidbg.arm.backend.Unicorn2Factory;
import com.github.unidbg.arm.context.Arm64RegisterContext;
import com.github.unidbg.debugger.BreakPointCallback;
import com.github.unidbg.debugger.Debugger;
import com.github.unidbg.debugger.FunctionCallListener;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.dvm.DalvikModule;
import com.github.unidbg.linux.android.dvm.DvmClass;
import com.github.unidbg.linux.android.dvm.VM;
import com.github.unidbg.memory.Memory;
import com.github.unidbg.pointer.UnidbgPointer;
import unicorn.Arm64Const;
import unicorn.ArmConst;

import java.io.*;

public class MainActivity {
    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;
    private final DvmClass security;
    private final boolean logging;
    private long instructionCount = 0; // 用于指令计数

    MainActivity(boolean logging) {
        this.logging = logging;

        // 1. 创建模拟器实例
        emulator = AndroidEmulatorBuilder.for64Bit()
                .setProcessName("com.example.ndkdemo")
                .addBackendFactory(new Unicorn2Factory(true))
                .build();

        // 2. 设置内存和系统库解析
        final Memory memory = emulator.getMemory();
        memory.setLibraryResolver(new AndroidResolver(23));

        // 3. 创建Dalvik虚拟机并加载SO文件
        vm = emulator.createDalvikVM();
        vm.setVerbose(logging);
        DalvikModule dm = vm.loadLibrary(new File("unidbg-android/src/test/java/com/zj/wuaipojie/util/libndkdemo.so"), false);
        dm.callJNI_OnLoad(emulator);
        module = dm.getModule();

        // 4. 获取DVM类
        security = vm.resolveClass("com/example/ndkdemo/MainActivity");

        // 5. 设置指令计数钩子 (可选)
        emulator.getBackend().hook_add_new(new CodeHook() {
            @Override
            public void hook(Backend backend, long address, int size, Object user) {
                instructionCount++;
            }
            @Override
            public void onAttach(UnHook unHook) {}
            @Override
            public void detach() {}
        }, module.base, module.base + module.size, null);

        // 6.设置函数追踪与x0寄存器检查
        attachTraceAndInspectX0(module.base + 0x15F8);
    }

    /**
     * 【模块化功能】: 附加一个函数调用追踪器 (traceFunctionCall)。
     * 此功能会监听所有的函数调用，并以树状结构打印出调用关系。
     * @param traceStream 日志输出流
     */
    public void attachFunctionCallTracer(final PrintStream traceStream) {
        Debugger debugger = emulator.attach();
        System.out.println("函数调用关系追踪器已附加，结果将输出到日志文件。");

        debugger.traceFunctionCall(null, new FunctionCallListener() {
            private int depth = 0;

            private String getPrefix(int currentDepth) {
                if (currentDepth  emulator, long callerAddress, long functionAddress) {
                String prefix = getPrefix(depth + 1);
                String details = emulator.getUnwinder().formatAddressDetails(functionAddress);
                traceStream.printf("%sCALL -> %s%n", prefix, details);
                depth++;
            }

            @Override
            public void postCall(Emulator emulator, long callerAddress, long functionAddress, Number[] args) {
                depth--;
                String prefix = getPrefix(depth + 1);
                String details = emulator.getUnwinder().formatAddressDetails(functionAddress);

                Backend backend = emulator.getBackend();
                Number retVal = emulator.is64Bit() ? backend.reg_read(Arm64Const.UC_ARM64_REG_X0) : backend.reg_read(ArmConst.UC_ARM_REG_R0);
                long retValLong = retVal.longValue();

                // 尝试将返回值作为指针解析
                String retValFormatted = String.format("0x%x", retValLong);
                UnidbgPointer pointer = UnidbgPointer.pointer(emulator, retValLong);
                if (pointer != null) {
                    String cstring = safeReadCString(pointer);
                    // 如果是一个可打印的字符串，则附加到日志中
                    if (isPrintable(cstring)) {
                        retValFormatted += String.format(" -> \"%s\"", cstring);
                    }
                }

                traceStream.printf("%sRET   {
            traceHook[0] = emu.traceCode(module.base, module.base + module.size);
            return true;
        });

        long returnAddress = callAddress + 4;
        emulator.attach().addBreakPoint(returnAddress, new BreakPointCallback() {
            @Override
            public boolean onHit(Emulator emu, long address) {
                if (traceHook[0] != null) {
                    traceHook[0].stopTrace();
                    System.out.println();
                }

                Arm64RegisterContext ctx = emu.getContext();
                long x0 = ctx.getXLong(0);
                System.out.printf("[+] 检查地址: 0x%x, x0寄存器值: 0x%x\n", address, x0);

                UnidbgPointer pointer = UnidbgPointer.pointer(emu, x0);
                if (pointer == null) {
                    System.out.println("[-] x0的值不是一个有效的指针或指向未映射的内存");
                    return true;
                }

                String cstring = safeReadCString(pointer);
                if (cstring != null && isPrintable(cstring)) {
                    System.out.println("[+] x0指向的字符串: " + cstring);
                } else {
                    System.out.println("[-] x0指向的内容不是一个可打印的字符串");
                }

                int dumpSize = 256;
                byte[] data = pointer.getByteArray(0, dumpSize);
                System.out.println("[+] x0指向内存的HexDump (前" + dumpSize + "字节):");
                System.out.println(prettyHexDump(data, x0));
                System.out.println("--- x0寄存器检查结束 ---\n");

                return true;
            }
        });
    }

    private static String safeReadCString(UnidbgPointer p) {
        try {
            return p.getString(0);
        } catch (Exception e) {
            return null;
        }
    }

    private static boolean isPrintable(String s) {
        if (s == null || s.isEmpty()) {
            return false;
        }
        int printableChars = 0;
        for (int i = 0; i = 32 && c = 2 && (double) printableChars / s.length() > 0.8;
    }

    private static String prettyHexDump(byte[] data, long baseAddr) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i = 32 && b 
```

## 二、MCP 配置

### MCP 概念

MCP（Model Context Protocol，模型上下文协议）是什么？

MCP是一种标准化协议，旨在为人工智能模型（如大语言模型）与外部工具、数据源之间的交互提供统一接口。(为 AI 装上了手脚)

MCP是AI智能体与外部工具的"USB接口"，定义了AI模型与外部工具（如API、数据库、文档编辑器等）的交互标准，开发者无需为每个工具单独开发适配代码。

MCP协议旨在实现大型语言模型（LLM）与外部数据源和工具之间的无缝集成，通过提供标准化的接口，使AI应用程序能够安全、可控地与本地或远程资源进行交互。

![](https://img2595.oss-cn-hangzhou.aliyuncs.com/20250811212706536.png)

[项目地址](https://github.com/mrexodia/ida-pro-mcp)

### 要求

1. IDA Pro 8.3 以上，最好是 9
2. `python3.11` 或更高版本，使用 `idapyswitch` 切换到 Python 版本
3. 支持 MCP 的客户端，这里以 Cursor 为例

### 环境配置

```bash
pip install git+https://github.com/mrexodia/ida-pro-mcp
```
