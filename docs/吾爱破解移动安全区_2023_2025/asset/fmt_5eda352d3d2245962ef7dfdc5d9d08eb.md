
```
oldImpl(handle,selector,args);

})

})
```
可以看到上述代码提前将方法地址保存在 `oldImpl` 然后使用 `ObjC.implement` 进行方法的替换，是否在内执行原方法可以自己去实现

## 6. 枚举内存中所有模块/符号/地址

```
frida-ps -Ua 查看运行的应用
objection -g UnCrackable1 explore objection注入应用进程
memory list modules 列举内存中加载的模块
```

![](https://attach.52pojie.cn/forum/202308/02/151447yok6ekblklo1zxkl.png)

18.png (25.16 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYzMjQwOHw4MDIyNDM1MnwxNzkzNTY5Mjc3fDE2ODA4NnwxODE0NTQ4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMjQwOHw4MDIyNDM1MnwxNzkzNTY5Mjc3fDE2ODA4NnwxODE0NTQ4&nothumb=yes)

2023-8-2 15:14 上传

`memory list exports UnCrackable\ Level\ 1` 列出模块中的导出符号

`memory list exports Foundation`

![](https://attach.52pojie.cn/forum/202308/02/151451ut9oejpnwntt0otz.png)

19.png (360.26 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYzMjQwOXxkYWRkMzQxZHwxNzkzNTY5Mjc3fDE2ODA4NnwxODE0NTQ4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMjQwOXxkYWRkMzQxZHwxNzkzNTY5Mjc3fDE2ODA4NnwxODE0NTQ4&nothumb=yes)

2023-8-2 15:14 上传

我们去看看 `objection` 是如何实现的枚举模块及枚举模块内导出符号，代码在 `agent-->generic-->memory.ts`

```
export const listModules = (): Module[] => {
  return Process.enumerateModules();
};

export const listExports = (name: string): ModuleExportDetails[] | null => {
  const mod: Module[] = Process.enumerateModules().filter((m) => m.name === name);
  if (mod.length 
```

可以看到其实现逻辑其实很简单，就是使用了 `frida` 的 `api` `Process.enumerateModules()` 得到模块对象数组，然后根据模块对象调用 `enumerateExports()` 得到导出符号，与安卓中 `c` 相关的符号地址枚举其实是一样的，自己实现代码如下：

```
function listExports(name) {
//①可以在枚举模块是保存模块的name属性作为参数然后作为参数给Module.enumerateExports(name)
// var exporttArr = Module.enumerateExports(name);
//②也可以在枚举模块是保存模块对象当作参数，然后用模块对象调用name.enumerateExports()
var exporttArr = name.enumerateExports();
// var exporttArr = name.enumerateSymbols();
console.log("枚举模块==》",name.name)
for(var i = 0;i",modules[i].name,"--address:",JSON.stringify(modules[i].base))
console.log("=======================================")
console.log("模块名称:",JSON.stringify(modules[i].name));
console.log("模块地址:",JSON.stringify(modules[i].base));
console.log("大小:",JSON.stringify(modules[i].size));
console.log("文件系统路径",JSON.stringify(modules[i].path));
}
return modules[0]
}
function main(){
var modulename = listModules();
console.log(modulename)
listExports(modulename)
}
setImmediate(main)
```

## 7. 无脑自动化hook 应用包下所有函数

hook应用包下所有函数，我们可以去网上搜索[//codeshare.frida.re/@interference-security/ios-app-all-classes-methods-hooks/" target="_blank" rel="noopener noreferrer nofollow"](https://codeshare.frida.re/@interference-security/ios-app-all-classes-methods-hooks/:rel="nofollow noopener" href="https)查看如何编写

```
function get_timestamp()
{
        var today = new Date();
        var timestamp = today.getFullYear() + '-' + (today.getMonth()+1) + '-' + today.getDate() + ' ' + today.getHours() + ":" + today.getMinutes() + ":" + today.getSeconds() + ":" + today.getMilliseconds();
        return timestamp;
}

function hook_class_method(class_name, method_name)
{
        var hook = eval('ObjC.classes.'+class_name+'["'+method_name+'"]');
                Interceptor.attach(hook.implementation, {
                        onEnter: function(args) {
                        console.log(" [" + get_timestamp() + " ] Detected call to: " + class_name + " -> " + method_name);
                }
        });
}

function run_hook_all_methods_of_classes_app_only()
{
        console.log(" Started: Hook all methods of all app only classes");
        var free = new NativeFunction(Module.findExportByName(null, 'free'), 'void', ['pointer'])
    var copyClassNamesForImage = new NativeFunction(Module.findExportByName(null, 'objc_copyClassNamesForImage'), 'pointer', ['pointer'])
    var p = Memory.alloc(Process.pointerSize)
    Memory.writeUInt(p, 0)
    var path = ObjC.classes.NSBundle.mainBundle().executablePath().UTF8String()
    var pPath = Memory.allocUtf8String(path)
    var pClasses = copyClassNamesForImage(pPath, p)
    var count = Memory.readUInt(p)
    var classesArray = new Array(count)
    for (var i = 0; i  " + funcName2);
                                }
                                catch(err)
                                {
                                        console.log(" [" + get_timestamp() + "] Hooking Error: " + err.message);
                                }
                        }
                }
    }
    free(pClasses)
        console.log(" Completed: Hook all methods of all app only classes");
}

function hook_all_methods_of_classes_app_only()
{
        setImmediate(run_hook_all_methods_of_classes_app_only)
}

hook_all_methods_of_classes_app_only()
```

可以看到该脚本主要实现逻辑大体如下：

①在 `run_hook_all_methods_of_classes_app_only` 方法中获取所有类，具体获取方法是使用 `copyClassNamesForImage` 这一 `Objective-C` 运行时中的一个私有 `API` 去 获取当前 `App` 中定义的所有类名，并用内存数组保存

②然后遍历类方法，并使用 `hasOwnProperty` 这一 `js` 的 `api` 判断类是否属于当前 `app`, 如果是就遍历类中方法

③使用 `hook_class_method` 方法进行每个方法的实际 `hook`

![](https://attach.52pojie.cn/forum/202308/02/151453xsvaj77h7zkuk77s.png)

20.png (69.19 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYzMjQxMHwxN2MwNWZmNnwxNzkzNTY5Mjc3fDE2ODA4NnwxODE0NTQ4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMjQxMHwxN2MwNWZmNnwxNzkzNTY5Mjc3fDE2ODA4NnwxODE0NTQ4&nothumb=yes)

2023-8-2 15:14 上传

## 8. Objection 内存漫游搜刮所有对象

先使用 `objection` 搜刮 `ViewController` 对象

```
ios heap search instances ViewController
```

![](https://attach.52pojie.cn/forum/202308/02/151456ly9ehv99qz3wfj39.png)

21.png (18.58 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYzMjQxMXw4OTZlYWU0MHwxNzkzNTY5Mjc3fDE2ODA4NnwxODE0NTQ4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMjQxMXw4OTZlYWU0MHwxNzkzNTY5Mjc3fDE2ODA4NnwxODE0NTQ4&nothumb=yes)

2023-8-2 15:14 上传

然后我们去 `objection` 代码中查看其具体实现，位置在 `agent-->src-->ios-->heap.ts`
