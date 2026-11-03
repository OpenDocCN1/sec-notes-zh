
# Java DES 与 DESede 对称加密示例

## DES 解密代码片段

```java
DESKeySpec desKeySpec = new DESKeySpec(keys.getBytes());
// 实例化秘密密钥工厂
SecretKeyFactory secretKeyFactory = SecretKeyFactory.getInstance("DES");
// 生成秘密密钥
SecretKey secretKey = secretKeyFactory.generateSecret(desKeySpec);
// 实例化Cipher
Cipher cipher = Cipher.getInstance("DES/ECB/PKCS5Padding"); // 加密算法/加密模式/填充方式
// 初始化操作：加载密钥进cipher并设置为解密模式
cipher.init(Cipher.DECRYPT_MODE, secretKey);
// 执行解密操作
byte[] res2 = cipher.doFinal(new BASE64Decoder().decodeBuffer(str));
System.out.println("解密后的明文：" + new String(res2));
}
}
```

以上这段代码是一个`Java`类`DesExample`，它为我们演示对字符串进行DES加密和解密操作。以下是这段代码的讲解：

### 类结构说明
- `DesExample`类包含了两静态属性`keys`和`values`，用于存储DES加密密钥和待加密的字符串。
- `DesEncryption`方法是一个私有静态方法，用于对输入的字符串进行DES加密操作。该方法接收加密密钥和待加密的字符串作为参数，并返回加密后的字节数组。

### `main`方法执行流程
`main`方法是程序的入口点。在这个方法中：
- 初始化`keys`为字符串`"a12345678"`，`values`为字符串`"12345678"`。
- 调用`DesEncryption`方法对`values`进行DES加密，将加密后的结果使用BASE64编码转换为字符串。
- 实例化DES密钥材料并生成秘密密钥，供后续解密使用。
- 实例化`Cipher`对象，并设置为DES算法、ECB模式以及PKCS5填充方式。
- 用生成的秘密密钥初始化`Cipher`对象，设置为解密模式。
- 对加密后的字符串进行BASE64解码，然后用`Cipher`对象执行解密操作，得到解密后的字节数组。
- 将解密后的字节数组转换为字符串，并输出到控制台。

我在代码中添加的注释为每个关键步骤提供了解释，可以帮助大家更容易理解代码执行流程。

## 注意事项
需要注意的是，代码中使用了`sun.misc.BASE64Decoder`和`sun.misc.BASE64Encoder`这两个类，它们是`sun.*`的非标准API，不建议在生产环境中使用，因为它们不是公共API，并可能在将来的`Java`版本中被移除。

## 运行效果

```
ZEi7hkmMp+Dr8CE3zdeHkg==
12345678
```

我们可以看到从加密前的`"12345678"`到加密后的`"ZEi7hkmMp+Dr8CE3zdeHkg=="`，再到解密后得到的明文`"12345678"`这么一个完整的过程。但这里要注意的是加密和解密的过程是相对的，比如说上面的代码中对于明文的加密流程是先通过`getBytes()`方法将明文转换为`byte`序列，然后再进行加密操作并返回密文，最后把密文进行BASE64编码操作；我们再来看看上面代码中解密的过程，因为加密和解密的过程是相对的，所以你在代码中可以看到解密的第一步是先对密文进行BASE64解码操作，然后再进行解密操作，最后把解密后的内容转换为字符串，从而得到最后的明文。

`DES`对称加密算法我们就讲到这，接下来我们快速了解`DESede`对称加密算法，其实这两种对称加密算法是差不多的，要说区别有这么几种，在讲区别之前我们先来看看`DESede`对称加密算法如何使用`JAVA`代码实现：

## DESede 加密示例

