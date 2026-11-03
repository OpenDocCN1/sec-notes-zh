
```
import xml.etree.ElementTree as ET
import json
import numpy as np
from fastdtw import fastdtw
from scipy.spatial.distance import euclidean
from tqdm import tqdm

def parse_glyphs(file_path):
    """
    解析字体文件中的 TTGlyph 信息
    """
    tree = ET.parse(file_path)
    root = tree.getroot()

    glyphs = {}

    for glyph in root.findall(".//TTGlyph"):
        name = glyph.get("name")
        points = []
        for pt in glyph.findall(".//pt"):
            x = int(pt.get("x"))
            y = int(pt.get("y"))
            on = int(pt.get("on", 0))  # 默认值为0，如果不存在则设为0
            points.append((x, y))

        # 将点集转换为字符串，作为字典的键
        key = str(points)
        glyphs[key] = name

    return glyphs

def get_unicode_character(name):
    """
    根据 glyph 名称（如 uni5148）获取对应汉字
    """
    if name.startswith("uni"):
        try:
            unicode_value = int(name[3:], 16)
            return chr(unicode_value)
        except ValueError:
            return None
    return None

def normalize_points(points):
    """
    归一化点集
    """
    if not points:  # 检查点集是否为空
        return []

    points = np.array(points)  # 将点集转换为NumPy数组
    min_x, min_y = np.min(points, axis=0)
    max_x, max_y = np.max(points, axis=0)

    # 防止除以零
    if max_x == min_x:
        max_x = min_x + 1
    if max_y == min_y:
        max_y = min_y + 1

    normalized_points = (points - [min_x, min_y]) / [max_x - min_x, max_y - min_y]
    return normalized_points

def calculate_similarity(points1, points2):
    """
    使用DTW计算两个点集之间的相似度
    """
    points1_normalized = normalize_points(points1)
    points2_normalized = normalize_points(points2)

    if len(points1_normalized) == 0 or len(points2_normalized) == 0:
        return 0.0  # 如果任一点集为空，相似度为0

    distance, path = fastdtw(points1_normalized, points2_normalized, dist=euclidean)

    aligned_points1 = [points1_normalized[i] for i, _ in path]
    aligned_points2 = [points2_normalized[j] for _, j in path]

    distances = [np.linalg.norm(np.array(p1) - np.array(p2)) for p1, p2 in zip(aligned_points1, aligned_points2)]
    average_distance = np.mean(distances)

    similarity_score = 1 / (1 + average_distance)
    return similarity_score

def build_mapping(xml_old_path, xml_cx_path):
    """
    建立思源黑体和超星字体的对照关系
    """
    old_glyphs = parse_glyphs(xml_old_path)
    print(f'思源字体：{len(old_glyphs)}')
    cx_glyphs = parse_glyphs(xml_cx_path)
    print(f'超星字体：{len(cx_glyphs)}')

    mapping = []

    total_combinations = len(old_glyphs) * len(cx_glyphs)
    with tqdm(total=total_combinations, desc="Processing") as pbar:
        for old_key, old_name in old_glyphs.items():
            for cx_key, cx_name in cx_glyphs.items():
                similarity = calculate_similarity(eval(old_key), eval(cx_key))
                if similarity >= 0.9:
                    mapping.append({
                        "chaoxing": {
                            "cx_name": cx_name,
                            "cx_character": get_unicode_character(cx_name)
                        },
                        "si_yuan": {
                            "sy_name": old_name,
                            "sy_character": get_unicode_character(old_name)
                        },
                        "similarity": similarity
                    })
                pbar.update(1)

    return mapping

if __name__ == "__main__":
    xml_old_path = r"D:\UserData\Desktop\思源黑体.xml"
    xml_cx_path = r"D:\UserData\Desktop\chaoxing_font.xml"

    result = build_mapping(xml_old_path, xml_cx_path)

    # 输出到文件
    with open("glyph_mapping2.json", "w", encoding="utf-8") as f:
        json.dump(result, f, ensure_ascii=False, indent=4)

    # print(json.dumps(result[:5], ensure_ascii=False, indent=4))
```

但是运行效果不如人意

