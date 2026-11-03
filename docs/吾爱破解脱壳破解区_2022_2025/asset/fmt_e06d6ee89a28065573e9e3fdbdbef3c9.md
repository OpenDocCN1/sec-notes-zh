
# 视频倍速注入脚本

## 前端播放控制脚本

```
(function () {
  let rateList = [1.0, 1.5, 2.0];
  let currentRateIndex = 0;
  let currentRate = rateList[currentRateIndex];

  // 右上角 - 播放进度面板
  const infoPanel = document.createElement('div');
  infoPanel.style.position = 'fixed';
  infoPanel.style.top = '10px';
  infoPanel.style.left = '10px';
  infoPanel.style.background = 'rgba(0, 0, 0, 0.7)';
  infoPanel.style.color = 'white';
  infoPanel.style.padding = '6px 12px';
  infoPanel.style.borderRadius = '6px';
  infoPanel.style.fontSize = '14px';
  infoPanel.style.fontFamily = 'monospace';
  infoPanel.style.zIndex = 999999;
  document.body.appendChild(infoPanel);

  function formatTime(seconds) {
    if (isNaN(seconds)) return '--:--';
    const m = Math.floor(seconds / 60).toString().padStart(2, '0');
    const s = Math.floor(seconds % 60).toString().padStart(2, '0');
    return `${m}:${s}`;
  }

  // 中间提示用 Toast
  const toast = document.createElement('div');
  toast.style.position = 'fixed';
  toast.style.top = '50px';
  toast.style.left = '50%';
  toast.style.transform = 'translateX(-50%)';
  toast.style.background = 'rgba(0, 0, 0, 0.7)';
  toast.style.color = 'white';
  toast.style.padding = '8px 16px';
  toast.style.borderRadius = '8px';
  toast.style.fontSize = '16px';
  toast.style.zIndex = 999999;
  toast.style.transition = 'opacity 0.3s ease';
  toast.style.opacity = '0';
  toast.innerText = '';
  document.body.appendChild(toast);

  function showToast(message) {
    toast.innerText = message;
    toast.style.opacity = '1';
    clearTimeout(toast._timeout);
    toast._timeout = setTimeout(() => {
      toast.style.opacity = '0';
    }, 1500);
  }

  function getPlayingVideo() {
    const videos = document.querySelectorAll('.--player-video-root video');
    for (const video of videos) {
      const style = getComputedStyle(video);
      if (style.opacity === '1' && style.zIndex === '-10000') {
        return video;
      }
    }
    return null;
  }

  // 每秒更新播放信息 + 设置倍速
  setInterval(() => {
    const video = getPlayingVideo();
    if (video) {
      // 强制 playbackRate
      if (video.playbackRate !== currentRate) {
        video.playbackRate = currentRate;
      }

      // 更新 info 面板
      infoPanel.innerText = `播放进度：${formatTime(video.currentTime)} / ${formatTime(video.duration)}\n播放速度：${currentRate}x`;
    } else {
      infoPanel.innerText = `未检测到正在播放的视频`;
    }
  }, 1000);

  // 快捷键：R = 切换速度，←/→ = 快进/快退
  document.addEventListener('keydown', (e) => {
    const video = getPlayingVideo();
    if (!video) return;

    if (e.key === 'ArrowLeft') {
      video.currentTime = Math.max(0, video.currentTime - 5);
      showToast(`⏪ 后退 5 秒：${formatTime(video.currentTime)}`);
    } else if (e.key === 'ArrowRight') {
      video.currentTime = Math.min(video.duration, video.currentTime + 5);
      showToast(`⏩ 前进 5 秒：${formatTime(video.currentTime)}`);
    } else if (e.key.toLowerCase() === 'r') {
      currentRateIndex = (currentRateIndex + 1) % rateList.length;
      currentRate = rateList[currentRateIndex];
      showToast(`当前播放速度：${currentRate}x`);
    }
  });
})();
```

然后我们再将这些代码整合到`main.loader.js`中。

## Electron 主进程注入代码

