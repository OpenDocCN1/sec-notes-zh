
![](https://attach.52pojie.cn/forum/202302/06/171750p1b66k3tnzpb7byt.png)

734b2f5e2903eb0d5073a6d27b91c63d_3f10d964f9c245459e99fc1e1372a48a.png (488.14 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjU4ODc2MnxlODE0ZjYwMHwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4ODc2MnxlODE0ZjYwMHwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes)

2023-2-6 17:17 上传

PS：分析时所写的笔记和代码已上传，可以从[//gitee.com/kbtxwer/happy_new_year_52pojie_2023_crack/tree/master/%E8%A7%A3%E9%A2%98%E9%A2%86%E7%BA%A2%E5%8C%85%E4%B9%8B%E4%BA%94%20%7BWindows%20%20%E4%B8%AD%E7%BA%A7%E9%A2%98%7D" target="_blank" rel="noopener noreferrer nofollow"](https://gitee.com/kbtxwer/happy_new_year_52pojie_2023_crack/tree/master/%E8%A7%A3%E9%A2%98%E9%A2%86%E7%BA%A2%E5%8C%85%E4%B9%8B%E4%BA%94%20%7BWindows%20%20%E4%B8%AD%E7%BA%A7%E9%A2%98%7D:rel="nofollow noopener" href="https://gitee.com)。

## 解题领红包之六 {Android 中级题}

这题的坑非常多，而且不像是crackme。

首先运行APP，并用`jadx`查看APP的`MainActivity`代码：

```
public final class MainActivity extends AppCompatActivity {
    public static final Companion Companion = new Companion(null);
    private TextView automedia;
    private ActivityMainBinding binding;
    private MediaRecorder mediaRecorder;
    private boolean permissionToRecordAccepted;
    private final int REQUEST_RECORD_AUDIO_PERMISSION = ItemTouchHelper.Callback.DEFAULT_DRAG_ANIMATION_DURATION;
    private String[] permissions = {"android.permission.RECORD_AUDIO"};

    public final native String decrypt(String str) throws IOException, IllegalBlockSizeException, InvalidKeyException, BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException;

    public final native String encrypt(String str) throws IOException, IllegalBlockSizeException, InvalidKeyException, BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException;

    public final native boolean get_RealKey(String str);

    /* JADX INFO: Access modifiers changed from: protected */
    @Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_main);
        View findViewById = findViewById(R.id.tv_audio);
        Intrinsics.checkNotNullExpressionValue(findViewById, "findViewById(R.id.tv_audio)");
        this.automedia = (TextView) findViewById;
        if (ContextCompat.checkSelfPermission(this, "android.permission.RECORD_AUDIO") != 0) {
            ActivityCompat.requestPermissions(this, this.permissions, this.REQUEST_RECORD_AUDIO_PERMISSION);
        } else {
            startRecording();
        }
    }

    private final void Check_Volume(double d) {
        TextView textView = this.automedia;
        if (textView == null) {
            Intrinsics.throwUninitializedPropertyAccessException("automedia");
            textView = null;
        }
        textView.setText("当前分贝:" + d);
        boolean z = false;
        if (84.0d  0) {
            return 20 * Math.log10(maxAmplitude);
        }
        return 0.0d;
    }

    /* JADX INFO: Access modifiers changed from: protected */
    @Override // androidx.appcompat.app.AppCompatActivity, androidx.fragment.app.FragmentActivity, android.app.Activity
    public void onDestroy() {
        super.onDestroy();
        MediaRecorder mediaRecorder = this.mediaRecorder;
        if (mediaRecorder == null) {
            Intrinsics.throwUninitializedPropertyAccessException("mediaRecorder");
            mediaRecorder = null;
        }
        mediaRecorder.release();
    }

    private final void xigou(Context context) {
        AssetFileDescriptor openFd = context.getAssets().openFd("xigou.mp3");
        Intrinsics.checkNotNullExpressionValue(openFd, "context.assets.openFd(fileName)");
        MediaPlayer mediaPlayer = new MediaPlayer();
        mediaPlayer.setDataSource(openFd.getFileDescriptor(), openFd.getStartOffset(), openFd.getLength());
        mediaPlayer.prepare();
        mediaPlayer.start();
    }

    /* compiled from: MainActivity.kt */
    @Metadata(d1 = {"\u0000\f\n\u0002\u0018\u0002\n\u0002\u0010\u0000\n\u0002\b\u0002\b\u0086\u0003\u0018\u00002\u00020\u0001B\u0007\b\u0002￠\u0006\u0002\u0010\u0002¨\u0006\u0003"}, d2 = {"Lcom/zj/wuaipojie2023_2/MainActivity$Companion;", "", "()V", "app_release"}, k = 1, mv = {1, 7, 1}, xi = 48)
    /* loaded from: classes.dex */
    public static final class Companion {
        public /* synthetic */ Companion(DefaultConstructorMarker defaultConstructorMarker) {
            this();
        }

        private Companion() {
        }
    }

    static {
        System.loadLibrary("52pj");
    }
}
```

运行过程就不截图了，总之主要逻辑就是先获取录音权限，然后计算你的音量，小于100分贝就播放`xigou.mp3`，在100~101分贝就将静态资源文件释放到数据目录，并提示你去找`flag`，大于101分贝则不做处理。

因此`Java`代码就没必要再理会了，这段代码最大的价值就是告诉我们`lib52pj.so`里有哪些函数。

运用社会工程学，得知app中没有签名校验，`so`里除了反调试外没有其他保护。

由于这个`so`在app中根本没被调用，我们不妨将`so`拖入`IDA`分析一下，再决定要做什么。

注意到`JNIOnload`方法里有个`ptrace`，这个是让`so`自己附加自己，从而导致调试器无法附加的代码：

![](https://attach.52pojie.cn/forum/202302/06/171816ok783rjak9j8i9g8.png)

7e7c9e6f6f3c880d9ea7e7f9fa368bf5_eb7d7a4e71ed439da67ce6be8a66a12a.png (74.26 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjU4ODc2M3xiNDBmY2UwNHwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4ODc2M3xiNDBmY2UwNHwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes)

2023-2-6 17:18 上传

按住`Ctrl + Alt + K`将反汇编中的调用改成`NOP`即可

![](https://attach.52pojie.cn/forum/202302/06/171827glc3dbvv7bgcht65.png)

d504e254996bdfd3f7eea5bf51406b05_f37ed9db8c354370938425e94ba37ba7.png (29.45 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjU4ODc2NHw0MTQwZTkyYXwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4ODc2NHw0MTQwZTkyYXwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes)

2023-2-6 17:18 上传

下面还有个`anti_debug`函数，将开头改成`RET`直接返回即可

![](https://attach.52pojie.cn/forum/202302/06/171839cbsbrtzaysysw64n.png)

fc95dea11106b9c46623d6c69897f518_7e24b8853d7949e0aa6ea0158055efb1.png (53.7 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjU4ODc2NXwyYzBlMGVkMnwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4ODc2NXwyYzBlMGVkMnwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes)

2023-2-6 17:18 上传

（以上修改可能不适用于`arm-v7`，如有问题请改用 `x86_64`或`arm-v8`再试）

接下来再看看这三个在`Java`中声明了的函数：

### `get_RealKey`

```
bool __fastcall get_RealKey(JNIEnv *a1, __int64 a2, void *a3)

{

  int8x16_t *v3; // x19

  int8x16_t v5; // [xsp+0h] [xbp-30h] BYREF

  char v6; // [xsp+10h] [xbp-20h]

  __int64 v7; // [xsp+18h] [xbp-18h]

  v7 = *(_QWORD *)(_ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2)) + 40);

  v3 = (int8x16_t *)(*a1)->GetStringUTFChars(a1, a3, 0LL);

  if ( strlen((const char *)v3) != 16 )

    return 0LL;

  v6 = 0;

  // xmmword_3030 = {0xFB, 0xFE, 0xFB, 0xFE, 0xFB, 0xFE, 0xFB, 0xFE, 0xFB, 0xFE, 0xFB, 0xFE, 0xFB, 0xFE, 0xFB, 0xFE}

  v5 = vaddq_s8(*v3, (int8x16_t)xmmword_3030);

  return strcmp((const char *)&v5, "thisiskey") != 0;

}
```

### `decrypt`

```
jstring __fastcall Java_com_zj_wuaipojie2023_12_MainActivity_decrypt(JNIEnv *a1, __int64 a2, void *a3)
{
  const char *v5; // x21
  const char *v6; // x22

  v5 = (*a1)->GetStringUTFChars(a1, a3, 0LL);
  v6 = (const char *)AES_ECB_PKCS7_Decrypt((int)v5, "|wfkuqokj4548366");
  (*a1)->ReleaseStringUTFChars(a1, a3, v5);
  return (*a1)->NewStringUTF(a1, v6);
}
```

### `encrypt`

```
jstring __fastcall Java_com_zj_wuaipojie2023_12_MainActivity_encrypt(JNIEnv *a1, __int64 a2, void *a3)

{

  const char *v5; // x21

  const char *v6; // x22

  v5 = (*a1)->GetStringUTFChars(a1, a3, 0LL);

  v6 = (const char *)AES_ECB_PKCS7_Encrypt((int)v5, "|wfkuqokj4548366");

  (*a1)->ReleaseStringUTFChars(a1, a3, v5);

  return (*a1)->NewStringUTF(a1, v6);

}
```

好了，关键方法都摆到这了。经测试得知写死在加解密方法的密钥是假的，使`get_RealKey`运算后的字符串等于`thisiskey`的输入构成的密钥也是假的，猜猜密钥在哪？

答案是将传入加解密方法假密钥放入`get_RealKey`里参与运算，求和之后的结果是真密钥！
