
```
    }

    public static void Encrypt(char[] arr) {

        int[] KEY;

        ArrayList Resultlist = new ArrayList<>();

        for (char c : arr) {

            Resultlist.add(Integer.valueOf((c + '@') ^ 32));    // valueOf() 方法用于返回给定参数的原生 Number 对象值，参数可以是原生数据类型, String等。

                                                                //此处返回一个int值

        }

        System.out.println(Resultlist);

        ArrayList KEYList = new ArrayList<>();

        for (int i : new int[]{180, 136, 137, 147, 191, 137, 147, 191, 148, 136, 133, 191, 134, 140, 129, 135, 191, 65}) {

            KEYList.add(Integer.valueOf(i));

        }

        System.out.println("Result:");

        if (Resultlist.equals(KEYList)) {

            System.out.println("Congratulations！");

        } else {

            System.err.println("Error！");

        }

    }

}
```

计算什么的好烦的，直接爆破

```
import java.util.ArrayList;

/* renamed from: reverse  reason: default package */
public class reverse {
    public static void main(String[] args) {
        int text[] = {180, 136, 137, 147, 191, 137, 147, 191, 148, 136, 133, 191, 134, 140, 129, 135, 191, 65};
        ArrayList flag = new ArrayList<>();

        for(int i=0; i
```

爆破得到的数组通过ascll转一下即可

需要注意点的是需要将全部的可见字符全部用于跑字典，不然就会出现问题

## 3> [GXYCTF2019]luck_few

`DIE` 分析后可以看到为 `elf` 64 位程序，先用 `ida64` 打开看是否可以分析出什么

发现有一个 `get_flag` 函数，那不出意外获取 `flag` 的逻辑就在这里了

```
unsigned __int64 get_flag()

{

  unsigned int v0; // eax

  int i; // [rsp+4h] [rbp-3Ch]

  int j; // [rsp+8h] [rbp-38h]

  __int64 s; // [rsp+10h] [rbp-30h] BYREF

  char v5; // [rsp+18h] [rbp-28h]

  unsigned __int64 v6; // [rsp+38h] [rbp-8h]

  v6 = __readfsqword(0x28u);

  v0 = time(0LL);

  srand(v0);

  for ( i = 0; i 
```

大致观察后可以发现是在 `case 4` 中给 `f2` 赋值，之后再 `case 5` 中对 `f2` 进行变换，最后在 `case 1` 中进行合并输出

写个脚本跑一些就可以得到结果

需要注意到是，`f2` 是存储在栈中的，所以写 `f2` 数组时应该倒着写，

即

`char arr[] = {0x69, 0x63, 0x75, 0x67, 0x60, 0x6f, 0x66, 0x7f, 0};`

得到的 `flag` 改成 `flag{}` 格式再提交...

## 3> findit

又一道安卓 apk 题目

`Android-packinfo` 查壳后没有发现什么信息，直接用 `jadx` 打开

找到主函数

```
package com.example.findit;

import android.os.Bundle;
import android.support.v7.app.ActionBarActivity;
import android.view.MenuItem;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;

public class MainActivity extends ActionBarActivity {
    /* access modifiers changed from: protected */
    [//www.52pojie.cn/home.php?mod=space&uid=1892347" target="_blank"](https://www.52pojie.cn/home.php?mod=space&uid=1892347:href="https) // android.support.v7.app.ActionBarActivity, android.support.v4.app.FragmentActivity
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        final EditText edit = (EditText) findViewById(R.id.widget2);
        final TextView text = (TextView) findViewById(R.id.widget1);
        final char[] a = {'T', 'h', 'i', 's', 'I', 's', 'T', 'h', 'e', 'F', 'l', 'a', 'g', 'H', 'o', 'm', 'e'};
        final char[] b = {'p', 'v', 'k', 'q', '{', 'm', '1', '6', '4', '6', '7', '5', '2', '6', '2', '0', '3', '3', 'l', '4', 'm', '4', '9', 'l', 'n', 'p', '7', 'p', '9', 'm', 'n', 'k', '2', '8', 'k', '7', '5', '}'};
        ((Button) findViewById(R.id.widget3)).setOnClickListener(new View.OnClickListener() {
            /* class com.example.findit.MainActivity.AnonymousClass1 */

            public void onClick(View v) {
                char[] x = new char[17];
                char[] y = new char[38];
                for (int i = 0; i = 'A') || (a[i] = 'a')) {
                        x[i] = (char) (a[i] + 18);
                    } else if ((a[i]  'Z') && (a[i]  'z')) {
                        x[i] = a[i];
                    } else {
                        x[i] = (char) (a[i] - '\b');
                    }
                }
                if (String.valueOf(x).equals(edit.getText().toString())) {
                    for (int i2 = 0; i2  'Z') && (b[i2]  'z')) {
                            y[i2] = b[i2];
                        } else {
                            y[i2] = (char) (b[i2] + 16);
                            if ((y[i2] > 'Z' && y[i2] = 'z') {
                                y[i2] = (char) (y[i2] - 26);
                            }
                        }
                    }
                    text.setText(String.valueOf(y));
                    return;
                }
                text.setText("答案错了肿么办。。。不给你又不好意思。。。哎呀好纠结啊~~~");
            }
        });
    }

    public boolean onOptionsItemSelected(MenuItem item) {
        if (item.getItemId() == R.id.action_settings) {
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
}
```