```java
package com.java.Reverse;

import sun.misc.BASE64Decoder;

import sun.misc.BASE64Encoder;

import javax.crypto.Cipher;

import javax.crypto.SecretKey;

import javax.crypto.SecretKeyFactory;

import javax.crypto.spec.DESKeySpec;

import javax.crypto.spec.DESedeKeySpec;

public class DESedeExample {

    private static String DESedeDecode(String keys, String values) throws Exception{

        // 实例化DES密钥材料

        DESedeKeySpec deSedeKeySpec = new DESedeKeySpec(keys.getBytes());

        // 实例化秘密密钥工厂

        SecretKeyFactory secretKeyFactory = SecretKeyFactory.getInstance("DESede");

        // 生成秘密密钥

        SecretKey secretKey = secretKeyFactory.generateSecret(deSedeKeySpec);

        // 实例化

        Cipher cipher = Cipher.getInstance("DESede/ECB/PKCS5Padding"); // 加密算法/加密模式/填充方式

        // 初始化操作：把密钥加载进cipher并设置为加/解密模式，反编译时这里1表示加密模式，2表示解密模式

        cipher.init(Cipher.DECRYPT_MODE, secretKey);

        // 执行操作

        byte[] decode = cipher.doFinal(new BASE64Decoder().decodeBuffer(values));

        return new String(decode);

    }

    public static void main(String[] args) throws Exception{

        String key = "123456781234567812345678";

        String value = "a12345678";

        // 实例化DESede密钥材料

        DESedeKeySpec deSedeKeySpec = new DESedeKeySpec(key.getBytes());

        // 实例化秘密密钥工厂

        SecretKeyFactory secretKeyFactory = SecretKeyFactory.getInstance("DESede");

        // 生成秘密密钥

        SecretKey secretKey = secretKeyFactory.generateSecret(deSedeKeySpec);

        // 实例化

        Cipher cipher = Cipher.getInstance("DESede/ECB/PKCS5Padding"); // 加密算法/加密模式/填充方式

        // 初始化操作：把密钥加载进cipher并设置为加/解密模式，反编译时这里1表示加密模式，2表示解密模式

        cipher.init(Cipher.ENCRYPT_MODE, secretKey);

        // 执行操作

        byte[] res = cipher.doFinal(value.getBytes());

        String str = new BASE64Encoder().encode(res);

        System.out.println("加密后的内容为：" + str);

        String decode = DESedeDecode(key, str);

        System.out.println("解密后的内容为：" + decode);

    }

}
```

### 引入的类和包
- `sun.misc.BASE64Decoder` 和 `sun.misc.BASE64Encoder`: 这两个类用于进行 BASE64 编码和解码，以便处理二进制数据（如加密后的字节）。
- `javax.crypto.Cipher`: 用于加密和解密操作的核心类。
- `javax.crypto.SecretKey`: 存储密钥的对象。
- `javax.crypto.SecretKeyFactory`: 用于生成从指定密钥材料中生成的秘密密钥的工厂类。
- `javax.crypto.spec.DESKeySpec` 和 `javax.crypto.spec.DESedeKeySpec`: 这些 specifications 用于创建 DES 和 DESede 密钥材料。

### 类功能
该类的功能是通过 `DESede` 算法加密和解密文本。`DESede` 是通过三重 DES（Triple DES）算法实现的一个变种，提供比单一 DES 更强的安全性。

### 主要方法

#### `DESedeDecode(String keys, String values)`
该方法负责执行 `DESede` 解密操作。
- 参数:
  - `keys`: 用于解密的密钥字符串
  - `values`: 被 BASE64 编码的密文
- 步骤:
  - 将密钥字符串转换为 `DESedeKeySpec`。
  - 使用 `SecretKeyFactory` 实例化一个密钥工厂。
  - 根据密钥材料生成 `SecretKey`。
  - 获取 `Cipher` 实例以实现 `DESede` 算法的解密模式。
  - 初始化 `Cipher` 对象为解密模式。
  - 使用 `doFinal()` 方法对 BASE64 解码后的密文进行解密，返回解密后的明文字符串。

#### `main(String[] args)`
主方法用于演示加密和解密的过程。
步骤:
- 定义一个密钥和要加密的明文。
- 将密钥字符串转换为 `DESedeKeySpec`。
- 生成秘密密钥。
- 创建 `Cipher` 实例以执行加密过程。
- 将 `Cipher` 初始化为加密模式，并对明文进行加密。
- 将加密后的结果进行 BASE64 编码，便于打印和传输。
- 打印加密后的内容。
- 调用 `DESedeDecode` 方法进行解密，并打印解密结果。

### 使用示例
假设密钥为 `123456781234567812345678`，要加密的明文为 `a12345678`，它的过程如下：
- 将明文加密，结果通过 BASE64 编码输出。
- 将加密的 BASE64 字符串反转回明文并输出。

我们简单的了解了以上代码，我们可以看出这两种算法最明显的区别是密钥的长度不一样，一个是八位的，一个是二十四位的。其次就是使用的实例化函数和实例化得到的结果不一样，还有就是 `SecretKeyFactory.getInstance()` 函数指明的类型也是不一样的，其他的地方就是差不多的。

好了，简单的过了 `DESede` 对称加密算法，我们接下来去了解下一个常用的对称加密算法——`AES`；废话不多说，直接上代码：
