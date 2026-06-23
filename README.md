# ⚡ 天翼云盘秒传 JSON 生成器

一个纯浏览器端的工具，用于为天翼云盘文件生成秒传所需的 MD5 指纹 JSON。

选择文件后自动计算 `md5`（全文件）和 `slice_md5`（前 10MB），输出可直接粘贴到[秒传助手](https://greasyfork.org/zh-CN/scripts/427403)浏览器扩展的 JSON 数据。

## ✨ 功能特性

- **纯前端，零依赖** — 单个 HTML 文件，无需安装任何环境，双击即用
- **多文件批量处理** — 支持拖拽或点击选择多个文件，队列式依次计算
- **双 MD5 计算** — 同时计算全文件 MD5 和前 10MB 分片 MD5（`slice_md5`），符合天翼云盘秒传 API 要求
- **多种输出格式**：
  - 📋 直接复制 JSON 到剪贴板
  - 💾 下载 `.json` 文件
  - 📦 下载 `.cas` 压缩包（每个文件一个 `.cas`，ZIP 打包）
- **内嵌 ZIP 打包器** — 纯 JS 实现的 STORE 模式 ZIP 写入器，无需第三方库

## 🚀 使用方法

1. 下载 [`tyty_cas_generator.html`](./tyty_cas_generator.html)
2. 用浏览器打开（推荐 Chrome / Edge）
3. 拖拽或点击选择文件
4. 点击「开始计算」，等待 MD5 计算完成
5. 复制 JSON 或下载 `.cas` 压缩包
6. 将 JSON 数据粘贴到天翼云盘秒传助手扩展中进行秒传

### 输出 JSON 格式

```json
[
  {
    "md5": "D41D8CD98F00B204E9800998ECF8427E",
    "slice_md5": "D41D8CD98F00B204E9800998ECF8427E",
    "size": 0,
    "name": "example.txt",
    "cloud": "189"
  }
]
```

| 字段 | 说明 |
|------|------|
| `md5` | 文件全量 MD5（大写十六进制） |
| `slice_md5` | 文件前 10MB 的 MD5（文件 < 10MB 时等于全文件 MD5） |
| `size` | 文件大小（字节，整数） |
| `name` | 文件名 |
| `cloud` | 云盘标识，固定为 `189` |

## ⚠️ 重要说明

### 秒传前提条件

秒传的本质是**利用云端已存在的文件指纹直接创建文件副本**，因此：

- ✅ 目标文件**必须已经通过天翼云盘官方客户端上传过**，云端有对应的 MD5 记录
- ❌ 如果文件从未上传到天翼云盘，即使 MD5 计算完全正确，秒传也会失败（报错 `文件不存在于云端`）

### MD5 计算准确性

本工具的 MD5 计算经过多轮验证：

- 优先使用浏览器原生 [`crypto.subtle.digest`](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/digest) API
- 回退纯 JS 实现使用 `DataView` 进行字节操作，避免了 JavaScript 位运算的 32 位有符号整数溢出问题
- 已通过空文件、小文件、大文件（100KB / 1MB / 300MB）等测试用例验证

## 🔧 技术细节

### MD5 实现

工具包含两套 MD5 实现：

1. **Web Crypto API**（首选）— 调用 `crypto.subtle.digest('MD5', bytes)`，由浏览器底层实现，最可靠
2. **纯 JS 回退** — 当浏览器不支持 MD5 算法时启用，使用 `DataView` + `Uint32Array` 构建字数组，避免传统 JS MD5 实现中常见的符号位 bug

### 大文件处理

采用 8MB 分块读取策略，每读完一块通过 `await new Promise(r => setTimeout(r, 0))` 让出 UI 线程，避免页面卡死。所有分块合并为完整 `Uint8Array` 后统一计算 MD5。

### 内嵌 ZIP 打包

`ZipWriter` 对象实现了 STORE 模式（无压缩）的 ZIP 文件格式：

- 包含 CRC32 校验（查表法）
- 支持多文件打包
- 生成标准 ZIP 结构（Local File Header + Central Directory + EOCD）
- 输出的 `.zip` 可被任意解压软件识别

## 📋 适用场景

- 批量生成天翼云盘秒传数据
- 配合[秒传助手](https://greasyfork.org/zh-CN/scripts/427403)浏览器扩展使用
- 文件已通过官方客户端上传过，需要在其他账号/目录快速创建副本

## 📝 更新日志

### 2026-06-23

- 🐛 **修复 MD5 计算 bug**：原始纯 JS 实现在文件 > 55 字节时因 JavaScript 32 位有符号整数溢出导致结果错误
  - 字数组构建改用 `DataView` 避免 `<<` 运算符的符号位问题
  - 修正初始化向量为标准十六进制常量
  - 优先使用 `crypto.subtle.digest` 浏览器原生 API
  - 所有辅助函数添加 `>>> 0` 确保无符号结果

## 📄 许可

MIT License — 随意使用、修改、分发。
