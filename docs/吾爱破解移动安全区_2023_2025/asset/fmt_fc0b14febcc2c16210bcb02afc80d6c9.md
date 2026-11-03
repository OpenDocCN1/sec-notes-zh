
```
            app = data.info.makeApplication(data.restrictedBackupMode, null);

             // demoli-add start   
            String curPkgName = data.appInfo.packageName;
            String path_for_gadget = "/sdcard/" + curPkgName+"demoligadgethook";
            File path_curPkgName = new File(path_for_gadget);
            if(path_curPkgName.exists()){
                String arch = System.getProperty("os.arch");
                try{
                    if (arch != null && arch.contains("64")){
                        System.load("/system/lib64/demoli.so");
                    }else{
                        System.load("/system/lib/demoli.so");
                    }
                    Slog.e("demoli", "load gadget successful");
                }
                catch (Exception e) {
                    Slog.e("demoli", "demoli_failed=" + e.toString());
                }
            }
            // demoli-add end

            // demoli-add start
            //String curPkgName = app.getPackageName();
            String path_for_dex = "/sdcard/"+curPkgName+"demolidexhook.dex";
            File file = new File(path_for_dex);
            if(file.exists()){
                try {
                    //获取到 BaseDexClassLoader 的  pathList字段
                    // private final DexPathList pathList;
                    // find("BaseDexClassLoader","")
                    Field pathListField = BaseDexClassLoader.class.getDeclaredField("pathList");
                    //破坏封装，设置为可以调用
                    pathListField.setAccessible(true);
                    //拿到当前ClassLoader的pathList对象
                    Object pathListObj = pathListField.get(app.getClassLoader());
                    //获取当前ClassLoader的pathList对象的字节码文件（DexPathList ）
                    Class dexPathListClass = pathListObj.getClass();
                    //拿到DexPathList 的 dexElements字段
                    // private final Element[] dexElements；
                    Field dexElementsField = dexPathListClass.getDeclaredField("dexElements");
                    //破坏封装，设置为可以调用
                    dexElementsField.setAccessible(true);
                    //使用插件创建 ClassLoader
                    DexClassLoader pathClassLoader = new DexClassLoader(file.getPath(), file.getAbsolutePath(), null, app.getClassLoader());
                    //拿到插件的DexClassLoader 的 pathList对象
                    Object newPathListObj = pathListField.get(pathClassLoader);
                    //拿到插件的pathList对象的 dexElements变量
                    Object newDexElementsObj = dexElementsField.get(newPathListObj);
                    //拿到当前的pathList对象的 dexElements变量
                    Object dexElementsObj=dexElementsField.get(pathListObj);
                    int oldLength = Array.getLength(dexElementsObj);
                    int newLength = Array.getLength(newDexElementsObj);
                    //创建一个dexElements对象
                    Object concatDexElementsObject = Array.newInstance(dexElementsObj.getClass().getComponentType(), oldLength + newLength);
                    //先添加新的dex添加到dexElement
                    for (int i = 0; i  dexPathListClass2 = pathListObj2.getClass();
                    //拿到DexPathList 的 dexElements字段
                    // private final Element[] dexElements；
                    Field dexElementsField2 = dexPathListClass2.getDeclaredField("dexElements");
                    //破坏封装，设置为可以调用
                    dexElementsField2.setAccessible(true);
                    //使用插件创建 ClassLoader
                    DexClassLoader pathClassLoader2 = new DexClassLoader(file_apk.getPath(), file_apk.getAbsolutePath(), null, app.getClassLoader());
                    //拿到插件的DexClassLoader 的 pathList对象
                    Object newpathListObj2 = pathListField2.get(pathClassLoader2);
                    //拿到插件的pathList对象的 dexElements变量
                    Object newDexElementsObj2 = dexElementsField2.get(newpathListObj2);
                    //将插件的 dexElements对象设置给 当前ClassLoader的pathList对象
                    dexElementsField2.set(pathListObj2, newDexElementsObj2);
                    Slog.e("dexhook","demoli add apk successful!");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }else{
                Slog.e("apkhook",path_for_apk+" not exists");
            }
            // demoli add end
```
## 六、通用改机方案
- 加载这个文件
- 在 `/system/core/init/property_service.cpp` 中
然后需要把修改后的 `mybuild.prop` 加入到编译链中。安卓8是如下的方案，但是安卓10的修改路径不一样。需要放置在最后
- 自己创建这个文件
![](https://img-pool-own.oss-cn-shanghai.aliyuncs.com/img/image-20230321123933111.png)
## 1、自定义 su
![](https://img-pool-own.oss-cn-shanghai.aliyuncs.com/img/image-20230321161818474.png)
![](https://img-pool-own.oss-cn-shanghai.aliyuncs.com/img/image-20230321162039374.png)
![](https://img-pool-own.oss-cn-shanghai.aliyuncs.com/img/image-20230321162243126.png)
![](https://img-pool-own.oss-cn-shanghai.aliyuncs.com/img/image-20230321164001899.png)
![](https://img-pool-own.oss-cn-shanghai.aliyuncs.com/img/image-20230321164121706.png)
电脑太垃圾了 等我编译出来在发个效果
