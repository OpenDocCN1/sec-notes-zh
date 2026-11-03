
# `NoteActivity$EncryptAndSendTask` 逆向分析

## 类代码

```java
package site.qifen.note.ui;

import android.os.AsyncTask;
import android.os.Build;
import android.util.Log;
import dalvik.system.InMemoryDexClassLoader;
import java.lang.reflect.Method;
import java.nio.ByteBuffer;
import java.text.SimpleDateFormat;
import java.util.Date;
import site.qifen.note.model.Note;
import site.qifen.note.model.sendRequest;
import site.qifen.note.util.NoteUtil;

class NoteActivity$EncryptAndSendTask extends AsyncTask {
    final NoteActivity this$0;

    private NoteActivity$EncryptAndSendTask(NoteActivity noteActivity) {
        this.this$0 = noteActivity;
    }

    @Override
    public String doInBackground(String... params) {
        String contentText = params[0];
        try {
            byte[] dexData = this.this$0.loadData("Sex.jpg");
            ByteBuffer dexBuffer = ByteBuffer.wrap(dexData);
            InMemoryDexClassLoader classLoader = null;
            if (Build.VERSION.SDK_INT >= 26) {
                classLoader = new InMemoryDexClassLoader(dexBuffer, this.this$0.getClassLoader());
            }
            Class checkerClass = classLoader.loadClass("site.qifen.note.ui.Encrypto");
            Method checkMethod = checkerClass.getMethod("encrypt", String.class);
            this.this$0.contentText_back = contentText;
            String cipher = (String) checkMethod.invoke(checkerClass.getDeclaredConstructor(new Class[0]).newInstance(new Object[0]), this.this$0.sendInit(contentText));
            String response = sendRequest.sendPost("http://yuanshen.com/", "data=" + cipher);
            Log.d("JNITest", "Server Response: " + response);
            return cipher;
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    @Override
    public void onPostExecute(String cipher) {
        if (cipher != null) {
            String titleText = this.this$0.noteWriteTitleEdit.getText().toString();
            String tagText = this.this$0.noteWriteTagEdit.getText().toString();
            String date = new SimpleDateFormat("yyyy-MM-dd HH:mm").format(new Date());
            if (NoteActivity.access$100(this.this$0) == null) {
                NoteActivity.access$200(this.this$0).insertNote(new Note(tagText, titleText, this.this$0.contentText_back, date, false));
                NoteUtil.toast("保存成功");
                this.this$0.finish();
                return;
            }
            NoteActivity.access$100(this.this$0).setTitle(titleText);
            NoteActivity.access$100(this.this$0).setContent(this.this$0.contentText_back);
            NoteActivity.access$100(this.this$0).setDate(date);
            NoteActivity.access$100(this.this$0).setTag(this.this$0.contentText_back);
            NoteActivity.access$200(this.this$0).updateNote(NoteActivity.access$100(this.this$0));
            NoteUtil.toast("修改成功");
            this.this$0.finish();
            return;
        }
        NoteUtil.toast("加密失败");
    }
}
```

## 初步分析

我们可以发现，就是 `doInBackground` 函数调用了 `loadData`，并且还使用了 `classLoader` 去加载我们刚刚得到得到的类。

然后我们发现 `cipher` 就是加密后的数据，它是由我们输入的文本经 `sendInit` 处理，然后再被刚刚加载的类加密得来的。

我们跳转去看看 `sendInit`。

### 查看 `sendInit`