![](https://blog.ittoolman.com/usr/uploads/2024/11/409476519.png)

这么长的时间肯定是不能忍的，所有采用多线程的处理方式，cpu就应该忙起来了。

```
from concurrent.futures import ProcessPoolExecutor, as_completed

import json

import numpy as np

from fastdtw import fastdtw

from scipy.spatial.distance import euclidean

from tqdm import tqdm

import xml.etree.ElementTree as ET

# 其他函数不变，保持之前的代码

def calculate_similarity(points1, points2):
    """
    使用DTW计算两个点集之间的相似度
    """
    points1_normalized = normalize_points(points1)
    points2_normalized = normalize_points(points2)

    if len(points1_normalized) == 0 or len(points2_normalized) == 0:
        return 0.0  # 如果任一点集为空，相似度为0

    distance, path = fastdtw(points1_normalized, points2_normalized, dist=euclidean)

    aligned_points1 = [points1_normalized[i] for i, _ in path]
    aligned_points2 = [points2_normalized[j] for _, j in path]

    distances = [np.linalg.norm(np.array(p1) - np.array(p2)) for p1, p2 in zip(aligned_points1, aligned_points2)]
    average_distance = np.mean(distances)

    similarity_score = 1 / (1 + average_distance)
    return similarity_score

def normalize_points(points):
    """
    归一化点集
    """
    if not points:  # 检查点集是否为空
        return []

    points = np.array(points)  # 将点集转换为NumPy数组
    min_x, min_y = np.min(points, axis=0)
    max_x, max_y = np.max(points, axis=0)

    # 防止除以零
    if max_x == min_x:
        max_x = min_x + 1
    if max_y == min_y:
        max_y = min_y + 1

    normalized_points = (points - [min_x, min_y]) / [max_x - min_x, max_y - min_y]
    return normalized_points

def parallel_calculate_similarity(old_key, old_name, cx_glyphs):
    """
    并行计算相似度
    """
    results = []

    for cx_key, cx_name in cx_glyphs.items():
        similarity = calculate_similarity(eval(old_key), eval(cx_key))

        if similarity >= 0.9:
            results.append({
                "chaoxing": {
                    "cx_name": cx_name,
                    "cx_character": get_unicode_character(cx_name)
                },
                "si_yuan": {
                    "sy_name": old_name,
                    "sy_character": get_unicode_character(old_name)
                },
                "similarity": similarity
            })

    return results

def get_unicode_character(name):
    """
    根据 glyph 名称（如 uni5148）获取对应汉字
    """
    if name.startswith("uni"):
        try:
            unicode_value = int(name[3:], 16)
            return chr(unicode_value)
        except ValueError:
            return None

    return None

def parse_glyphs(file_path):
    """
    解析字体文件中的 TTGlyph 信息
    """
    tree = ET.parse(file_path)
    root = tree.getroot()

    glyphs = {}

    for glyph in root.findall(".//TTGlyph"):
        name = glyph.get("name")
        points = []

        for pt in glyph.findall(".//pt"):
            x = int(pt.get("x"))
            y = int(pt.get("y"))
            on = int(pt.get("on", 0))  # 默认值为0，如果不存在则设为0
            points.append((x, y))

        # 将点集转换为字符串，作为字典的键
        key = str(points)
        glyphs[key] = name

    return glyphs

def build_mapping_parallel(xml_old_path, xml_cx_path):
    """
    并行建立思源黑体和超星字体的对照关系
    """
    old_glyphs = parse_glyphs(xml_old_path)
    print(f'思源字体：{len(old_glyphs)}')
    cx_glyphs = parse_glyphs(xml_cx_path)
    print(f'超星字体：{len(cx_glyphs)}')

    mapping = []

    # 使用进程池进行并行处理
    with ProcessPoolExecutor() as executor:
        futures = []

        # 为每个思源字体字形提交任务
        for old_key, old_name in old_glyphs.items():
            futures.append(executor.submit(parallel_calculate_similarity, old_key, old_name, cx_glyphs))

        # 通过 as_completed 获取计算结果
        for future in tqdm(as_completed(futures), total=len(futures), desc="Processing"):
            mapping.extend(future.result())

    return mapping

if __name__ == "__main__":
    xml_old_path = r"D:\UserData\Desktop\思源黑体.xml"
    xml_cx_path = r"D:\UserData\Desktop\chaoxing_font.xml"

    result = build_mapping_parallel(xml_old_path, xml_cx_path)

    # 输出到文件
    with open("glyph_mapping_parallel.json", "w", encoding="utf-8") as f:
        json.dump(result, f, ensure_ascii=False, indent=4)

    # 打印部分结果
```
