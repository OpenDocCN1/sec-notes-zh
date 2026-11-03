
```
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.nio.ByteBuffer;
import java.util.HashMap;

public class Main {

    private static File fix(ByteBuffer byteBuffer, int v, int v1, int v2) throws Exception {
        File file = new File("./data");
        int classDataOffset = (int) (((Integer) D.getClassDefData(byteBuffer, v).get("class_data_off")));
        HashMap classData = D.getClassData(byteBuffer, classDataOffset);
        classData.get("direct_methods")[v1][2] = v2;
        byte[] encodedClassData = D.encodeClassData(classData);
        byteBuffer.position(classDataOffset);
        byteBuffer.put(encodedClassData);
        byteBuffer.position(0x20);
        byte[] data = new byte[byteBuffer.capacity() - 0x20];
        byteBuffer.get(data);
        byte[] sha1 = Utils.getSha1(data);
        byteBuffer.position(12);
        byteBuffer.put(sha1);
        int checksum = Utils.checksum(byteBuffer);
        byteBuffer.position(8);
        byteBuffer.putInt(Integer.reverseBytes(checksum));
        byte[] bufferData = byteBuffer.array();
        File outputFile = new File(file, "2.dex");
        FileOutputStream fileOutputStream = new FileOutputStream(outputFile);
        fileOutputStream.write(bufferData);
        fileOutputStream.close();
        return outputFile;
    }

    private static ByteBuffer readDex() {
        try {
            File inputFile = new File("./data/1.dex");
            if (!inputFile.exists()) {
                System.out.printf("File does not exist.");
                return null;
            }
            FileInputStream fileInputStream = new FileInputStream(inputFile);
            byte[] data = new byte[fileInputStream.available()];
            fileInputStream.read(data);
            ByteBuffer byteBuffer = ByteBuffer.wrap(data);
            fileInputStream.close();
            System.out.printf("\n File read successfully.");
            return byteBuffer;
        } catch (Exception ex) {
            return null;
        }
    }

    public static void main(String[] args) throws Exception {
        ByteBuffer buf = readDex();
        byte byteValue = buf.get(); // 读取一个字节
        char charValue = (char) byteValue;
        System.out.printf("%c", charValue);
        byteValue = buf.get(); //
        charValue = (char) byteValue;
        System.out.printf("%c", charValue);
        byteValue = buf.get(); //
        charValue = (char) byteValue;
        System.out.printf("%c", charValue);
        //key = [0,3,7908]
        //key = [1,1,8108]
        File fixed = fix(buf, 0, 3, 7908);
    }

}
```

```
import java.math.BigInteger;
import java.nio.ByteBuffer;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.ArrayList;

/* loaded from: assets/classes.dex */
public class Utils {

    public static final String SHA1 = "SHA1";

    public static byte[] toULEB128(int i) {
        int i2 = i >> 28;
        if (i2 > 0) {
            return new byte[]{(byte) ((i & 127) | 128), (byte) (((i >> 7) & 127) | 128), (byte) (((i >> 14) & 127) | 128), (byte) (((i >> 21) & 127) | 128), (byte) (i2 & 15)};
        }
        int i3 = i >> 21;
        if (i3 > 0) {
            return new byte[]{(byte) ((i & 127) | 128), (byte) (((i >> 7) & 127) | 128), (byte) (((i >> 14) & 127) | 128), (byte) (i3 & 127)};
        }
        int i4 = i >> 14;
        if (i4 > 0) {
            return new byte[]{(byte) ((i & 127) | 128), (byte) (((i >> 7) & 127) | 128), (byte) (i4 & 127)};
        }
        int i5 = i >> 7;
        return i5 > 0 ? new byte[]{(byte) ((i & 127) | 128), (byte) (i5 & 127)} : new byte[]{(byte) (i & 127)};
    }

    public static byte[] getSha1(byte[] bArr) {
        try {
            return MessageDigest.getInstance("SHA").digest(bArr);
        } catch (Exception unused) {
            return null;
        }
    }

    public static String md5(byte[] bArr) {
        try {
            String bigInteger = new BigInteger(1, MessageDigest.getInstance("md5").digest(bArr)).toString(16);
            for (int i = 0; i  arrayList, int i, int i2) {
        int i3 = 0;
        while (i3  127) {
            int i3 = byteBuffer.get() & 255;
            i2 = (i2 & 127) | ((i3 & 127)  127) {
                int i4 = byteBuffer.get() & 255;
                i2 |= (i4 & 127)  127) {
                    int i5 = byteBuffer.get() & 255;
                    i2 |= (i5 & 127)  127) {
                        i2 |= (byteBuffer.get() & 255)  0) {
            int v = 0;
            int v1 = 0;
            while(v  0) {
            int v2 = 0;
            int v3 = 0;
            while(v2  0) {
            int v4 = 0;
            int v5 = 0;
            while(v4  0) {
            int v6 = 0;
            int v7 = 0;
            while(v6  0) {
            int v5 = 0;
            int v6 = 0;
            while(v5  0) {
            int v7 = 0;
            int v8 = 0;
            while(v7  0) {
            int v9 = 0;
            int v10 = 0;
            while(v9  0) {
            int v11 = 0;
            int v12 = 0;
            while(v11 
```

（其实都是从安卓复制过来的然后修修报错能运行就问题不大了

![](./../image/1710399685934-34.png)

最后也是终于获得了机缘

![](./../image/1710399685934-35.png)
