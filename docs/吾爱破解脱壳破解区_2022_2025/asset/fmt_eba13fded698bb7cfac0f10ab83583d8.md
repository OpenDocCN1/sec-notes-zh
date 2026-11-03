
# 修复 `classes.dex` 并生成 `2.dex`

## 核心修复代码

```java
public static File fix(ByteBuffer byteBuffer0, int v, int v1, int v2, Context context0) throws Exception {
    try {
        //File file0 = context0.getDir("data", 0);
        File file0 = new File("/sdcard/Download");
        int v3 = (int)(((Integer)D.[i]getClassDefData(byteBuffer0, v).get("class_data_off")));
        HashMap hashMap0 = D.[i]getClassData(byteBuffer0, v3);
        ((int[][])hashMap0.get("direct_methods"))[v1][2] = v2;
        byte[] arr_b = D.[i]encodeClassData(hashMap0);
        byteBuffer0.position(v3);
        byteBuffer0.put(arr_b);
        byteBuffer0.position(0x20);
        byte[] arr_b1 = new byte[byteBuffer0.capacity() - 0x20];
        byteBuffer0.get(arr_b1);
        byte[] arr_b2 = Utils.[i]getSha1(arr_b1);
        byteBuffer0.position(12);
        byteBuffer0.put(arr_b2);
        int v4 = Utils.[i]checksum(byteBuffer0);
        byteBuffer0.position(8);
        byteBuffer0.putInt(Integer.[i]reverseBytes(v4));
        byte[] arr_b3 = byteBuffer0.array();
        File file1 = new File(file0, "2.dex");
        FileOutputStream fileOutputStream0 = new FileOutputStream(file1);
        fileOutputStream0.write(arr_b3);
        fileOutputStream0.close();
        return file1;
    }
    catch(Exception exception0) {
        exception0.printStackTrace();
        return null;
    }
}

public static ByteBuffer read(String filename) {
    try {
        //File file0 = new File(context0.getDir("data", 0), "decode.dex");
        File file0 = new File("/sdcard/Download",filename);
        if(!file0.exists()) {
            Log.[i]e([i]TAG,file0.getAbsolutePath().toString()+" does not exit");
            return null;
        }

        FileInputStream fileInputStream0 = new FileInputStream(file0);
        byte[] arr_b = new byte[fileInputStream0.available()];
        fileInputStream0.read(arr_b);
        ByteBuffer byteBuffer0 = ByteBuffer.[i]wrap(arr_b);
        fileInputStream0.close();
        return byteBuffer0;
    }
    catch(Exception unused_ex) {
        Log.[i]e([i]TAG,unused_ex.toString());
        return null;
    }
}
```

## 调用修复方法

```java
// 这个是调用上面的方法
binding.fab.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Snackbar.[i]make(view, "Replace with your own action", Snackbar.[i]LENGTH_LONG)
                .setAnchorView(R.id.[i]fab)
                .setAction("Action", null).show();

        try {
            //C.fix(C.read("classes.dex"),0,3,7908,MainActivity.this);
            C.[i]fix(C.[i]read("classes.dex"),1,1,8108,MainActivity.this);
        } catch (Exception e) {
            Log.[i]e("zj",e.toString());
        }
    }
});
```

## 上传与拉取 `classes.dex`

我的代码主要思路就是，将 `classes.dex` 传到 `Dowload` 文件夹中，然后执行修复，然后把 `2.dex` 给拉回来反向分析。

上传 `classes.dex` 可以用工具，也可以用命令：

```bash
adb push classes.dex /sdcard/Download
```

拉取的命令为：

```bash
adb pull /sdcard/Download/2.dex
```

## 运行注意事项与 `md5()` 修复

运行上面的代码段，需要注意的问题是，首先测试的 App 需要有存储的权限，注意现在的 Android 版本都需要动态获取权限才行的。

还有就是需要用到反编译出来的 `D.java` 和 `Utils.java`，其中 `Utils.java` 有一个 `md5()` 函数有问题，需要手动修复一下：