```
require('bytenode');

const path = require('node:path');

const arch = process.arch;

const { app } = require('electron');

app.commandLine.appendSwitch('remote-debugging-port', '9222');

app.commandLine.appendSwitch('remote-allow-origins', '*');

const injectionScript = `(${function () {

  (function () {

    let rateList = [1.0, 1.5, 2.0, 3.0];

    let currentRateIndex = 0;

    let currentRate = rateList[currentRateIndex];

    // 右上角 - 播放进度面板

    const infoPanel = document.createElement('div');

    infoPanel.style.position = 'fixed';

    infoPanel.style.top = '10px';

    infoPanel.style.left = '10px';

    infoPanel.style.background = 'rgba(0, 0, 0, 0.7)';

    infoPanel.style.color = 'white';

    infoPanel.style.padding = '6px 12px';

    infoPanel.style.borderRadius = '6px';

    infoPanel.style.fontSize = '14px';

    infoPanel.style.fontFamily = 'monospace';

    infoPanel.style.zIndex = 999999;

    document.body.appendChild(infoPanel);

    function formatTime(seconds) {

      if (isNaN(seconds)) return '--:--';

      const m = Math.floor(seconds / 60).toString().padStart(2, '0');

      const s = Math.floor(seconds % 60).toString().padStart(2, '0');

      return `${m}:${s}`;

    }

    // 中间提示用 Toast

    const toast = document.createElement('div');

    toast.style.position = 'fixed';

    toast.style.top = '10px';

    toast.style.left = '50%';

    toast.style.transform = 'translateX(-50%)';

    toast.style.background = 'rgba(0, 0, 0, 0.7)';

    toast.style.color = 'white';

    toast.style.padding = '8px 16px';

    toast.style.borderRadius = '8px';

    toast.style.fontSize = '16px';

    toast.style.zIndex = 999999;

    toast.style.transition = 'opacity 0.3s ease';

    toast.style.opacity = '0';

    toast.innerText = '';

    document.body.appendChild(toast);

    function showToast(message) {

      toast.innerText = message;

      toast.style.opacity = '1';

      clearTimeout(toast._timeout);

      toast._timeout = setTimeout(() => {

        toast.style.opacity = '0';

      }, 1500);

    }

    function getPlayingVideo() {

      const videos = document.querySelectorAll('.--player-video-root video');

      for (const video of videos) {

        const style = getComputedStyle(video);

        if (style.opacity === '1' && style.zIndex === '-10000') {

          return video;

        }

      }

      return null;

    }

    // 每秒更新播放信息 + 设置倍速

    setInterval(() => {

      const video = getPlayingVideo();

      if (video) {

        // 强制 playbackRate

        if (video.playbackRate !== currentRate) {

          video.playbackRate = currentRate;

        }

        // 更新 info 面板

        infoPanel.innerText = `播放进度：${formatTime(video.currentTime)} / ${formatTime(video.duration)}\n播放速度：${currentRate}x`;

      } else {

        infoPanel.innerText = `未检测到正在播放的视频`;

      }

    }, 1000);

    // 快捷键：R = 切换速度，←/→ = 快进/快退

    document.addEventListener('keydown', (e) => {

      const video = getPlayingVideo();

      if (!video) return;

      if (e.key === 'ArrowLeft') {

        video.currentTime = Math.max(0, video.currentTime - 5);

        showToast(`⏪ 后退 5 秒：${formatTime(video.currentTime)}`);

      } else if (e.key === 'ArrowRight') {

        video.currentTime = Math.min(video.duration, video.currentTime + 5);

        showToast(`⏩ 前进 5 秒：${formatTime(video.currentTime)}`);

      } else if (e.key.toLowerCase() === 'r') {

        currentRateIndex = (currentRateIndex + 1) % rateList.length;

        currentRate = rateList[currentRateIndex];

        showToast(`当前播放速度：${currentRate}x`);

      }

    });

  })();  

}})()`;

app.on('web-contents-created', (event, contents) => {

  contents.on('did-finish-load', () => {

    contents.executeJavaScript(injectionScript)

      .then(() => console.log('[注入] 成功注入 video 控制脚本'))

      .catch(err => console.error('[注入失败]', err));

  });

});

require(path.join(__dirname, './bin/main-darwin-' + arch + '.jsc'));
```

![](https://attach.52pojie.cn/forum/202506/23/175859t3g9e6j8yxlye8mu.png)

image-20250623174818378.png (539.38 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NDcxMnwxNjdlNzQwNHwxNzkzNTcwODY0fDE2ODA4NnwyMDQwOTc1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDcxMnwxNjdlNzQwNHwxNzkzNTcwODY0fDE2ODA4NnwyMDQwOTc1&nothumb=yes)

2025-6-23 17:58 上传

一切大功告成，直接使用`steam`打开游戏后，就会自动注入代码，可以随时倍速播放了！

所有功能都正常，但有时候倍速播放可能会导致比较短的循环视频黑屏，但无所谓了，那些也不影响刷剧情~
