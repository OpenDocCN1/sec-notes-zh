
# 通过 Hook 获取语音合成音频并下载

## 背景与限制

```
window.URL.createObjectURL(new Blob([g.privAudioData], { type: 'audio/mp3' }));
```

![](https://attach.52pojie.cn/forum/202203/19/103602qthk1zi0d6z6kh2y.png)

`image-20220318221621847.png` (146.01 KB, 下载次数: 248)

[href="forum.php?mod=attachment&aid=MjUwNTAxOHxiYzdlMTcwOXwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAxOHxiYzdlMTcwOXwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:36 上传

一开始我想到用 chrome 的 overrides（fiddler 的 autoresponse 也可以做到）。

![](https://attach.52pojie.cn/forum/202203/19/103615ggkmgxm0e9em6uz9.png)

`image-20220318222128555.png` (291.84 KB, 下载次数: 260)

[href="forum.php?mod=attachment&aid=MjUwNTAxOXwxMDI0MzAzY3wxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAxOXwxMDI0MzAzY3wxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:36 上传

在把`g.privAudioData`导出到`window`，这样就方便后面调用了。

![](https://attach.52pojie.cn/forum/202203/19/103645wj3th1k084dt1mdh.png)

`image-20220318222527046.png` (305.75 KB, 下载次数: 256)

[href="forum.php?mod=attachment&aid=MjUwNTAyMXxkZGQxNDhkZHwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAyMXxkZGQxNDhkZHwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:36 上传

但是后面在下载的时候遇到了浏览器限制，并且这种替换线上文件的方法，对用户的配置要求较高。

## 3> 0x04

我的目标是做一个装上就可以用的 tampermonkey 脚本。

为了拿到`g.privAudioData`，我在当前的作用域中找找看有没有生命周期比较长的比如`JSON`等之类的可以 hook，发现并没有找到，但是我发现`g`是传进来的，于是找上一层，不行再找上一层。

最终我找到`document.addEventListener`，这个注入点可能不是最好的，如果大佬有更好的欢迎指正。

先说上面一层是：

![](https://attach.52pojie.cn/forum/202203/19/103718o5f1heheh0557uvf.png)

`image-20220318223715387.png` (333.49 KB, 下载次数: 263)

[href="forum.php?mod=attachment&aid=MjUwNTAyM3wwOTZhZTcxY3wxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAyM3wwOTZhZTcxY3wxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:37 上传

![](https://attach.52pojie.cn/forum/202203/19/103737gq16w5bt4wlqc7w1.png)

`image-20220318223921100.png` (352.29 KB, 下载次数: 252)

[href="forum.php?mod=attachment&aid=MjUwNTAyNHxkYmIwNWRlMXwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAyNHxkYmIwNWRlMXwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:37 上传

发现这里定义了一个匿名函数赋给了全局变量`window.initializeTTSDemo`，此处也没有找到合适的 hook 点，但是`arguments[1].privResult`里面有我们要的 MP3，我们要的东西没丢。

```
'use strict';

(function ($) {

    // This demo is only supported in Edge, Firefox, Chrome and Opera. Source: http://caniuse.com/#search=getUserMedia

    window.initializeTTSDemo = function (localizedResources) {

        $(document).ready(function () {

                        // 省略好多代码            

            SpeechSDK = window.SpeechSDK;

            // 省略好多代码

            function SpeakOnce() {

                var config = SpeechSDK.SpeechTranslationConfig.fromAuthorizationToken(localizedResources.token, localizedResources.region),

                    synthesizer,

                    audioConfig;

                // due to a bug in Chromium (https://bugs.chromium.org/p/chromium/issues/detail?id=1028206)

                // mp3 playback has some beeps, using a higher bitrate here as a workaround.

                config.speechSynthesisOutputFormat = SpeechSDK.SpeechSynthesisOutputFormat.Audio24Khz160KBitRateMonoMp3;

                player = new SpeechSDK.SpeakerAudioDestination();

                player.onAudioEnd = function () {

                    stopli.hidden = true;

                    playli.hidden = false;

                };

                audioConfig = SpeechSDK.AudioConfig.fromSpeakerOutput(player);

                synthesizer = new SpeechSDK.SpeechSynthesizer(config, audioConfig);

                synthesizer.synthesisCompleted = function () {

                    //arguments[1].privResult 里面有我们要的MP3

                    synthesizer.close();

                    synthesizer = null;

                };

                synthesizer.SynthesisCanceled = function (s, e) {

                    var details;

                    stopli.hidden = true;

                    playli.hidden = false;

                    details = SpeechSDK.CancellationDetails.fromResult(e);

                    if (details.reason === SpeechSDK.CancellationReason.Error) {

                        status.innerText = localizedResources.srTryAgain;

                    }

                };

                synthesizer.speakSsmlAsync(ssml.value, function () { }, function (error) {

                    status.innerText = localizedResources.srTryAgain + ' ' + error;

                });

            }

           // 省略好多代码

})(jQuery);
```

我非常想重写`initializeTTSDemo`，但是失败了。

在使用 tampermonkey 脚本时，使用默认注入方式和改成了`@run-at document-start`都不能覆盖这个函数。

于是我继续找`initializeTTSDemo`执行的地方：

![](https://attach.52pojie.cn/forum/202203/19/103759s98r9o59m4w9b0dm.png)

`image-20220318225231325.png` (332.34 KB, 下载次数: 246)

[href="forum.php?mod=attachment&aid=MjUwNTAyNXw5NmMxODkwN3wxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAyNXw5NmMxODkwN3wxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:37 上传

最终还是找到了`index.html` 狗头。

hook `document.addEventListener` 可以改写`initializeTTSDemo`：

![](https://attach.52pojie.cn/forum/202203/19/103814svm2gmonkkwon7ty.png)

`image-20220318225605405.png` (165.83 KB, 下载次数: 255)

[href="forum.php?mod=attachment&aid=MjUwNTAyNnw3OGM5MWE1M3wxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAyNnw3OGM5MWE1M3wxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:38 上传

`initializeTTSDemo`中大部分都是抄的，核心的修改如下：

```
synthesizer.synthesisCompleted = function () {
                            // 此时已经合成完毕了 arguments[1].privResult.privAudioData 为MP3文件
                            //debugger;
                            window.xtw.privAudioData = arguments[1].privResult.privAudioData;
                            window.xtw.playdlbut.innerHTML = "下载音频";
                            window.xtw.playdlbut.onclick = function () {
                                // https://stackoverflow.com/questions/19327749/javascript-blob-filename-without-link
                                function saveFile(name, type, data) {
                                    if (data !== null && navigator.msSaveBlob) {
                                        return navigator.msSaveBlob(new Blob([data], { type: type }), name);
                                    }
                                    var a = $("");
                                    var url = window.URL.createObjectURL(new Blob([data], { type: type }));
                                    a.attr("href", url);
                                    a.attr("download", name);
                                    $("body").append(a);
                                    a[0].click();
                                    window.URL.revokeObjectURL(url);
                                    a.remove();
                                    window.xtw.playdlbut.innerHTML = "先点播放";
                                };
                                //debugger;
                                if (window.xtw.privAudioData) {
                                    saveFile(new Date().getTime(), "audio/mp3", window.xtw.privAudioData);
                                    window.xtw.privAudioData = null;
                                } else {
                                    alert('先点播放，等按钮变成下载音频再点击')
                                }
                            };
                            synthesizer.close();
                            synthesizer = null;
                        };
```

最后把按钮放在合适的位置，方便使用：

```
if (!window.xtw.playdlbut) {

                        window.xtw.playdlbut = document.createElement("button");
```
