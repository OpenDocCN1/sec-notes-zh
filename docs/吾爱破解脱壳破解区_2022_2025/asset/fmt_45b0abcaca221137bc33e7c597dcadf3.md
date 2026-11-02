
# [原创] 关于 java-web 应用程序的逆向-camunda 企业版

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=1776897](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1776897)
- 抓取时间：2026-11-01T22:55:03

关于 camunda，它是一款工作流组件，源自 activity5。支持 BPMN（工作流和流程自动化）、CMMN（案例管理）和 DMN（业务决策管理）的 Java 框架。德国的一个公司开发的。

![camunda介绍](https://attach.52pojie.cn/forum/202304/21/142734mihbxfrphyfrfhyx.png)

`camunda介绍.png`（92.64 KB，下载次数：2）  
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTIxMnw0NTQxZDViMXwxNzkzNTczNzAwfDE2ODA4NnwxNzc2ODk3&nothumb=yes)  
2023-4-21 14:27 上传

## 题外话篇：一、如何下载企业版

下载地址：[https://camunda.com/download/](https://camunda.com/download/)

下载之前需要在 camunda 官网注册账号，账号和密码会发送到你的邮箱，如图：

![邮件](https://attach.52pojie.cn/forum/202304/21/143434i1m8dc1288gozhd8.png)

`邮件.png`（67.07 KB，下载次数：1）  
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTIxNnxkMWY1MTc4YnwxNzkzNTczNzAwfDE2ODA4NnwxNzc2ODk3&nothumb=yes)  
2023-4-21 14:34 上传

![邮件2](https://attach.52pojie.cn/forum/202304/21/143551ww4833c340w8www4.png)

`邮件2.png`（60.82 KB，下载次数：1）  
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTIxOXxhMjM0ZDg2YXwxNzkzNTczNzAwfDE2ODA4NnwxNzc2ODk3&nothumb=yes)  
2023-4-21 14:35 上传

![下载地址](https://attach.52pojie.cn/forum/202304/21/143703hrst5z05urttqg0s.png)

`下载地址.png`（104.92 KB，下载次数：0）  
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTIyNHw2YTgxY2FjNXwxNzkzNTczNzAwfDE2ODA4NnwxNzc2ODk3&nothumb=yes)  
2023-4-21 14:37 上传

点击即可下载。

## 题外话篇：二、关于 camunda 的体验

注册后，他们的销售会联系你购买他们企业版，也就是推推业务员。

![会议1](https://attach.52pojie.cn/forum/202304/21/144604dnwdbn00dltqk0dc.png)

`会议1.png`（44.35 KB，下载次数：1）  
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTIyOXw3MmJlNDY3YnwxNzkzNTczNzAwfDE2ODA4NnwxNzc2ODk3&nothumb=yes)  
2023-4-21 14:46 上传

基本都是给你发 meeting 会议链接。

![会议链接](https://attach.52pojie.cn/forum/202304/21/144939tx0nb9vt02hhcn9z.png)

`会议链接.png`（29.18 KB，下载次数：1）  
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTIzMHwyYmU1NzYwMXwxNzkzNTczNzAwfDE2ODA4NnwxNzc2ODk3&nothumb=yes)  
2023-4-21 14:49 上传

简单的通过文字会议形式了解了一下的结果就是：人家不服务中国，不做中国市场业务。感兴趣伙伴可以注册下沟通就知道。

## 正文

下载下来后内容如下：

![压缩包](https://attach.52pojie.cn/forum/202304/21/145621xmmibbd7x1wqerz4.png)

`压缩包.png`（39.13 KB，下载次数：1）  
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTIzNXwxNTkzZmY3N3wxNzkzNTczNzAwfDE2ODA4NnwxNzc2ODk3&nothumb=yes)  
2023-4-21 14:56 上传

- `configuration`：一些启动的配置。
- `internal`：主要的程序包。
- `start.bat`：Windows 下的启动脚本。

启动后访问地址：[http://localhost:8080/camunda-welcome/index.html](http://localhost:8080/camunda-welcome/index.html)

### 界面

![界面1](https://attach.52pojie.cn/forum/202304/21/145918e1jhc5k1dmxfd1bo.png)

`界面1.png`（210.99 KB，下载次数：1）  
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTIzNnxkYWIyZTY1NXwxNzkzNTczNzAwfDE2ODA4NnwxNzc2ODk3&nothumb=yes)  
2023-4-21 14:59 上传

随便点击 `tasklist` 或其他的几个界面，就如登录界面。默认账户密码都是 `demo`。

登录后：

![界面2](https://attach.52pojie.cn/forum/202304/21/150130xnjjrqrpggzkotqj.png)

`界面2.png`（71.82 KB，下载次数：1）  
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTIzN3wxYmNmYzkzMHwxNzkzNTczNzAwfDE2ODA4NnwxNzc2ODk3&nothumb=yes)  
2023-4-21 15:01 上传

![界面4](https://attach.52pojie.cn/forum/202304/21/174828wnstcgcj3wj8uwvv.png)

`界面4.png`（95.19 KB，下载次数：1）  
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTMwMnwzN2I3OGUxOXwxNzkzNTczNzAwfDE2ODA4NnwxNzc2ODk3&nothumb=yes)  
2023-4-21 17:48 上传

这是没有注册的。

可以看到这个是普通界面，并没有涵盖企业版的功能，需要输入注册的 License 才能看到企业级功能。

依次点击：`Admin` -> `License`，也可直接访问：

[http://localhost:8080/camunda/app/admin/default/#/system?section=system-settings-license](http://localhost:8080/camunda/app/admin/default/#/system?section=system-settings-license)

进入以下界面：

![界面3](https://attach.52pojie.cn/forum/202304/21/150518b4511fw2f4z111fx.png)

`界面3.png`（75.44 KB，下载次数：1）  
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTIzOHw5NmVkMTYxNXwxNzkzNTczNzAwfDE2ODA4NnwxNzc2ODk3&nothumb=yes)  
2023-4-21 15:05 上传

随便输入点什么，提交一下看看请求地址。

![请求1](https://attach.52pojie.cn/forum/202304/21/150741hqoaglcquoq0q483.png)

`请求1.png`（27.51 KB，下载次数：1）  
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTIzOXxkYzU1NzBmNnwxNzkzNTczNzAwfDE2ODA4NnwxNzc2ODk3&nothumb=yes)  
2023-4-21 15:07 上传

请求地址：[http://localhost:8080/camunda/api/admin/plugin/license/default/key](http://localhost:8080/camunda/api/admin/plugin/license/default/key)

返回数据：

```json
{"invalidMessage":"License Key has wrong format.","customerId":null,"validUntil":null,"valid":false,"unlimited":false,"licenseProvided":true}
```

提示格式不对。

![启动1](https://attach.52pojie.cn/forum/202304/21/151003z3bpetihid8hqtds.png)

`启动1.png`（18.1 KB，下载次数：1）  
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTI0MHw1ODcwYzU5MXwxNzkzNTczNzAwfDE2ODA4NnwxNzc2ODk3&nothumb=yes)  
2023-4-21 15:10 上传

通过启动脚本的日志可以看出是使用的 `Spring-Boot`，那么直接可以通过对应的请求地址，直接定位到他的服务类代码。

### 通过启动脚本分析

![启动脚本1](https://attach.52pojie.cn/forum/202304/21/151545s2myddd9dy25u3es.png)

`启动脚本1.png`（49.58 KB，下载次数：1）  
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTI0MnxiNDQ1MWQ4NHwxNzkzNTczNzAwfDE2ODA4NnwxNzc2ODk3&nothumb=yes)  
2023-4-21 15:15 上传

![启动脚本2](https://attach.52pojie.cn/forum/202304/21/151548eqlaokqyhtb4z0ah.png)

`启动脚本2.png`（43.57 KB，下载次数：1）  
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTI0M3w3ZGQ5ZWEzNnwxNzkzNTczNzAwfDE2ODA4NnwxNzc2ODk3&nothumb=yes)  
2023-4-21 15:15 上传

结论：`start.bat` 调用了 `internal\run.bat`。

`run.bat` 执行的程序包为 `internal\camunda-bpm-run-core.jar`。

接下来就是分析这个程序包了，通过 `jd-gui` 查看，可以看到是没有加密或者混淆的，可以随便造。

社区版代码：[https://github.com/camunda/camunda-bpm-platform](https://github.com/camunda/camunda-bpm-platform)

可以看看社区版的代码，这个是开源的，只是功能没有企业版强大。

### 最后的分析结论

`internal\webapps\camunda-license-check-2.7.0.jar`

分析完你会知道，最终验证的代码都在这个程序包里面。

### 题外话：调试方法

这里有很多种追踪方法。这里说下我所用的方式：`remotedebug`。

在 `run.bat` 里面的启动方法 `-jar` 前面加入以下代码：

```
-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005
```

通过调试模式启动这个服务。

使用 `jd-gui` 打开 `camunda-license-check-2.7.0.jar`，保存所有文件为源码，保存后直接导入 `IDE`，有报错不管他。

![存源码](https://attach.52pojie.cn/forum/202304/21/154710gfe81eozcwccofgc.png)

`存源码.png`（15.89 KB，下载次数：0）
