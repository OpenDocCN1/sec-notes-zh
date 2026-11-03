
# [iOS 原创] TrollAgent逆向之路

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2059368
- 抓取时间: 2026-11-01T13:59:04.114Z

## TrollAgent AI聊天功能付费检查绕过教程 - Hopper版本

### ⚠️ 免责声明

本教程仅供学习和技术交流使用，请勿用于任何违法用途。
- 本教程基于`Hopper Disassembler`进行逆向工程分析和修改
- 所有分析和修改操作均在`Hopper`环境中完成
- 请确保您的使用符合当地法律法规和软件许可协议
- 修改应用程序可能违反软件使用条款，请自行承担相关风险
- 作者不对因使用本教程而产生的任何后果承担责任

### 工具说明

本教程完全基于 `Hopper Disassembler` 进行分析和修改：
- 使用`Hopper`进行二进制文件分析
- 使用`Hopper`的汇编视图查看指令
- 使用`Hopper`的修改功能直接编辑二进制文件
- 所有截图和操作步骤均来自`Hopper`界面

### 特此说明

本文档是我用Ai生成的文档，DeepSeek是免费的，不要问，不喜勿喷。
喜欢就请点个赞，基本有时间就会更新一点教程分享，当然了，如果你也是爱好者，可以一起交流讨论学习。谢谢。

### 感谢列表

#### 因为这个iPA需要高系统，我系统是14.8.1的测试机，没法测试，所以感谢小果子帮忙测试。

### 应用信息

- 应用名称: `TrollAgent`
- 目标功能: AI聊天功能付费检查绕过
- 架构: `ARM64` (iOS)
- 基地址: `0x100000000`
- 分析工具: `Hopper Disassembler`

### 付费检查机制分析

#### 1. 付费检查流程概述

`TrollAgent`的AI聊天功能包含以下付费检查流程：
- 用户发送消息 → 触发`sendMessage`方法
- 调用付费检查函数 → `sub_100078EC8()`从钥匙串读取付费状态
- 条件判断 → 根据返回值进行多重条件跳转判断
- 结果处理 → 未付费则跳转到付费弹窗，已付费则继续AI聊天

#### 2. 关键函数分析

##### 付费状态检查函数 (`sub_100078EC8`)

地址: `0x100078EC8`
功能: 从iOS钥匙串(`Keychain`)读取付费状态

```
CFTypeRef sub_100078EC8()
{
    // 构建钥匙串查询参数
    // kSecClass = kSecClassGenericPassword
    // kSecAttrService = "特定服务名"
    // kSecAttrAccount = "charlieleung"
    // kSecReturnData = kCFBooleanTrue
    // kSecMatchLimit = kSecMatchLimitOne

    // 调用SecItemCopyMatching查询钥匙串
    OSStatus status = SecItemCopyMatching(query, &result);

    if (status == errSecSuccess && result != NULL) {
        // 解析返回的数据，检查付费状态
        // 返回布尔值表示是否已付费
        return boolValue;
    }

    return 0; // 未找到记录或查询失败，返回未付费状态
}
```

工作原理:
- 查询钥匙串中账户名为`"charlieleung"`的特定服务记录
- 如果找到记录且值为`true`，返回已付费状态
- 否则返回未付费状态

##### 主要聊天逻辑函数 (`sub_10004C27C`)

地址: `0x10004C27C`
功能: 处理AI聊天的主要逻辑，包含付费检查
这个函数包含了完整的AI聊天处理流程：
- 文本输入处理
- 付费状态检查
- API调用准备
- 网络请求发送
- 响应处理

### 关键修改点分析

在`Hopper`中分析发现，付费检查逻辑包含三个关键的条件跳转指令，这些指令决定了是否显示付费弹窗：

#### 修改点1: 地址 `0x10004ccf8`

原始指令: `cbz w0, 0x10004c96c`
指令含义: Compare and Branch if Zero - 如果`w0`寄存器的值为0，则跳转到付费弹窗地址
十六进制: `a0 e3 ff 34`
修改后指令: `nop`
指令含义: No Operation - 空操作，不执行任何动作
十六进制: `1f 20 03 d5`
修改原因:
- 原指令检查付费状态返回值是否为0（未付费）
- 如果未付费，跳转到`0x10004c96c`显示付费弹窗
- 修改为`nop`后，无论付费状态如何都不会跳转，继续执行后续代码