```java
public static String md5(byte[] arr_b) {
    int v;
    try {
        String s = new BigInteger(1, MessageDigest.[i]getInstance("md5").digest(arr_b)).toString(16);
        v = 0;
        while(true) {
            if(v >= 0x20 - s.length()) {
                return s;
            }
            s = "0" + s;
            ++v;
        }
    }
    catch(NoSuchAlgorithmException unused_ex) {
        throw new RuntimeException("ops!!");
    }
}
```

## 结果与再分析

这回子玄天帝应该就醒了吧，可但是看一下结果

![](https://attach.52pojie.cn/forum/202403/02/214826pt6c7dwyzjpijj0t.png)

`image.png` (77.65 KB, 下载次数: 1)

[forum.php?mod=attachment&aid=MjY3OTM3OXwwYTY5NmVmY3wxNzkzNTcyNjAxfDE2ODA4NnwxODk1NTU5&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3OTM3OXwwYTY5NmVmY3wxNzkzNTcyNjAxfDE2ODA4NnwxODk1NTU5&nothumb=yes)

2024-3-2 21:48 上传

再一次哭了，看来我不是荒，我可能真的没有大道机缘，55555

试了好几次，依旧如此，莫说大道机缘，咋给点小道消息也不往我写了一天的代码啊，55555

冷静，一定要冷静，看了一眼 `smali` 是对的，那就说明只是缘份还没到，那就再等等吧

话说，又过了一天，睡了一觉，继续寻找我的机缘。

### 结合 `smali` 查看

其实结合 `smali` 来仔细琢磨一下玄帝给的机缘，看上去好好像是已经直接算好了。

还是玄帝对我好啊，怕我不会算，直接帮我算出来了，还好时机已过，要不我非得拿这个提交上去，然后结果不对，找管理理论{:1_918:}

既然玄帝如此不靠谱，咱们还是自己来找方案吧。

![](https://attach.52pojie.cn/forum/202403/03/090115z6ofmevbrfu0fjrz.png)

`image.png` (103.27 KB, 下载次数: 1)

[forum.php?mod=attachment&aid=MjY3OTQwMnw1MWU4Mjk4ZXwxNzkzNTcyNjAxfDE2ODA4NnwxODk1NTU5&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3OTQwMnw1MWU4Mjk4ZXwxNzkzNTcyNjAxfDE2ODA4NnwxODk1NTU5&nothumb=yes)

2024-3-3 09:01 上传

在这个地方，右键，选择第二项，点进去

![](https://attach.52pojie.cn/forum/202403/03/090214q8t4i808jjif4luf.png)

`image.png` (16.69 KB, 下载次数: 3)

[forum.php?mod=attachment&aid=MjY3OTQwM3w5YjllMTBhMHwxNzkzNTcyNjAxfDE2ODA4NnwxODk1NTU5&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3OTQwM3w5YjllMTBhMHwxNzkzNTcyNjAxfDE2ODA4NnwxODk1NTU5&nothumb=yes)

2024-3-3 09:02 上传

### 关闭自动解密后反编译

把这个改成 `disable`，这个选择就是实现自动解密的功能，这里我们需要关掉它，避免影响机缘大道

然后再进行反编译，这时候真实的大道机缘就出现啦

![](https://attach.52pojie.cn/forum/202403/03/090436jhv4zqv84hvglyoh.png)

`image.png` (12.51 KB, 下载次数: 4)

[forum.php?mod=attachment&aid=MjY3OTQwNHw2ZmM0ZDRkNHwxNzkzNTcyNjAxfDE2ODA4NnwxODk1NTU5&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3OTQwNHw2ZmM0ZDRkNHwxNzkzNTcyNjAxfDE2ODA4NnwxODk1NTU5&nothumb=yes)

2024-3-3 09:04 上传

## 结语

所以，机关算计太聪明，反害了卿卿性命。谢了，玄天帝！
