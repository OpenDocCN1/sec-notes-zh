
# [Android 原创] [原创]AI时代免费工具：用通义灵码和MCP工具构建逆向分析工作流

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=2078564
- 抓取时间：2026-11-01T13:51:15.443Z

## AI时代免费工具：用通义灵码和IDA Pro MCP，JEB MCP构建免费的AI逆向分析工作流

### 前言

大模型对我而言，最重要的是学习效率的提升，因此免费的大模型资源对学习者来说很有性价比；

本文采用：VSCode + 通义灵码。

如果工作需要的话还是建议：Cursor + ChatGPT 5.1/Gemini 3 Pro。

### 目录

- 使用的工具介绍
  - `idapromcp333` 简介
  - JEB MCP 简介
- APK 样本介绍
  - Small Detector - 签名校验强度自测工具
    - 1. `RootCheck` 检测分析
      - 1.1 分析报告
      - 1.2 Frida Hook 脚本
    - 2. `HookCheck` 检测分析
      - 2.1 分析报告
      - 2.2 Frida Hook 脚本
    - 3. `SignatureCheck` 检测分析
      - 3.1 分析报告
      - 3.2 Frida Hook 脚本

## 使用的工具介绍

### `idapromcp333` 简介

`idapromcp333` 是 IDA Pro MCP 智能逆向分析平台的增强版，基于 `mrexodia/ida-pro-mcp` 二次开发。

该工具通过 MCP（Model Context Protocol）协议将 IDA Pro 与各类 AI 助手（如通义灵码、Cursor、Cline 等）无缝集成，实现全自动智能化的二进制逆向分析。

#### 核心功能

- 函数/变量批量重命名与类型修复
- 智能注释生成
- 结构体自动声明
- 代码混淆检测
- 反调试逻辑识别
- 基础算法识别（加密、哈希等）
- 爆破脚本自动生成与执行
- 主流程图自动生成
- 结构化分析报告导出（Markdown 格式）

#### 源码仓库

- 增强版仓库：https://github.com/namename333/idapromcp_333
- 原版仓库：https://github.com/mrexodia/ida-pro-mcp

### JEB MCP 简介

JEB MCP 是为 JEB Pro 反编译器开发的 MCP 服务器插件，专门用于 APK 逆向工程。

它允许 AI 助手通过自然语言指令调用 JEB 的强大反编译功能，实现 Android 应用的智能化分析。

#### 核心功能

- APK 基本信息提取（包名、版本、主 `Activity` 等）
- `AndroidManifest.xml` 解析
- 权限、组件（`Activity`/`Service`/`Receiver`/`Provider`）分析
- 反编译代码获取（Java/Kotlin）
- 方法调用关系追踪
- 方法重写链分析
- `Intent-filter` 提取
- 导出组件检测

#### 源码仓库

- 主要仓库：https://github.com/flankerhqd/jebmcp
- 参考项目：https://github.com/dawnslab/jebmcp

## VSCode 连接配置