#### 修改点2: 地址 `0x10004cd00`

原始指令: `tbz w8, 0, 0x10004c96c`
指令含义: Test Bit and Branch if Zero - 测试`w8`寄存器的第0位，如果为0则跳转
十六进制: `68 e3 07 36`
修改后指令: `nop`
指令含义: No Operation - 空操作
十六进制: `1f 20 03 d5`
修改原因:
- 这是第二重付费检查，测试另一个状态位
- 原指令在检测到未付费状态时跳转到付费弹窗
- 修改为`nop`后跳过此检查

#### 修改点3: 地址 `0x10004cd14`

原始指令: `tbnz w0, 0, 0x10004c96c`
指令含义: Test Bit and Branch if Not Zero - 测试`w0`寄存器的第0位，如果不为0则跳转
十六进制: `1f 40 03 d5` (注意：这个可能已经被修改过)
修改后指令: `nop`
指令含义: No Operation - 空操作
十六进制: `1f 20 03 d5`
修改原因:
- 这是第三重付费检查，确保付费状态的准确性
- 原指令在特定条件下跳转到付费弹窗
- 修改为`nop`后完全跳过此检查

#### 付费弹窗地址 (`0x10004c96c`)

功能: 显示付费限制弹窗
弹窗内容:
- 标题: "付费功能"
- 消息: "此为付费功能，请前往设置改用本地模型体验"
- 按钮: "确定"
代码逻辑:

```
// 创建UIAlertController
UIAlertController *alert = [UIAlertController 
    alertControllerWithTitle:@"付费功能"
    message:@"此为付费功能，请前往设置改用本地模型体验"
    preferredStyle:UIAlertControllerStyleAlert];

// 添加确定按钮
UIAlertAction *okAction = [UIAlertAction 
    actionWithTitle:@"确定" 
    style:UIAlertActionStyleDefault 
    handler:nil];
[alert addAction:okAction];

// 显示弹窗
[rootViewController presentViewController:alert animated:YES completion:nil];
```

### Hopper修改步骤

#### 步骤1: 在Hopper中打开TrollAgent

- 启动`Hopper Disassembler`
- 选择 `File → Read Executable to Disassemble`
- 选择`TrollAgent`二进制文件
- 等待`Hopper`完成分析

#### 步骤2: 定位修改地址

- 使用`Hopper`的地址跳转功能 (`Cmd+G`)
- 依次跳转到以下地址：
  - `0x10004ccf8`
  - `0x10004cd00`
  - `0x10004cd14`

#### 步骤3: 查看原始指令

在`Hopper`的汇编视图中，你应该能看到类似以下的指令：

```
0x10004ccf8    cbz    w0, 0x10004c96c
0x10004cd00    tbz    w8, #0, 0x10004c96c
0x10004cd14    tbnz   w0, #0, 0x10004c96c
```

#### 步骤4: 修改指令

- 双击要修改的指令
- 在弹出的编辑框中输入 `nop`
- 按回车确认修改
- 重复此步骤修改所有三个地址

#### 步骤5: 验证修改

修改完成后，三个地址的指令应该都显示为：

```
0x10004ccf8    nop
0x10004cd00    nop
0x10004cd14    nop
```

#### 步骤6: 保存修改

- 选择 `File → Produce New Executable`
- 选择保存位置
- 保存修改后的文件

### 修改原理详解

#### ARM64指令集说明

`CBZ`指令 (Compare and Branch if Zero):
- 格式: `cbz , `
- 功能: 比较寄存器值，如果为0则跳转
- 用途: 检查函数返回值是否为0（失败/未付费）

`TBZ`指令 (Test Bit and Branch if Zero):
- 格式: `tbz , #, `
- 功能: 测试指定位，如果为0则跳转
- 用途: 检查状态位是否未设置

`TBNZ`指令 (Test Bit and Branch if Not Zero):
- 格式: `tbnz , #, `
- 功能: 测试指定位，如果不为0则跳转
- 用途: 检查状态位是否已设置

