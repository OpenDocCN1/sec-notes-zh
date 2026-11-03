
# [Web逆向] 去除宝塔面板的推广以及专业版功能

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2053156
- 抓取时间: 2026-11-01T22:05:35

说明：本教程只适用于宝塔面板`v11.0.0`。其他版本还需要自行测试。

宝塔面板做运维的大部分都知道，简单便捷，虽然`1panel`出来了，但是还是感觉宝塔面板比较顺手。
但是我不是vip用户，宝塔面板也是关闭一些权限，剩下的那些vip功能还是收费，看着碍眼，索性去照代码里面删掉该部分即可。通过查看`/www/server/panel/BTPanel`可以得知前端使用`vue`写的。

## 关闭不必要的功能

设置->常用设置->面板栏隐藏，隐藏如下功能
WAF、WP Tools、多用户、邮局、节点管理

## 去除首页的软件推荐功能。

一开始我本以为这个是在`utils-lib.js`里面带着的，后来通过减慢请求查看到是在请求里面带着的。通过搜索关键词`get_pay_type`发现，是通过`getRecommendContent`这个请求函数带过来的。所以我们需要在这里设置返还值为空即将
`getRecommendContent=()=>useAxios.post("ajax/get_pay_type")`
替换成
`getRecommendContent=()=>{return []}`

## 去除其他界面的vip功能

这些就比较简单了，都是通过路由控制的，会一些`vue`基础的人很容易看出来。定位到`utils-lib.js`去掉如下内容

### 去除SSH登录日志

```
{path:"ssh",name:"ssh",meta:{title:"SSH登录日志"},component:()=>__vitePreload((()=>import("./index52.js?v=1754557180")),__vite__mapDeps([]),import.meta.url)},
```

### 去除自动部署

```
,{path:"auto-deploy",name:"auto-deploy",meta:{title:"自动部署"},component:()=>__vitePreload((()=>import("./index84.js?v=1754557180").then((e=>e.i))),__vite__mapDeps([]),import.meta.url)}
```

### 去除面板日报

```
,{path:"daily",name:"daily",meta:{title:"面板日报"},component:()=>__vitePreload((()=>import("./index12.js?v=1754557180")),__vite__mapDeps([]),import.meta.url)}
```

### 去除安全检测等其他功能

```
,{path:"safe-detect",name:"safe-detect",meta:{title:"安全检测"},component:()=>__vitePreload((()=>import("./index41.js?v=1754557180")),__vite__mapDeps([]),import.meta.url)},{path:"keyword",name:"keyword",meta:{title:"违规词检测"},component:()=>__vitePreload((()=>import("./index42.js?v=1754557180")),__vite__mapDeps([]),import.meta.url)},{path:"php-safe",name:"php-safe",meta:{title:"PHP网站安全"},component:()=>__vitePreload((()=>import("./index43.js?v=1754557180")),__vite__mapDeps([]),import.meta.url)},{path:"intrusion",name:"intrusion",meta:{title:"入侵防御"},component:()=>__vitePreload((()=>import("./index44.js?v=1754557180")),__vite__mapDeps([]),import.meta.url)},{path:"fixed",name:"fixed",meta:{title:"系统加固"},component:()=>__vitePreload((()=>import("./index45.js?v=1754557180")),__vite__mapDeps([]),import.meta.url)},{path:"network-scan",name:"network-scan",meta:{title:"扫描感知"},component:()=>__vitePreload((()=>import("./index46.js?v=1754557180")),__vite__mapDeps([]),import.meta.url)}
```
