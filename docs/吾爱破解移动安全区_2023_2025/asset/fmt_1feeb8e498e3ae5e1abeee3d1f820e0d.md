
```
dex::ClassDataField staticFields[static_fields_size];
read += DexFileUtils::readFields(class_data + read, staticFields, static_fields_size);

dex::ClassDataField instanceFields[instance_fields_size];
read += DexFileUtils::readFields(class_data + read, instanceFields, instance_fields_size);

dex::ClassDataMethod directMethods[direct_methods_size];
read += DexFileUtils::readMethods(class_data + read, directMethods, direct_methods_size);

dex::ClassDataMethod virtualMethods[virtual_methods_size];
read += DexFileUtils::readMethods(class_data + read, virtualMethods, virtual_methods_size);
```

获取类中所有字段和方法的详细信息，为后续修补做准备

![](https://attach.52pojie.cn/forum/202511/04/120219bl6vi5vfnn8u7yql.png)

这里就将之前读取到的所有的方法都传入`patchMethod`中来修改。

![](https://attach.52pojie.cn/forum/202511/04/120219hrb8tj18r9vri1vi.png)

然后就是`patchMethod`了，这里主要是利用了之前维护好的`dexMap`，修改对应内存段权限后在Map查找`CodeItem`，然后使用`memcopy`填入。

这样的流程就完成了类的动态回填。

## 总结

`dpt-shell`上有很多值得学习的技术和加固原理，一次非常充实的学习过程
