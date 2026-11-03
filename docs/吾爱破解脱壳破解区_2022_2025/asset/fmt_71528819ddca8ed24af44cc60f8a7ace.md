
```python
        queryString = queryString[:-1]

        captchaResult = session.post(f"https://zyzfx.psyyun.com/code/check?{queryString}").json()

        if captchaResult["data"]["success"] == True:

            # # AES CBC Encrypt Password

            encrypted_password_b64 = aes_cbc_zero_padding_encrypt(Password, "ruigeruigeruigee")

            # Login

            # 将URL参数转换为JSON格式再编码为查询字符串

            params = {

                "randomStr": "blockPuzzle",

                "code": encryptedCodeBase64,

                "grant_type": "password",

                "username": UserName

            }

            # 手动构建查询字符串，确保特殊字符被正确编码

            query_string = "&".join([f"{k}={urllib.parse.quote(str(v), safe='')}" for k, v in params.items()])

            url = f"https://zyzfx.psyyun.com/auth/oauth/token?{query_string}"

            session.headers.update({"Content-Type":"application/x-www-form-urlencoded","ignore":"1","origin":"https://zyzfx.psyyun.com"})

            response = session.post(url, data={"username":UserName,"password": encrypted_password_b64})

            loginResult = response.json()

            print(f"登录成功,用户名：{loginResult['user_info']['username']}")

            session.headers.update({"Authorization":f"Bearer {response.json()['access_token']}"})

            break
```

## 返回内容

### 登录失败

```
{"code":1,"msg":"登录失败，用户名或密码错误","data":"unauthorized"}
```

### 登录成功

![](https://attach.52pojie.cn//forum/202509/27/163159zmmdmv9kjvnxkxmi.png?l)

> 

其中需要添加请求头`Authorization: Bearer access_token`内容
添加这个请求头后即可请求接下来的如获取心理测评表等API操作
之后就是答题部分(抓包即可抓下来),`Python`代码(自动完成题目):

```
DefaultSelectIndex = 0 # 默认选择选项
Login("UserName","Password") // 用户名与密码
# 心理测评表获取(空代表完成)
getSacleTable = session.get("https://zyzfx.psyyun.com/plantest/userPlan/getMyUpscomingScaleTabByPage?source=2&type=1¤t=1&size=10").json()
if getSacleTable["data"]["records"] != []:
                print(f"正在测评: {i['studentName']}")
                for j in getSacleTable["data"]["records"]:
                                print(f"测评名称: {j['planTitle']}")
                                print(f"测评ID: {j['id']}")
                                # Get Test List
                                getTestList = session.get(f"https://zyzfx.psyyun.com/plantest/userPlan/getMyPlanTestScaleByPageAndPlanId?planId={j['id']}").json()
                                for k in getTestList["data"]:
                                                print(f"测评名称:{k['otherName']}")
                                                print(f"测评ID: {k['testId']}")
                                                print(f"ScaleID: {k['scaleId']}")
                                                getQuestionCount = int(k["questionCount"])
                                                page = getQuestionCount / 10
                                                print("开始答题")
                                                CommitList = []
                                                for m in range(int(page + 1)):
                                                                getQuestionList = session.get(f"https://zyzfx.psyyun.com/scale/question/getMyQuestionAndOptionByscaleId?scaleId={k['scaleId']}&testId={k['testId']}&pushId=0¤t={str(m)}").json()
                                                                for n in getQuestionList["data"]["records"]:
                                                                                getDefaultAnswer = n["optionList"][DefaultSelectIndex]
                                                                                CommitList.append({"questionId":int(getDefaultAnswer["questionId"]),"answerId":int(getDefaultAnswer["id"]),"answerContent":getDefaultAnswer["answerContent"],"answerType":1,"score":int(getDefaultAnswer["score"])})
                                                buildCommitJson = {
                                                                "scaleId": int(k["scaleId"]),
                                                                "testId": int(k["testId"]),
                                                                "studentId": 0,
                                                                "useTime": 450,
                                                                "isFinish": True,#m == page - 1,
                                                                "current": int(CommitList[-1]["questionId"]),
                                                                "pushId": 0,
                                                                "answerList": CommitList,
                                                                "visitorId": None
                                                }
                                                # Commit
                                                session.headers.update({"Content-Type":"application/json"})
                                                commitResult = session.post("https://zyzfx.psyyun.com/plantest/userPlan/commitAnswer",json=buildCommitJson).json()
                                                print(f"提交结果: {commitResult}")
                                                print(f"Complete: {k['otherName']}")
```

## 效果展示

正在测评: xxx
测评名称: 2025年秋季心理测评
测评ID: 3329
测评名称:心理测试2
测评ID: 3329
ScaleID: 11470
开始答题
提交结果: {'code': 0, 'msg': '成功', 'data': None, 'success': True}
Complete: 心理测试2
测评名称:心理测试1
测评ID: 3329
ScaleID: 326
开始答题
提交结果: {'code': 0, 'msg': '成功', 'data': None, 'success': True}
Complete: 心理测试1
进程已结束，退出代码为 0
