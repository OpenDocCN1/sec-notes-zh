
# Frida PyNumber Hook 脚本

```
 // ---------------------------------------------------------------------

// Hook PyLong_FromUnicodeObject / PyLong_FromString

if (symbols.PyLong_FromUnicodeObject) {

    Interceptor.attach(symbols.PyLong_FromUnicodeObject, {

        onEnter(args) {

            this.sRaw = objToStr(args[0]);

            this.base = args[1].toInt32();

        },

        onLeave(retval) {

            const got = objToStr(retval);

            logLine(`[PyLong_FromUnicodeObject] int(s, base=${this.base}) s=${this.sRaw} -> ${formatForLogging(got)}`);

        }

    });

}

if (symbols.PyLong_FromString) {

    Interceptor.attach(symbols.PyLong_FromString, {

        onEnter(args) {

            try { this.s = Memory.readUtf8String(args[0]); } catch (e) { this.s = ''; }

            this.base = args[2].toInt32();

        },

        onLeave(retval) {

            const got = objToStr(retval);

            logLine(`[PyLong_FromString] int(s, base=${this.base}) s=${this.s} -> ${formatForLogging(got)}`);

        }

    });

}

function symbolFor(name) {

    const map = {

        PyNumber_Add: '+',

        PyNumber_InPlaceAdd: '+=',

        PyNumber_Subtract: '-',

        PyNumber_Multiply: '*',

        PyNumber_And: '&',

        PyNumber_Or: '|',

        PyNumber_Xor: '^',

        PyNumber_Lshift: '>>',

        PyNumber_InPlaceRshift: '>>=',

        PyNumber_Power: '**',

        PyNumber_Remainder: '%',

        PyNumber_Index: 'index'

    };

    return map[name] || '?';

}

// ---------------------------------------------------------------------

//

// function hookPyNumber(name, ptr) {

//     if (!ptr) return;

//     Interceptor.attach(ptr, {

//         onEnter(args) {

//             this.aPtr = args[0];

//             this.bPtr = args[1];

//             this.aRaw = objToStr(this.aPtr);

//             this.bRaw = objToStr(this.bPtr);

//             this.a = formatForLogging(this.aRaw, 8);

//             this.b = formatForLogging(this.bRaw, 8);

//             try {

//                 this.cPtr = args[2];

//                 if (this.cPtr && !this.cPtr.isNull()) {

//                     this.cRaw = objToStr(this.cPtr);

//                     this.c = formatForLogging(this.cRaw, 8);

//                 } else {

//                     this.cPtr = null;

//                 }

//             } catch (e) { this.cPtr = null; }

//         },

//         onLeave(retval) {

//             if (name === 'PyNumber_Power') {

//                 if (this.cPtr) {

//                     logLine(`[${name}] pow(${this.a}, ${this.b}, ${this.c}) = ${formatForLogging(objToStr(retval), 8)}`);

//                 } else {

//                     logLine(`[${name}] ${this.a} ** ${this.b} = `);

//                 }

//                 return;

//             }

//             logLine(`[${name}] ${this.a} ${symbolFor(name)} ${this.b} = ${formatForLogging(objToStr(retval), 8)}`);

//         }

//     });

// }

function hookPyNumber(name, ptr) {

    if (!ptr) return;

    Interceptor.attach(ptr, {

        onEnter(args) {

            // onEnter 的逻辑保持不变，依然是捕获原始指针和字符串

            this.aPtr = args[0];

            this.bPtr = args[1];

            this.aRaw = objToStr(this.aPtr);

            this.bRaw = objToStr(this.bPtr);

            this.a = formatForLogging(this.aRaw, 8);

            this.b = formatForLogging(this.bRaw, 8);

            try {

                this.cPtr = args[2];

                if (this.cPtr && !this.cPtr.isNull()) {

                    this.cRaw = objToStr(this.cPtr);

                    this.c = formatForLogging(this.cRaw, 8);

                } else {

                    this.cPtr = null;

                    this.cRaw = null;

                    this.c = null;

                }

            } catch (e) {

                this.cPtr = null;

                this.cRaw = null;

                this.c = null;

            }

        },

        onLeave(retval) {

            // --- 主要修改部分在这里 ---

            // 1. 统一检查并准备用于日志的字符串

            // 检查第一个操作数 a

            let aStr = this.aRaw.length > 32 ? "num" : this.a;

            // 检查第二个操作数 b

            let bStr = this.bRaw.length > 32 ? "num" : this.b;

            // 检查返回值 (c)

            const outRaw = objToStr(retval);

            let outFmt = outRaw.length > 32 ? "num" : formatForLogging(outRaw, 8);

            // 2. 根据不同的函数名，使用准备好的字符串来格式化日志

            if (name === 'PyNumber_Power') {

                // 对于 pow(a, b, mod)，第三个参数也需要检查

                let modStr = this.c ? (this.cRaw.length > 32 ? "num" : this.c) : "None";

                logLine(`[${name}] pow(${aStr}, ${bStr}, ${modStr}) = ${outFmt}`);

                return;

            }

            // 对于 PyNumber_Remainder 和其他所有二元运算

            logLine(`[${name}] ${aStr} ${symbolFor(name)} ${bStr} = ${outFmt}`);

        }

    });

}

// ---------------------------------------------------------------------

[

    'PyNumber_Add',

    'PyNumber_InPlaceAdd',

    'PyNumber_Subtract',

    'PyNumber_Multiply',

    'PyNumber_And',

    'PyNumber_Or',

    'PyNumber_Xor',

    'PyNumber_Lshift',

    'PyNumber_Rshift',

    'PyNumber_InPlaceRshift',

    'PyNumber_Remainder',

    'PyNumber_Index',

    'PyNumber_Power'

].forEach(n => hookPyNumber(n, symbols[n]));

// 比较操作

const cmpOps = ['', '>='];

if (symbols.PyObject_RichCompareBool) {

    Interceptor.attach(symbols.PyObject_RichCompareBool, {

        onEnter(args) {

            this.ptrA = args[0];

            this.ptrB = args[1];

            this.op = args[2].toInt32();

            this.aRaw = objToStr(this.ptrA);

            this.bRaw = objToStr(this.ptrB);

            this.a = formatForLogging(this.aRaw, 8);

            this.b = formatForLogging(this.bRaw, 8);

        },

        onLeave(retval) {

            const opStr = cmpOps[this.op] || `op(${this.op})`;

            if (this.ptrA.equals(this.ptrB)) return;

            if (this.aRaw === this.bRaw && this.op === 2) return;

            logLine(`[CompareBool] ${this.a} ${opStr} ${this.b} -> ${retval.toInt32()}`);

        }

    });

}

if (symbols.PyObject_RichCompare) {

    Interceptor.attach(symbols.PyObject_RichCompare, {

        onEnter(args) {

            this.aRaw = objToStr(args[0]);

            this.bRaw = objToStr(args[1]);

            this.op = args[2].toInt32();

            this.a = formatForLogging(this.aRaw, 8);

            this.b = formatForLogging(this.bRaw, 8);

        },

        onLeave(retval) {

            logLine(`[CompareObj] ${this.a} ${cmpOps[this.op]} ${this.b} -> ${objToStr(retval)}`);

        }

    });

}

logLine(`\n✅ Hooks installed on ${pymod.name}`);

logLine(`🧮 PyNumber_* operations + Compare ops are now being traced in HEX...\n`);
```

