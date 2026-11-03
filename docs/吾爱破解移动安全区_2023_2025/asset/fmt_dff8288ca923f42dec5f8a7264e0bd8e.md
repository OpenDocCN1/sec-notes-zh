
```
body: JSON.stringify({
    boxes,
    scores,
    classes,
}),
credentials: "include",
}).then((res) => res.json()).then((res) => {
    const {hint, labels, colors} = res;
    document.querySelector('#result').textContent = hint; // 错误时显示提示，正确时显示 flag
})
```

简而言之就是调用`TF.js`（`Tensorflow`，Google开发的一款深度学习框架）加载`yolo`目标检测模型，对上传的图片执行本地目标检测

然后将检测结果（`boxes` -> 【目标框】，`scores` 【置信度】，`classes` 【分类标签】）以`POST`形式传递至后端，接受后端返回的提示，并在图中框出来

emmm，这个玩玩就好了，修改也没啥意思，因为判断逻辑在后端，程序能帮你做的也只能是根据提示不断修改，意义不大（doge）

> flagC{ce92f978} 过期时间: 2024-02-18 10:10:00
