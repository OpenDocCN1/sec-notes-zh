
# 扫码洗衣设备服务端代码

```python
import requests
from flask import Flask, request, jsonify, render_template
import pymysql
from utils import PGSH

app = Flask(__name__)

# ------------------ MySQL 配置 ------------------
DB_CONFIG = {
    "host": "localhost",
    "user": "root",
    "password": "123456",
    "database": "pangguai_life",
    "charset": "utf8mb4"
}

# ------------------ 数据库操作工具 ------------------
def get_conn():
    """获取数据库连接"""
    return pymysql.connect(**DB_CONFIG)

def ensure_table_exists():
    """初始化表结构"""
    conn = get_conn()
    cur = conn.cursor()
    cur.execute("""
        CREATE TABLE IF NOT EXISTS washing_machine (
            machine_id CHAR(255) PRIMARY KEY,
            id CHAR(10),
            shopId CHAR(9),
            categoryCode CHAR(2),
            categoryName VARCHAR(255),
            name VARCHAR(255),
            orgName VARCHAR(255),
            imei VARCHAR(255)
        ) CHARACTER SET utf8mb4;
    """)
    conn.commit()
    conn.close()

# ------------------ 前端页面 ------------------
@app.route("/")
def index():
    return render_template("index.html")

# ------------------ 扫码接口 ------------------
@app.route("/scan", methods=["POST"])
def scan_qr():
    data = request.get_json()
    nqt_value = data.get("nqt")

    if not nqt_value:
        return jsonify({"code": 400, "msg": "缺少NQT参数"})

    scan_data = pgsh.qr_scan(nqt_value)

    if scan_data.get("code") != 0:
        return jsonify({"code": 500, "msg": "扫码失败"})

    d = scan_data["data"]

    detail = pgsh.qr_details(d["id"])

    t = detail['data']

    machine_id = nqt_value

    conn = get_conn()
    cur = conn.cursor()

    # 检查是否存在
    cur.execute("SELECT 1 FROM washing_machine WHERE machine_id=%s", (machine_id,))
    if cur.fetchone():
        conn.close()
        return jsonify({"code": 200, "msg": "该机器已存在"})

    # 插入新数据
    cur.execute("""
        INSERT INTO washing_machine (machine_id, id, shopId, categoryCode, categoryName, name, orgName, imei)
        VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
    """, (machine_id, d["id"], str(d["shopId"]), d["categoryCode"], d["categoryName"], t["name"], t["orgName"], t["imei"]))
    conn.commit()
    conn.close()

    return jsonify({"code": 0, "msg": "保存成功"})

# ------------------ 设备列表接口 ------------------
@app.route("/api/machines")
def api_machines():
    conn = get_conn()
    cur = conn.cursor(pymysql.cursors.DictCursor)
    cur.execute("SELECT * FROM washing_machine")
    machines = cur.fetchall()
    conn.close()

    # 判断是否工作
    for m in machines:
        try:
            detail = pgsh.qr_details(m["id"])
            t = detail['data']
            m["status"] = "工作中" if t.get("deviceErrorCode") == 2 else "空闲"
        except:
            m["status"] = "未知"

    return jsonify(machines)

# ------------------ 桶自洁接口 ------------------
@app.route("/self_clean", methods=["POST"])
def self_clean():
    data = request.get_json()
    machine_id = data.get("machine_id")
    if not machine_id:
        return jsonify({"code": 400, "msg": "缺少machine_id"})

    try:
        result = pgsh.self_clean(machine_id)
        # 假设 pgsh.self_clean 返回 {"code": 0, "msg": "..."}
        if result.get("code") == 0:
            return jsonify({"code": 0, "msg": "自洁启动成功"})
        else:
            return jsonify({"code": 500, "msg": result.get("msg", "启动失败")})
    except Exception as e:
        return jsonify({"code": 500, "msg": f"启动异常: {str(e)}"})

if __name__ == "__main__":
    ensure_table_exists()
    pgsh = PGSH()
    app.run(host='0.0.0.0', debug=True)
```

## 数据库构建

数据库构建的也比较简单，如下

![](https://attach.52pojie.cn/forum/202511/05/122527ydmm8dvcq6mzowa6.png)

`image.png` (28.95 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMTYwNXwzOWI2YmY0YnwxNzkzNTc3MjcxfDE2ODA4NnwyMDY5OTc5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMTYwNXwzOWI2YmY0YnwxNzkzNTc3MjcxfDE2ODA4NnwyMDY5OTc5&nothumb=yes)

2025-11-5 12:25 上传

![](https://attach.52pojie.cn/forum/202511/05/122535e4gpz54u25vzvu54.png)

`image.png` (62.81 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMTYwNnw1MWQzMGQyMnwxNzkzNTc3MjcxfDE2ODA4NnwyMDY5OTc5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMTYwNnw1MWQzMGQyMnwxNzkzNTc3MjcxfDE2ODA4NnwyMDY5OTc5&nothumb=yes)

2025-11-5 12:25 上传

## 定时任务

由于接下来一段时间事情会有点多，所以定时任务就不再实现，原理实际上很容易，有兴趣的可以自己实现一下
