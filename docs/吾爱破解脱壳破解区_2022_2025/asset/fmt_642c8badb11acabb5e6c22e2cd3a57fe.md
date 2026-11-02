
```
./clang -mllvm -fla try.cpp -o try
```

成功生成混淆后的文件，用`ida`查看效果

![image-20240115132134904.png](https://attach.52pojie.cn/forum/202401/15/175116evb3ozw3p8vi39t3.png)

`image-20240115132134904.png` (80.75 KB, 下载次数: 0)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MDM5Mnw0YzU2NjNjM3wxNzkzNTcyNzQzfDE2ODA4NnwxODgwODQ1&nothumb=yes)

2024-1-15 17:51 上传

## 去平坦化

### 虚拟环境

安装虚拟环境：

```
pip install virtualenv
```

配置环境变量：

```
export WORKON_HOME=$HOME/.virtualenvs
export PROJECT_HOME=$HOME/workspace
source /usr/local/bin/virtualenvwrapper.sh
```

打开虚拟环境：

```
source ~/.local/bin/virtualenvwrapper.sh
```

创建新的虚拟环境：

```
mkvirtualenv demo
```

展示当前虚拟环境：

```
workon
```

进入虚拟环境：

```
workon demo
```

### 使用脚本

由于脚本需要`angr`，在当前虚拟环境中安装：

```
pip install angr
```

脚本 GitHub 地址：[https://github.com/cq674350529/deflat](https://github.com/cq674350529/deflat)

将脚本中的`am_graph.py`和`util.py`文件复制到下级

![image-20240115113143066.png](https://attach.52pojie.cn/forum/202401/15/175114qzpmq2826i2ki372.png)

`image-20240115113143066.png` (54.38 KB, 下载次数: 1)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MDM5MXw5NzZkNDRhMHwxNzkzNTcyNzQzfDE2ODA4NnwxODgwODQ1&nothumb=yes)

2024-1-15 17:51 上传

在`ida`中找到平坦化的入口地址，一般而言入口前都会存在大量`var_`（局部变量），下图入口地址为`0x401140`

![image-20240115132230565.png](https://attach.52pojie.cn/forum/202401/15/175121psnsesnaew4j1jdh.png)

`image-20240115132230565.png` (105.26 KB, 下载次数: 0)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MDM5M3xhMjYxOTVjY3wxNzkzNTcyNzQzfDE2ODA4NnwxODgwODQ1&nothumb=yes)

2024-1-15 17:51 上传

去平坦化命令格式为：

```
python3 deflat.py -f filename --addr address（入口地址，一般为main入口）
```

```
python3 deflat.py -f try --addr 0x401140
```

![image-20240115132332941.png](https://attach.52pojie.cn/forum/202401/15/175124jappg2vllmf1wp1p.png)

`image-20240115132332941.png` (122.93 KB, 下载次数: 0)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MDM5NXxlMmQwYmViY3wxNzkzNTcyNzQzfDE2ODA4NnwxODgwODQ1&nothumb=yes)

2024-1-15 17:51 上传

![image-20240115132452335.png](https://attach.52pojie.cn/forum/202401/15/175127xgqpr1nghiiffji6.png)

`image-20240115132452335.png` (73.3 KB, 下载次数: 0)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MDM5Nnw4NzEzYmFmYXwxNzkzNTcyNzQzfDE2ODA4NnwxODgwODQ1&nothumb=yes)

2024-1-15 17:51 上传

去平坦化成功后，会在当前目录下生成源文件`_recovered`

![image-20240115132612816.png](https://attach.52pojie.cn/forum/202401/15/175129xj4krwhjh2p1nwiy.png)

`image-20240115132612816.png` (62.83 KB, 下载次数: 0)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MDM5N3wyMjZjYzQxM3wxNzkzNTcyNzQzfDE2ODA4NnwxODgwODQ1&nothumb=yes)

2024-1-15 17:51 上传

在`ida`中查看去平坦化后效果

![image-20240115132654105.png](https://attach.52pojie.cn/forum/202401/15/175132sn1ncc1ucmdbdf2f.png)

`image-20240115132654105.png` (50.14 KB, 下载次数: 0)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MDM5OHxmMzU1YzFlOHwxNzkzNTcyNzQzfDE2ODA4NnwxODgwODQ1&nothumb=yes)

2024-1-15 17:51 上传

### 总结

- 定位入口地址
- 在`./deflat-master/flat_control_flow/`路径下启动虚拟环境

    ```
    source ~/.local/bin/virtualenvwrapper.sh
    workon
    workon demo
    ```

- 使用去平坦化命令

    ```
    python3 deflat.py -f filename --addr address（入口地址，一般为main入口）
    ```

## 参考文章

- [https://blog.csdn.net/weixin_51732593/article/details/126707046](https://blog.csdn.net/weixin_51732593/article/details/126707046)
- [https://www.jianshu.com/p/9136f7257e46](https://www.jianshu.com/p/9136f7257e46)
- [https://www.52pojie.cn/thread-1601573-1-1.html](https://www.52pojie.cn/thread-1601573-1-1.html)