## Hook 效果说明

但是有个问题，`hook`不是万能的，`hook`不全所有的函数，比如对于`Lshift`、`Rshift`等逻辑操作来说，是`hook`不到的，因为有可能他就不走 `Py`提供的库函数，直接在`pyd`中自己实现了

总结了一下，我写的`hook_test`：

```
PyNumber_Add: 1302235
PyNumber_Subtract: 1166455
PyNumber_Multiply: 83799682050
PyNumber_And: 288
PyNumber_Or: 1301947
PyNumber_Xor: 1301659
PyNumber_Lshift: 4937380
PyNumber_Rshift: 33945
PyNumber_InPlaceRshift: 33945
PyNumber_Power: 1523607579025
PyNumber_Remainder: 45
PyNumber_Index: 42
PyNumber_InPlaceAdd: 1247924
CompareObj

一共14种，但是我只`hook`到了9种：
```

```
using python module: python312.dll 0x7ffcf6920000

[+] export PyUnicode_AsUTF8 => 0x7ffcf6ac3c58

[+] export PyObject_Repr => 0x7ffcf6998af8

[+] export PyObject_Str => 0x7ffcf6998968

[-] export PyLong_Check not found

[+] export PyLong_AsLongLong => 0x7ffcf69e17d4

[+] export PyLong_AsUnsignedLongLongMask => 0x7ffcf6a9b880

[+] export PyLong_FromString => 0x7ffcf69e3d60

[+] export PyLong_FromUnicodeObject => 0x7ffcf69e35a0

[+] export PyNumber_Add => 0x7ffcf699fd9c

[+] export PyNumber_InPlaceAdd => 0x7ffcf6a76230

[+] export PyNumber_Subtract => 0x7ffcf6951bc8

[+] export PyNumber_Multiply => 0x7ffcf69e6808

[+] export PyNumber_And => 0x7ffcf6982ba8

[+] export PyNumber_Or => 0x7ffcf6951b10

[+] export PyNumber_Xor => 0x7ffcf6951ab4
```
