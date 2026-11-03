
https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDgxN3wzMDI1ZTczNHwxNzkzNTcxNTI0fDE2ODA4NnwxOTg3NTU1&nothumb=yes

2024-12-2 15:42 上传

经过测试已经可以顺利隐藏注册按钮。

![](https://attach.52pojie.cn/forum/202412/02/154416frriyi7cimocacdz.png)

`image.png` (179.63 KB, 下载次数: 1)

https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDgxOHxhNzliNTg4Y3wxNzkzNTcxNTI0fDE2ODA4NnwxOTg3NTU1&nothumb=yes

2024-12-2 15:44 上传

again按钮隐藏的算法和注册按钮算法一样，还是在输入一遍刚才错误的密码在输入计算出来的密码就隐藏掉了。

![](https://attach.52pojie.cn/forum/202412/02/154704uuuj1nnymx1ayy2z.png)

`image.png` (39.41 KB, 下载次数: 0)

https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDgxOXwzYWQwZWEzYXwxNzkzNTcxNTI0fDE2ODA4NnwxOTg3NTU1&nothumb=yes

2024-12-2 15:47 上传

### 注册机

[C++] 纯文本查看 复制代码

```
#include 
using namespace std;

int getUser(string user, int passEdi)
{

	int len = user.length();
	int eax = 0;
	int ebx = 0;
	int ecx = 0;
	int edx = 0;
	int edi = passEdi;

	for (int i = 0; i < len; i++)
	{
		for (int j = 0; j < len; j++)
		{
			edx = user[i];
			ecx = user[len - j - 1];
			edx *= ecx;
			edx *= edi;
			ebx += edx;
		}
	}

	eax = ebx;
	ecx = 0xA2C2A;
	edx = eax % ecx;
	return edx;

}

int getPass(int userHex) {
	int i = 1;
	while (true)
	{
		if (i / 0x59 + i % 0x50 + 0x1 == userHex)
		{
			return i;
		}
		i++;
	}
}

int getPassEdi(string pass)
{
	int len = pass.length();
	int eax = 0;
	int ebx = 0;
	int ecx = 0;
	int edx = 0;
	int edi = 0x11;
	int esi = 0x37B;
	for (int i = 0; i < len - 1; i++)
	{
		eax = pass[i + 1];
		edx = eax % edi + 0x1;
		eax = pass[i];
		edx *= eax;
		esi += edx;
	}
	return esi;
}

int main()
{
	
	int passEdi = getPassEdi("a123456");
	cout << "密码计算出的Edi：  " << hex << passEdi << endl;
	int userHex = getUser("123456",passEdi);
	cout << "用户名计算出来的Hex：  " << hex << userHex << endl;
	int passHex = getPass(userHex);
	cout << "通过用户名计算出的密码：  "<< dec << passHex << endl;

	return 0;
}
```
