
```
package com.example.checkselinuxenabled;

import android.os.Bundle;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;
import java.lang.reflect.Method;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if(isSelinuxFlagInEnabled())
        {
            Toast.makeText(getApplicationContext(), "SELinux开启",
                    Toast.LENGTH_SHORT).show();
        }
        else
        {
            Toast.makeText(getApplicationContext(), "SELinux没有开启",
                    Toast.LENGTH_SHORT).show();
        }
    }

    private boolean isSelinuxFlagInEnabled() {
        try {
            Class c = Class.forName("android.os.SystemProperties");
            Method get = c.getMethod("get", String.class);
            String selinux = (String) get.invoke(c, "ro.build.selinux");
            return "1".equals(selinux);
        } catch (Exception ignored) {

        }
        return false;
    }
}
```

## 检测 `ro.debuggable` 和 `ro.secure` 值

`"ro.debuggable"` 是 Android 操作系统中的一个属性，它指示设备是否已启用 Android Debug Bridge（ADB）功能。当该属性设置为 `"1"` 时，设备将启用 ADB 功能，允许开发人员使用各种工具和命令在设备上进行调试和测试。在生产环境中，该属性通常应该被设置为 `"0"` 以增加设备的安全性。如果该属性设置为 `"1"`，则任何人都可以使用 ADB 命令访问设备，从而可能会导致设备受到攻击或其他安全问题

`"ro.secure"` 是一个 Android 操作系统中的系统属性（System Property），它控制着该设备是否开启了安全性强化措施。如果该属性的值为 `"1"`，则表示该设备已开启安全性强化措施。如果该属性的值为 `"0"`，则表示该设备未开启安全性强化措施。

不过笔者root过的N5x的`ro.secure`值为`1`，不能盲目的相信`ro.secure`

```
bullhead:/ $ getprop ro.debuggable

getprop ro.debuggable

1

bullhead:/ $ getprop ro.secure

getprop ro.secure

1
```

所以，检测的逻辑是检查`ro.debuggable`属性是否为`true`，为`true`的话APP所运行环境很可能是Root环境

使用java代码实现

```
package com.example.checkdebuggableandsecure;

import android.os.Bundle;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;

import java.io.IOException;
import java.io.InputStream;
import java.util.HashMap;
import java.util.Map;
import java.util.NoSuchElementException;
import java.util.Scanner;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        checkForDangerousProps();
    }

    private void checkForDangerousProps() {

        final Map dangerousProps = new HashMap<>();
        dangerousProps.put("ro.debuggable", "1");

        boolean result = false;

        String[] lines = propsReader();

        if (lines == null){
            // Could not read, assume false;
            return;
        }

        for (String line : lines) {
            for (String key : dangerousProps.keySet()) {
                if (line.contains(key)) {
                    String badValue = dangerousProps.get(key);
                    badValue = "[" + badValue + "]";
                    if (line.contains(badValue)) {
                        Toast.makeText(getApplicationContext(), "检测到危险值 "+key+": "+badValue,
                                Toast.LENGTH_LONG).show();
                    }
                }
            }
        }
    }

    private String[] propsReader() {
        try {
            InputStream inputstream = Runtime.getRuntime().exec("getprop").getInputStream();
            if (inputstream == null) return null;
            String propVal = new Scanner(inputstream).useDelimiter("\\A").next();
            return propVal.split("\n");
        } catch (IOException | NoSuchElementException e) {
            return null;
        }
    }
}
```

N5x执行以上代码的结果

![N5x执行以上代码的结果](https://attach.52pojie.cn/forum/202303/22/162810xkk53v9kl3v7f5cj.png)

`202303215.png` (45.72 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMDYxMHxjZTg2ODY0ZXwxNzkzNTY5NjI0fDE2ODA4NnwxNzYzMTEx&nothumb=yes)

2023-3-22 16:28 上传

[备用图片](http://www.dtasecurity.cn:20080/IMAGE/OFFLINE/202303215.png)

## 检查特定路径是否有写权限

在 Linux 和 Android 的系统中，root 用户有着最高的权限，可以在系统中执行一些非常低级的操作，例如更改系统设置、系统文件、程序文件等。而其他常规用户则只能够在自己拥有的权限下进行操作，通过检查具体路径的读写权限，可以判断设备是否被root

下列Java代码将使用`mount`命令检查这些路径的读写权限，如果可读写，说明设备可能被root了

- `/system`
- `/system/bin`
- `/system/sbin`
- `/system/xbin`
- `/vendor/bin`
- `/sbin`
- `/etc`
- `/sys`
- `/proc`
- `/dev`
