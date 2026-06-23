# ⚡ 天翼云盘秒传 JSON 生成器

纯浏览器端工具，为天翼云盘文件生成秒传所需的 MD5 指纹 JSON。配合[秒传助手](https://greasyfork.org/zh-CN/scripts/427403)浏览器扩展使用。

## ✨ 特性

- 纯前端，单个 HTML 文件双击即用（依赖 JSZip CDN）
- 多文件批量处理，拖拽选择
- 同时计算全文件 MD5 + 前 10MB 分片 MD5
- 单文件直接下载 `.cas`，多文件 ZIP 打包（JSZip 生成，中文文件名不乱码）
- ZIP 文件名带生成时间戳，格式 `cas_files_YYYYMMDD_HHMMSS.zip`

## 🚀 用法

1. 下载 [`ty_cas_generator.html`](./ty_cas_generator.html)
2. 浏览器打开（推荐 Chrome / Edge）
3. 拖拽或选择文件 → 点击「开始计算」
4. 复制 JSON 或下载 `.cas` 文件

## 输出格式

```json
{
  "md5": "D41D8CD98F00B204E9800998ECF8427E",
  "slice_md5": "D41D8CD98F00B204E9800998ECF8427E",
  "size": 0,
  "name": "example.txt",
  "cloud": "189"
}
```

## ⚠️ 说明

- 秒传前提：目标文件**必须已通过天翼云盘官方客户端上传过**，云端有 MD5 记录
- MD5 使用纯 JS 实现（DataView 避免符号位溢出）

## 📝 更新日志

### 2026-06-23

- 修复 MD5 计算 bug（> 55 字节时 JS 位运算符号位溢出导致结果错误）
- 单文件直接下载 `.cas`，不再强制打包 ZIP
- 修复 ZIP 中文文件名乱码（flags 设置 bit 11 标识 UTF-8）
- 替换内嵌 ZipWriter 为 JSZip 库，彻底解决中文文件名乱码问题
- ZIP 文件名添加生成时间戳（精确到秒）

## 📄 许可

MIT License
