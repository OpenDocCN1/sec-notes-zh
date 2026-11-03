
# [Android 原创] 记录一次Unity加固的探索与实现

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=2069859
- 抓取时间：2026-11-01T13:57:18.586Z

## 前言

正值某比赛出题，一道困难题不知道要怎么出才好，突然想起了`il2cpp`在安卓平台的加密，但是本人又不太会这方面，只好从学习一下`il2cpp`的原理并且尝试进行加固，本文记录我的出题过程。

## 整体启动与`libil2cpp.so`被载入的流程

要直到如何保护`libil2cpp.so`首先需要知道这个so文件是在什么时候被载入的，根据`il2cpp`安卓端的启动流程，我们可以发现载入位置，其流程图如下：

![](https://blog.shangwendada.top/wp-content/uploads/2025/10/image-1761470166103.png)

在此处我们可以看到`libunity.so`通过`dlopen`加载`libil2cpp.so`。

## `global-metadata.dat`的加载时机与分支流程

光保护`il2cpp.so`大抵是不够的，很多加固方案肯定都会选择加密`global-metadata.dat`，接下来我们看看`il2cpp.so`中负责加载`global-metadata.dat`的代码位置吧，如下是加载metadata的流程图

![](https://blog.shangwendada.top/wp-content/uploads/2025/10/image-1761470914355.png)

源码分析可知加载`global-metadata.dat`的代码位置在`E:\Unity Edit\2020.3.48f1c1\Editor\Data\il2cpp\libil2cpp\vm\MetadataLoader.cpp`。

具体代码如下：

```cpp
os::FileHandle* handle = os::File::Open(resourceFilePath, kFileModeOpen, kFileAccessRead, kFileShareRead, kFileOptionsNone, &error);
if (error != 0)
{
    utils::Logging::Write("ERROR: Could not open %s", resourceFilePath.c_str());
    return NULL;
}
void* fileBuffer = utils::MemoryMappedFile::Map(handle);
```

3> 加固`global-metadata.dat`

4> 一些小问题

正如上文所提到的，加固`global-metadata.dat`主要是在`E:\Unity Edit\2020.3.48f1c1\Editor\Data\il2cpp\libil2cpp\vm\MetadataLoader.cpp`，理论上修改了此处的代码之后，再写一个脚本去加密metadata再打包回去，就可以运行了，但是如果直接修改`MetadataLoader.cpp`会导致后面如果不需要加固的项目每一次编译都需要加密`global-metadata.dat`才能运行，这样的话岂不是非常的不方便

4> 项目构建时自动加固`global-metadata.dat`

自然，这里先说一下一个可能的解决方案，也是我在NSSCTF 4th中出过的一个pyinstaller打包项目加固的原理，我们可以通过设置一个标记，比如`MHY0`，我们再魔改`MetadataLoader`的时候通过识别是否存在`MHY0`这个标识符来确定是否需要解密，这样就不会影响后续打包的项目直接运行。

当然，还有更加优秀的办法，我们看到如下GitHub项目：

[`//github.com/badApple001/Il2cppEncrtypt`](https://github.com/badApple001/Il2cppEncrtypt:https)

我们在Unity Hub编写完主体代码之后就可以开始考虑加固了，接下来讲述一下`Il2cppEncrtypt`这个项目构建时加固的原理。

Unity有一个很有意思的机制叫做Editor Scripting（Unity 编辑器扩展系统）暨所有放在`Assets/Editor/`或任何以`Editor`命名的文件夹里的脚本，都会被编译进一个编辑器专用的程序集（Editor Assembly），不会进入打包的游戏。

与此同时，Unity还存在`IPostprocessBuildWithReport`这个接口有什么用呢，来看一下介绍

`IPostprocessBuildWithReport`：Unity 的构建管线接口，`OnPostprocessBuild`会在构建完成后自动被调用，参数是`BuildReport`，包含构建结果、输出路径、平台等信息。

（上面的字那么多看的怪枯燥的吧，让GPT生成了一张图，润色一下，看个乐呵

![](https://blog.shangwendada.top/wp-content/uploads/2025/10/image-1762164880718.png)

![](https://blog.shangwendada.top/wp-content/uploads/2025/10/image-1762164880718.png)

同时在这个接口的上下文中，我们可以获取到打包的路径，从而进行对`global-metadata.dat`的加固

那么其实我们就可以扩展这个接口，并且重写`OnPostprocessBuild`，就可以实现在构建项目的时候一并完成对`global-metadata.dat`加固了。

以下代码摘自`Il2cppEncrtypt`：

```csharp
void IPostprocessBuildWithReport.OnPostprocessBuild( BuildReport report )
{
    SetDisplayLog( LogMessageFromCpp );
    if ( report.summary.platform == UnityEditor.BuildTarget.Android )
    {
        Debug.Log(report.summary.outputPath);
        EncryptionCode( Marshal.StringToHGlobalAnsi( report.summary.outputPath ) );
        OverrideLoader( Marshal.StringToHGlobalAnsi( report.summary.outputPath ) );
    }
    else if ( report.summary.platform == UnityEditor.BuildTarget.iOS )
    {
    }
    Debug.Log( "执行扩展程序完成" );
}
```

4> 导出项目并覆盖项目内`global-metadata.dat`，编译魔改的`il2cpp.so`

诚然，如果我们直接使用Unity Hub编译一个可以直接运行的unity app的话默认使用的是Unity Editor中的代码，这个十分坑爹，也许是我不知道如何修改，反正最后试了很久也是没招了。

导出Unity 项目之后通过Android Studio直接编译：

导出项目在装好安卓的SDK之后，Build Setting界面，直接就可以看到导出了。

![](https://blog.shangwendada.top/wp-content/uploads/2025/10/image-1762166351180.png)

导出之后在导出目录下可以看到两个包`launcher`和`unityLibrary`，其中`unityLibrary`主要加载`il2cpp`的内容

![](https://blog.shangwendada.top/wp-content/uploads/2025/10/image-1762167564566.png)

同时我们检查一下`Metadata`是否加密

其位置在于`unityLibrary\src\main\assets\bin\Data\Managed\Metadata`

![](https://blog.shangwendada.top/wp-content/uploads/2025/10/image-1762182389917.png)

可以看到这里原本应该是有意义的symbol，但此刻变成了乱码，即`Metadata`成功被加密

接下来我们需要在项目中的loader写入解密代码，但你会发现Android studio 好像无法完美识别到关于`il2cpp.so`的代码，我们需要手动在`unityLibrary\src\main\Il2CppOutputProject\IL2CPP\libil2cpp\vm\MetadataLoader.cpp`中修改，这里需要注意的是确保和我们加密代码一致，不然会导致崩溃。

如下代码中所示，主要的点也就是在拿到`fileBuffer`之后做解密即可。

```cpp
void *il2cpp::vm::MetadataLoader::LoadMetadataFile(const char *fileName)
{
#if IL2CPP_TARGET_ANDROID && IL2CPP_TINY_DEBUGGER && !IL2CPP_TINY_FROM_IL2CPP_BUILDER
    std::string resourcesDirectory = utils::PathUtils::Combine(utils::StringView("Data"), utils::StringView("Metadata"));
    std::string resourceFilePath = utils::PathUtils::Combine(resourcesDirectory, utils::StringView(fileName, strlen(fileName)));
    int size = 0;
    return loadAsset(resourceFilePath.c_str(), &size, malloc);
#elif IL2CPP_TARGET_JAVASCRIPT && IL2CPP_TINY_DEBUGGER && !IL2CPP_TINY_FROM_IL2CPP_BUILDER
    return g_MetadataForWebTinyDebugger;
#else
    std::string resourcesDirectory = utils::PathUtils::Combine(utils::Runtime::GetDataDir(), utils::StringView("Metadata"));
    std::string resourceFilePath = utils::PathUtils::Combine(resourcesDirectory, utils::StringView(fileName, strlen(fileName)));
    int error = 0;
    os::FileHandle *handle = os::File::Open(resourceFilePath, kFileModeOpen, kFileAccessRead, kFileShareRead, kFileOptionsNone, &error);
    if (error != 0)
    {
        utils::Logging::Write("ERROR: Could not open %s", resourceFilePath.c_str());
        return NULL;
    }
    void *fileBuffer = g_cacheFileHeader = utils::MemoryMappedFile::Map(handle);
    int ero;
    int64_t length = os::File::GetLength(handle, &ero);
    void *decBuffer = g_cacheDecodeHeader = PromiseAntiencryption(fileBuffer, length);
    os::File::Close(handle, &error);
    if (error != 0)
    {
        utils::MemoryMappedFile::Unmap(fileBuffer);
        fileBuffer = NULL;
        return NULL;
    }
    return decBuffer;
#endif
}
```

至此成功的完成了`Metadata`的加密，但是这够吗，显然远远不够，我们准备开始下一步探索，加密`libil2cpp.so`

3> 加固`il2cpp.so`

4> 一些小小问题

在上文中提到了，`libil2cpp.so`是被`libunity.so`加载的，但是我们在`libUnity`的编译脚本中可以看见似乎`build.gradle`是没有编译`libunity.so`的逻辑的，代码如下

```groovy
def BuildIl2Cpp(String workingDir, String targetDirectory, String architecture, String abi, String configuration) {
    exec {
        commandLine(workingDir + "/src/main/Il2CppOutputProject/IL2CPP/build/deploy/netcoreapp3.1/il2cpp.exe",
            "--compile-cpp",
            "--incremental-g-c-time-slice=3",
            "--avoid-dynamic-library-copy",
            "--profiler-report",
            "--libil2cpp-static",
            "--platform=Android",
            "--architecture=" + architecture,
            "--configuration=" + configuration,
            "--outputpath=" + workingDir + targetDirectory + abi + "/libil2cpp.so",
            "--cachedirectory=" + workingDir + "/build/il2cpp_"+ abi + "_" + configuration + "/il2cpp_cache",
            "--additional-include-directories=" + workingDir + "/src/main/Il2CppOutputProject/IL2CPP/external/bdwgc/include",
            "--additional-include-directories=" + workingDir + "/src/main/Il2CppOutputProject/IL2CPP/libil2cpp/include",
            "--tool-chain-path=" + android.ndkDirectory,
            "--map-file-parser=" + workingDir + "/src/main/Il2CppOutputProject/IL2CPP/MapFileParser/MapFileParser.exe",
            "--generatedcppdir=" + workingDir + "/src/main/Il2CppOutputProject/Source/il2cppOutput",
            "--baselib-directory=" + workingDir + "/src/main/jniStaticLibs/" + abi,
            "--dotnetprofile=unityaot")
        environment "ANDROID_SDK_ROOT", getSdkDir()
    }
    delete workingDir + targetDirectory + abi + "/libil2cpp.sym.so"
    ant.move(file: workingDir + targetDirectory + abi + "/libil2cpp.dbg.so", tofile: workingDir + "/symbols/" + abi + "/libil2cpp.so")
}
```

同时我们编译一次项目也会发现`JniLibs`的目录出现了时间差

![](https://blog.shangwendada.top/wp-content/uploads/2025/10/image-1762184857599.png)

种种迹象也说明了`libunity.so`似乎不再我们可控范围内，经过搜索得知这个`unity.so`是核心引擎，属于Unity的闭源部分，因此我们没有办法通过修改`unity.so`来拦截`il2cpp`的加载从而实现动态解密。

4> 通过Hook拦截`il2cpp.so`加载流程

既然无法从代码层面进行修改`ilbunity.so`，那么根据上文提到的`il2cpp.so`被加载流程，最后加载`libunity.so`肯定是要走`dlopen`的，那么是不是说我们只需要在这之前注册一个hook，但`dlopen`打开的是`libil2cpp`的时候我们进行加密呢，接下来我们开始尝试