[//cdn.nlark.com/yuque/0/2024/png/40436415/1721483625155-5cf54301-9a65-4312-a927-7f68c86a218f.png](https://cdn.nlark.com/yuque/0/2024/png/40436415/1721483625155-5cf54301-9a65-4312-a927-7f68c86a218f.png:https)

原来就在刚刚的 `NoteActivity` 里面。

这里 `native` 说明我们要去分析 `/lib` 里面的 `so` 文件了。

### Hook 验证

当然，我们可以先 Hook 一下，验证一下。

[//cdn.nlark.com/yuque/0/2024/png/40436415/1721483781125-596755fa-9b9d-4385-90a8-1d3858ef0504.png](https://cdn.nlark.com/yuque/0/2024/png/40436415/1721483781125-596755fa-9b9d-4385-90a8-1d3858ef0504.png:https)

参数 1，返回 49.000000。没问题，就是他。

### IDA 分析 `so`

我们用 `ida64` 打开 `so`。

我们直接找带 `sendInit` 的。

```cpp
v20 = *(_QWORD *)(_ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2)) + 40);

StringUTFChars = _JNIEnv::GetStringUTFChars(a1, a3, 0LL);

sub_15994(v19, StringUTFChars);

_JNIEnv::ReleaseStringUTFChars(a1, a3, StringUTFChars);

v8 = sub_15A40(v19);

v7 = sub_15AB4(v19);

std::vector::vector>(v18, v8, v7);

encrypt(v18);

sub_15C34(v16);

v15 = sub_15C74(v17);

v14 = sub_15CB4(v17);

while ( (sub_15CF0(&v15, &v14) & 1) != 0 )

{

    v3 = (std::__ndk1 *)sub_15D38(&v15);

    std::to_string(v3, *(double *)v3);

    sub_15D50(v12, ",");

    sub_15D98(v16, v13);

    std::string::~string(v13);

    std::string::~string(v12);

    sub_15E34(&v15);

}

if ( (sub_15E5C(v16) & 1) == 0 )

    sub_15EA0(v16);

v6 = (char *)sub_15FD4(v16);

v5 = _JNIEnv::NewStringUTF(a1, v6);

std::string::~string(v16);

sub_15668(v17);

sub_15FF8(v18);

std::string::~string(v19);

_ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2));

return v5;

}
```

### `encrypt` 与 DCT

我刚看到这个伪代码的时候确实是崩溃的，因为我伪代码分析能力太弱了，压根看不懂，丢给 chatgpt 都没救；但看到都有 2 个解了，就一个一个点进去看。到 `encrypt(v18);` 这个其实满显眼的函数时

```cpp
  v12[1] = *(_QWORD *)(_ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2)) + 40);
  v10 = sub_15548(a1);
  v12[0] = 0LL;
  result = (double *)std::vector::vector(a2, v10, v12);
  for ( i = 0; i 
```

直觉告诉我，就是这个了。丢给 chatgpt，分析是什么离散啥啥的，简称 DCT；又问有没有逆向的，chatgpt 给出了 IDCT 的 python 脚本。

### IDCT 解密脚本

```python
import numpy as np

def idct(dct_data):

    N = len(dct_data)

    result = np.zeros(N)

    for n in range(N):

        sum_value = 0.0

        for k in range(N):

            cos_term = np.cos((k * 3.14159265 * (n + 0.5)) / N)

            if k == 0:

                sum_value += dct_data[k] * np.sqrt(1.0 / N) * cos_term

            else:

                sum_value += dct_data[k] * np.sqrt(2.0 / N) * cos_term

        result[n] = sum_value

    return result

def decrypt_to_ascii(dct_data):

    # 执行IDCT解密

    decrypted_data = idct(dct_data)

    # 四舍五入并转化为整数

    int_data = np.rint(decrypted_data).astype(int)

    # 转换为ASCII字符

    char_data = [chr(num) for num in int_data]

    return ''.join(char_data)

# 使用 DCT 变换后的数据

encrypted_data = [458.853181,-18.325492,-18.251911,-2.097520,-21.198660,-22.304648,21.103162,-5.786284,-15.248906,15.329286,16.919499,-19.669045,30.928253,-37.588034,-16.593954,-5.505211,3.014744,6.553616,31.131491,16.472500,6.802400,-78.278577,15.280099,3.893073,56.493581,-34.576344,30.146729,4.445671,6.732204]

# 解密并转换为ASCII字符

decrypted_message = decrypt_to_ascii(encrypted_data)

# 打印解密后的消息

print("Decrypted message:", decrypted_message)
```

运行脚本就会得到 `flag`。
