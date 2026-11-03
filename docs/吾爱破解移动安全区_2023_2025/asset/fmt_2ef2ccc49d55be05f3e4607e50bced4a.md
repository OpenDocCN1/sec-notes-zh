
```js
        fetch('/auth/uid').then(res => res.text()).then(res => {
            if (res) {
                document.querySelector('#uid').textContent = res;
                document.querySelector('#logout-form').style.display = '';
                document.querySelector('#login-form').style.display = 'none';
            }
        });
```

那么就好办了，我们来偷梁换柱一下，把cookie中的`flagA`设置到`uid`中，再`fetch`......

![](https://attach.52pojie.cn/forum/202410/30/172919jvjb0bdb3uswdrgu.png)

a686c9177f3e67091d90d3c67dc79f3df8dc5573.png (40.16 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMjIxMnw3YzIxMzdkZnwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMjIxMnw3YzIxMzdkZnwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes)

2024-10-30 17:29 上传

> 
`flagA`{f96a1e5e}

注：`flagA`每10分钟刷新

## Flag5

还是那个网页，`F12`，网页里用注释提到了`Flag5`

![](https://attach.52pojie.cn/forum/202410/30/172922seoz1kak9c9m004m.png)

d1a20cf431adcbeffccae83feaaf2edda3cc9f09.png (97.57 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMjIxM3wwMDI1ZDUxMHwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMjIxM3wwMDI1ZDUxMHwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes)

2024-10-30 17:29 上传

我们把`style`中的属性全部去掉，得到以下一串东西（用图片了，直接发论坛MD渲染会崩）：

![](https://attach.52pojie.cn/forum/202410/30/172924zcm4ovaq7trtrzac.png)

79f0f736afc379311a8c1010adc4b74543a91126.png (17.82 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMjIxNHxmODZkMjcxNnwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMjIxNHxmODZkMjcxNnwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes)

2024-10-30 17:29 上传

我们暂且不管. _ / \那一堆符号（flag9的地方会说），剩下的就是`flag5`

> 
`flag5`{P3prqF}

## Flag6

还是那个网页，下方有个`flag6`的按钮，点击进入`flag6`

网页很干净，就一个按钮，点了就开始炼丹，电脑风扇呼呼响（doge）

![](https://attach.52pojie.cn/forum/202410/30/172926s31b3njsqr1sk3nm.png)

4d086e061d950a7b3aecd8fd4cd162d9f2d3c92a.png (72.13 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMjIxNXw0NGMyMjk0NnwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMjIxNXw0NGMyMjk0NnwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes)

2024-10-30 17:29 上传

还是来看看源码吧：

```js
document.querySelector("button").addEventListener("click", () => {
  const t0 = Date.now();
  for (let i = 0; i
```

简而言之就是它跑了一个从0到$10^8 - 1$的数字字符串`i`，当该字符串的`md5`为`1c450bbafad15ad87c32831fa1a616fc`时，输出`flag6`{${i}}，否则在`console`中定期输出计算进度

（好家伙，暴力破解`md5`，真有你的）

直接`md5`彩虹表反查，发现是今天的日期，绝了......

![](https://attach.52pojie.cn/forum/202410/30/172929n9k82pv8s8611pfk.png)

7aec54e736d12f2e20cc385909c2d5628535683a.png (316.82 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMjIxNnw3YTdiNzM4ZnwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMjIxNnw3YTdiNzM4ZnwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes)

2024-10-30 17:29 上传

> 
`flag6`{20240217}

## Flag7

作者在视频里面留下了一个[//github.com/ganlvtech/52pojie-2024-challenge" target="_blank" rel="noopener noreferrer nofollow"](https://github.com/ganlvtech/52pojie-2024-challenge:rel="nofollow noopener" href="https)，打开发现这个：

![](https://attach.52pojie.cn/forum/202410/30/172932zzsm0w13sxux9m5s.png)

d4628535e5dde711316bd4cfe1efce1b9d16614a.png (137.14 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMjIxN3w2YmFjNjcxZnwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMjIxN3w2YmFjNjcxZnwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes)

2024-10-30 17:29 上传

> 
"删除不小心提交的flag内容"

提示够明显了，我们直接点击`commit`寻找历史提交记录，找到了这个

![](https://attach.52pojie.cn/forum/202410/30/172934n066mdi00euuuuy0.png)

0eb30f2442a7d933c91d734aeb4bd11373f00155.png (106.96 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjczMjIxOHxmOTFlYzk5ZXwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMjIxOHxmOTFlYzk5ZXwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes)

2024-10-30 17:29 上传

> 
`flag7`{Djl9NQ}

## Flag8 & FlagB

2048小游戏

![](https://attach.52pojie.cn/forum/202410/30/172936s4wooa3tt9phcaoo.png)

dc54564e9258d10908a7388b9758ccbf6c814d47.png (77.19 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjczMjIxOXw4N2VkODIwZXwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMjIxOXw4N2VkODIwZXwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes)

2024-10-30 17:29 上传

首先肯定是玩咯，轻轻松松通过玩游戏顺利拿到 `flag8`

![](https://attach.52pojie.cn/forum/202410/30/172939g425ekooexx20sx2.png)

f636afc379310a55a3b04fcdf14543a982261076.png (110.65 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMjIyMHw5OThkZGU5YnwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMjIyMHw5OThkZGU5YnwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes)

2024-10-30 17:29 上传

> 
`flag8`{OaOjIK}

接着拿剩下的金币V了作者50（doge）

> 
竟然真的有人v我50，真的太感动了。作为奖励呢，我就提示你一下吧，关键词是“溢出”。

首先想到的肯定是多买点，然后让它溢出，可惜有可能弄得太猛了，导致溢出后金币数量增加了，作者也想到了这个，购买请求直接被拦截了

![](https://attach.52pojie.cn/forum/202410/30/172941t9h9kh9hlcclolcm.png)

4afbfbedab64034f7341533fe9c379310a551d7e.png (16.91 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMjIyMXwwMGRhYTE3YnwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMjIyMXwwMGRhYTE3YnwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes)

2024-10-30 17:29 上传

猜想是做了检验，即购买后金币数量不能高于现有数量。

手算了几个临界值，罢了，完全没用，

因为题目没有写明白它后端到底用的啥数据类型，因此放弃思考，直接用`request`组件爆破