很明显的可以看到有个地方会输出答案错误，那他上面的函数不出意外就是 `flag` 的输出函数了

继续写个脚本跑就行，要注意的是如果使用 `c/c++` 写，数组需要定义为 `unsigned char`，不然会溢出造成显示出问题

```
#include

using namespace std;

char b[] = {'p', 'v', 'k', 'q', '{', 'm', '1', '6', '4', '6', '7', '5', '2', '6', '2', '0', '3', '3', 'l', '4', 'm', '4', '9', 'l', 'n', 'p', '7', 'p', '9', 'm', 'n', 'k', '2', '8', 'k', '7', '5', '}'};

unsigned char y[40]={0};

int main(int argc, char const *argv[])

{

    for (int i2 = 0; i2  'Z') && (b[i2]  'z') ) 

            y[i2] = b[i2];

        else

        {

            y[i2] = (char) (b[i2] + 16);

            if ((y[i2] > 'Z' && y[i2] = 'z') 

                y[i2] = (char) (y[i2] - 26);
        }

    }

    cout 
```

## 3> 简单注册器

还是安卓

主要逻辑在这里

```
public void onClick(View v) {
    int flag = 1;
    String xx = editview.getText().toString();
    if (!(xx.length() == 32 && xx.charAt(31) == 'a' && xx.charAt(1) == 'b' && (xx.charAt(0) + xx.charAt(2)) - 48 == 56)) {
        flag = 0;
    }
    if (flag == 1) {
        char[] x = "dd2940c04462b4dd7c450528835cca15".toCharArray();
        x[2] = (char) ((x[2] + x[3]) - 50);
        x[4] = (char) ((x[2] + x[5]) - 48);
        x[30] = (char) ((x[31] + x[9]) - 48);
        x[14] = (char) ((x[27] + x[28]) - 97);
        for (int i = 0; i 
```

直接跑就行，够清晰了，跑出来记得加一个 `flag` 框

## 3> [GWCTF 2019]pyre

`pyc` 文件，用在线工具逆向源码

```
#!/usr/bin/env python

# visit http://tool.lu/pyc/ for more information

print 'Welcome to Re World!'

print 'Your input1 is your flag~'

l = len(input1)

for i in range(l):

    num = ((input1[i] + i) % 128 + 128) % 128

    code += num

for i in range(l - 1):

    code[i] = code[i] ^ code[i + 1]

print code

code = [

    '\x1f',

    '\x12',

    '\x1d',

    '(',

    '0',

    '4',

    '\x01',

    '\x06',

    '\x14',

    '4',

    ',',

    '\x1b',

    'U',

    '?',

    'o',

    '6',

    '*',

    ':',

    '\x01',

    'D',

    ';',

    '%',

    '\x13']
```

根据题目意思写了一个 c 语言版本的，用来测试

```
#include
using namespace std;

string arr = "flag{abcabc}";
int brr[32];

int main(int argc, char const *argv[])
{
    int length = arr.length();

    for (int i = 0; i =0; i--)
        brr[i] = brr[i] ^ brr[i + 1];
    for (int i = 0; i
```

异或部分倒着写就行，前面的求余那一块直接爆破即可

```
#include

#include

using namespace std;

char code[] = { 0x1f, 0x12, 0x1d, '(', '0', '4', 0x01, 0x06, 0x14, '4', ',', 0x1b, 'U', '?', 'o', '6', '*', ':', 0x01, 'D', ';', '%', 0x13, 0};

// char code[] = {102, 109, 99, 106, 127, 102, 104, 106, 105, 107, 109, 8, 0};

// char code[] = {11, 14, 9, 21, 25, 14, 2, 3, 2, 6, 101, 8, 0};

char arr[100] = {0};

int main(int argc, char const *argv[])

{

    int l = strlen(code);

    for (int i = 0; i=0; i--)

        code[i] = code[i] ^ code[i + 1];

    for (int i = 0; i=32 && j
```

得到的 `flag` 记得换格式提交

## 3> [BJDCTF2020]JustRE

没啥说的，函数不多，翻一下就出来了
