
[forum.php?mod=attachment&aid=MjY2NjA0NHxiOTM5YmM4ZnwxNzkzNTcyNzk4fDE2ODA4NnwxODcyNjM4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2NjA0NHxiOTM5YmM4ZnwxNzkzNTcyNzk4fDE2ODA4NnwxODcyNjM4&nothumb=yes)

2023-12-24 20:22 上传

按 `W` 键创建一个标记框，框起来验证区域的字。标注完一张图片后，记得按 `Ctrl + S` 保存。按 `D` 进入下一张图片，按 `A` 进入前一张图片。

> 我们在目标检测阶段只区分是不是字，不区分具体是什么字，所以标签只有一个 `text`。
>
> 下方点击顺序的四个字位置是固定的，所以后期我们统一进行自动化标注。

![](https://attach.52pojie.cn/forum/202312/24/202207ntgcog02jjw04iuw.png)

17.png (139.55 KB, 下载次数: 1)

[forum.php?mod=attachment&aid=MjY2NjA0NXxhNGMyY2EwZHwxNzkzNTcyNzk4fDE2ODA4NnwxODcyNjM4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2NjA0NXxhNGMyY2EwZHwxNzkzNTcyNzk4fDE2ODA4NnwxODcyNjM4&nothumb=yes)

2023-12-24 20:22 上传

## 使用YOLOv8训练目标检测模型

使用`YOLOv8`前，确保你的电脑有N卡，并[参考此博客](https://blog.csdn.net/qq_50677040/article/details/132131346)配置好`CUDA+cudnn`相关深度学习的环境，安装完上述环境，再安装Python包`ultralytics`即可。

### 目标检测模型训练配置yaml

```yaml
# 指定训练集、验证集、测试集路径
train: /path/to/your/train/images
val: /path/to/your/valid/images
test: /path/to/your/test/images
# 有多少个分类
nc: 1
# 分类分别是什么
names: ['text']
```

### 目标检测模型训练代码

```python
from ultralytics import YOLO

# 预训练模型下载 https://github.com/ultralytics/ultralytics?tab=readme-ov-file#models

model = YOLO("./model/yolov8n.pt", task="detect")

model.train(data="./dataset/detect.yaml", epochs=20, cache=True, imgsz=320, batch=16, workers=0, device=0)
```

### 导出onnx模型

```python
model = YOLO('./path/to/your/train/output/best.pt', task='detect')
model.export(format='onnx', imgsz=320, simplify=True)
```

### 模型使用

```python
model = YOLO(model='./path/to/your/model/best.onnx', task='detect')

results = model.predict(source='./path/to/your/test/dataset/images/folder', show=False, save=True, imgsz=500, device=0)

print(f"total_标签名字：{results[0].names}")

def y8_detect_xy(result):
   """
   输出坐标信息
   :param result:
   :return:
   """
   cls_xy = list()
   cls_dict = result.names
   cls_all = result.boxes.cls.tolist()
   print(f">>识别结果目标类 {len(cls_all)}个: {cls_all}")
   xyxy_all = result.boxes.xyxy.tolist()
   for i in range(len(cls_all)):
       label_name = cls_dict[int(cls_all[i])]
       box_xyxy = xyxy_all[i]
       box_mid_xy = [(box_xyxy[0] + box_xyxy[2]) / 2, (box_xyxy[1] + box_xyxy[3]) / 2]
       # print(f"目标点{i}: 标签名字: {label_name}, 中心坐标：{box_mid_xy}, xyxy坐标：{box_xyxy}")
       cls_xy.append({
           "label_id": i, "label_name": label_name,
           "box_mid_xy": box_mid_xy, "xyxy": box_xyxy
       })
   print(f"==识别结果: {cls_xy}==")
   return cls_xy

for result in results:
   y8_detect_xy(result)
```

### 训练效果

![](https://attach.52pojie.cn/forum/202312/24/202210quxhikxjijqx63xc.png)

18.png (1.17 MB, 下载次数: 2)

[forum.php?mod=attachment&aid=MjY2NjA0NnxhNDg4Mzc4NHwxNzkzNTcyNzk4fDE2ODA4NnwxODcyNjM4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2NjA0NnxhNDg4Mzc4NHwxNzkzNTcyNzk4fDE2ODA4NnwxODcyNjM4&nothumb=yes)

2023-12-24 20:22 上传

## 二、孪生神经网络训练

目的：

- 判断待点击字在大图中出现的位置
- 在大图中按顺序找出小图中的文字

步骤：

- 进行目标检测，裁剪预测的结果小图

剪裁小图代码

```python
def extract_correct_word():
   save_to = '/path/to/save/word'
   if not os.path.exists(save_to):
       os.mkdir(save_to)
   images = glob('./cut/images/*.png')
   for image_path in tqdm(images):
       results = model.predict(source=image_path, show=False, save=True, imgsz=500, device=0)
       num = image_path.split('/')[-1].split('.')[0].split('_')[1]
       im = np.asarray(Image.open(image_path).convert("RGB"))
       for result in results:
           labels = y8_detect_xy(result)
           for i, label in enumerate(tqdm(labels)):
               x1, y1, x2, y2 = label["xyxy"]
               word_im = im[math.floor(y1):math.ceil(y2), math.floor(x1):math.ceil(x2)]
               cv2.imwrite(os.path.join(save_to, 'x-{}-{}.png'.format(num, i)), word_im)
```

人工标注后整理，效果如下

![](https://attach.52pojie.cn/forum/202312/24/202212kfa92997oeq2hz8k.png)

19.png (87.09 KB, 下载次数: 2)

[forum.php?mod=attachment&aid=MjY2NjA0N3wxMGU1ZDEwNHwxNzkzNTcyNzk4fDE2ODA4NnwxODcyNjM4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2NjA0N3wxMGU1ZDEwNHwxNzkzNTcyNzk4fDE2ODA4NnwxODcyNjM4&nothumb=yes)

2023-12-24 20:22 上传

- 进行孪生神经网络训练

训练代码

![](https://attach.52pojie.cn/forum/202312/24/202214fmczo70giouci232.png)

20.png (90.05 KB, 下载次数: 1)

[forum.php?mod=attachment&aid=MjY2NjA0OHw3M2M1MTMxN3wxNzkzNTcyNzk4fDE2ODA4NnwxODcyNjM4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2NjA0OHw3M2M1MTMxN3wxNzkzNTcyNzk4fDE2ODA4NnwxODcyNjM4&nothumb=yes)

2023-12-24 20:22 上传

预测结果

![](https://attach.52pojie.cn/forum/202312/24/202216jmy26k5kdfsxjyj4.png)

21.png (177.24 KB, 下载次数: 0)

[forum.php?mod=attachment&aid=MjY2NjA0OXwyMzQwOTY4MXwxNzkzNTcyNzk4fDE2ODA4NnwxODcyNjM4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2NjA0OXwyMzQwOTY4MXwxNzkzNTcyNzk4fDE2ODA4NnwxODcyNjM4&nothumb=yes)

2023-12-24 20:22 上传

### 导出onnx模型

```python
def pth2onnx():
   device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
   model_path = r"/path/to/your/train/output/model.pth"
   model = Siamese((105, 105, 3))
   model.load_state_dict(torch.load(model_path, map_location=device))
   model.eval()
   dummy_input = [torch.randn(1, 3, 105, 105), torch.randn(1, 3, 105, 105)]
   torch.onnx.export(
       model, dummy_input, 'weights_self_12071503.onnx', verbose=True, input_names=['x1', 'x2'], output_names=['output']
   )
   print("Successful!")
```

## 三、模型融合使用

目的：

- 在验证点击区，依次找出待点击的文字

核心代码：

```python
def judge_max_sim(self, cls_xys, img):
    targets = [(i["xyxy"], i["box_mid_xy"]) for i in cls_xys if i["label_name"] == "target"]
    target_chars = []
    # 将提供的顺序，按照x坐标轴从小到大排序，确保识别的字符顺序正确
    targets = sorted(targets, key=lambda x: x[0][0])
    for xyxy, _ in targets:
        target_chars.append(img.crop(xyxy))

    texts = [(i["xyxy"], i["box_mid_xy"]) for i in cls_xys if i["label_name"] == "text"]
    text_chars = []
    for xyxy, _ in texts:
        text_chars.append(img.crop(xyxy))

    # 获取点击顺序
    click_seq_result = []
    for m, target_img in enumerate(target_chars):
        slys = []
        if len(texts) == 0:
            break
        elif len(texts) == 1:
            slys_index = 0
        else:
            for n, text_img in enumerate(text_chars):
                similarity = self.siam_model.predict(target_img, text_img)
                slys.append(similarity)
            slys_index = slys.index(max(slys))
        click_seq_result.append(texts[slys_index][1])
        texts.pop(slys_index)
        text_chars.pop(slys_index)
        if len(texts) == 0:
            break
    return click_seq_result

def get_xy_seq(self, img_path):
    """
    获取点击顺序
    :param img_path:
    :return:
    """
    img = self.open_image(img_path)
    # yolov8识别
    obj_result = self.yolo_v8_model_predict(img_path)[0]
    cls_xys = self.target_detection_xy(obj_result)
    # 孪生判断相似度
    xy_seq = self.judge_max_sim(cls_xys, img)
    return xy_seq
```

### 结果验证

![](https://attach.52pojie.cn/forum/202312/24/202218yrokmtqk9ymnj77y.png)

22.png (452.02 KB, 下载次数: 3)

[forum.php?mod=attachment&aid=MjY2NjA1MHxmNmFkMThkMHwxNzkzNTcyNzk4fDE2ODA4NnwxODcyNjM4&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2NjA1MHxmNmFkMThkMHwxNzkzNTcyNzk4fDE2ODA4NnwxODcyNjM4&nothumb=yes)

2023-12-24 20:22 上传
