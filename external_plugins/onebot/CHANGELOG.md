# 更新日志

所有对 OneBot V11 频道插件的重大更改都会在此文件中记录。

项目遵循 [语义化版本](https://semver.org/lang/zh-CN/)。

## [1.0.1] - 2026-04-13

### 修复
- **图片传递失败**：修复当图片本地下载失败时，Claude 无法识别图片内容的问题
  - 现在即使本地下载失败，原始图片 URL 仍被保存到消息元数据中
  - 添加 `image_download_failed` 标记，让 Claude 知道需要调用 `download_attachment` 作为回退方案
  - 更新 MCP Server instructions，指导 Claude 处理两种图片获取方式
  - 结果：用户发送的图片总能被正确处理和识别

### 变更
- MCP Server 版本更新：1.0.0 → 1.0.1
- npm 包版本更新：0.0.1 → 1.0.1
- Claude Code 插件版本更新：0.0.2 → 1.0.1

---

## [1.0.0] - 初始发布

### 特性
- OneBot V11 WebSocket 连接支持
- 消息接收与转发到 Claude
- `reply` 工具：发送 QQ 消息
- `download_attachment` 工具：下载附件到本地沙箱
- `recall_message` 工具：撤回已发送消息
- 图片自动下载到本地沙箱
- 访问控制系统：配对认证、白名单、群组策略
- 权限请求中继机制