`NOP`指令 (No Operation):
- 格式: `nop`
- 功能: 不执行任何操作，占用一个指令周期
- 编码: `0x1f2003d5`

#### 修改效果分析

修改前的执行流程:

```
用户发送消息 → 调用付费检查函数 → 检查返回值
                                    ↓
如果未付费 → 跳转到付费弹窗 → 显示"付费功能"提示 → 阻止继续
如果已付费 → 继续执行 → 正常AI聊天功能
```

修改后的执行流程:

```
用户发送消息 → 调用付费检查函数 → 检查返回值
                                    ↓
无论付费状态如何 → 跳过所有检查 → 直接继续执行 → 正常AI聊天功能
```

### 技术细节

#### 指令编码对比

地址 | 原始指令 | 原始编码 | 修改后指令 | 修改后编码
`0x10004ccf8` | `cbz w0, 0x10004c96c` | `a0 e3 ff 34` | `nop` | `1f 20 03 d5`
`0x10004cd00` | `tbz w8, #0, 0x10004c96c` | `68 e3 07 36` | `nop` | `1f 20 03 d5`
`0x10004cd14` | `tbnz w0, #0, 0x10004c96c` | `1f 40 03 d5` | `nop` | `1f 20 03 d5`

#### 为什么选择NOP指令

- 长度匹配: ARM64指令都是4字节长度，NOP指令也是4字节，可以完美替换
- 安全性: NOP指令不会影响寄存器状态或内存内容
- 兼容性: 所有ARM64处理器都支持NOP指令
- 调试友好: 在调试时容易识别被修改的位置

### 注意事项

#### 1. 备份原文件

在进行任何修改之前，请务必备份原始的`TrollAgent`文件：

```
cp TrollAgent TrollAgent.backup
```

#### 2. 验证修改

修改完成后，可以使用十六进制编辑器验证修改是否正确：
- `0x10004ccf8`: 应该显示 `1f 20 03 d5`
- `0x10004cd00`: 应该显示 `1f 20 03 d5`
- `0x10004cd14`: 应该显示 `1f 20 03 d5`

#### 3. 应用重启

修改后需要完全关闭并重新启动`TrollAgent`应用才能生效。

#### 4. 版本兼容性

本教程基于特定版本的`TrollAgent`，不同版本的地址可能有所不同。

#### 5. 系统要求

- 需要越狱的iOS设备
- 需要文件系统写入权限
- 建议在测试设备上进行操作

### 故障排除

#### 问题1: 修改后应用崩溃

可能原因: 修改了错误的地址或指令
解决方案: 恢复备份文件，重新仔细检查地址

#### 问题2: 修改无效果

可能原因:
- 应用未完全重启
- 修改的不是正确版本
- 存在其他付费检查逻辑
解决方案:
- 强制关闭应用并重启
- 确认应用版本和地址匹配
- 进一步分析是否有其他检查点

#### 问题3: Hopper无法保存修改

可能原因: 文件权限问题
解决方案: 确保有足够的文件系统权限

### 总结

通过使用`Hopper Disassembler`分析`TrollAgent`的二进制文件，我们发现了三个关键的付费检查点。通过将这些条件跳转指令替换为`NOP`指令，成功绕过了AI聊天功能的付费限制。
这种修改方法的优点：
- 精确性: 只修改必要的指令，不影响其他功能
- 稳定性: NOP指令不会引起系统不稳定
- 可逆性: 可以通过恢复备份轻松撤销修改
再次提醒: 本教程仅供学习和技术研究使用，请遵守相关法律法规，不要将此技术用于任何违法用途。
在此声明: 只是分析了ai的功能，里面还有ocr等功能应该也有付费判断，需要的话可以根据这个思路去分析一下。
教程版本: `1.0`
基于工具: `Hopper Disassembler`
目标架构: `ARM64` (iOS)
最后更新: 2025年9月
截图:
![](https://attach.52pojie.cn/forum/202509/10/222040h777czk3dbyq17jh.jpg)
![](https://attach.52pojie.cn/forum/202509/10/222043n1vq6g3rqur7zkxv.jpg)
文件:
[//pan.huang1111.cn/s/E7VnlTb](https://pan.huang1111.cn/s/E7VnlTb:https)
```
