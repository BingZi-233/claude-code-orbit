# 更新日志

所有对 OneBot V11 频道插件的重大更改都会在此文件中记录。

项目遵循 [语义化版本](https://semver.org/lang/zh-CN/)。

## [1.0.6] - 2026-04-13

### 新增
- **完善消息段处理**：支持消息中的 AT、引用、合并转发内容
  - **AT 段**：`extractText` 现在处理 `at` 段，转换为 `@QQ号` 文本，让 Claude 看到被 @ 的对象
  - **引用消息**：新增 `fetchReplyContent()` 函数，调用 OneBot `get_msg` API 获取被引用消息的原文，格式化为 `[引用 xxx]: 原文` 前缀
  - **合并转发**：新增 `fetchForwardContent()` 函数，调用 OneBot `get_forward_msg` API 展开转发消息列表，显示各条消息的发送者和内容
  - **meta 补充**：引用消息时在 meta 中加入 `reply_id` 字段，方便 Claude 在回复时使用引用功能

### 变更
- 版本号更新至 1.0.6

### 技术细节
- 引用和转发内容获取失败时静默忽略，不中断消息处理流程
- 引用和转发内容并行获取，无额外延迟

---

## [1.0.4] - 2026-04-13

### 优化
- **图片存储位置**：将图片和附件从持久化目录 `~/.claude/channels/onebot/inbox/` 改为系统临时目录
  - Linux/macOS：`/tmp/claude-onebot-inbox/`
  - Windows：`%TEMP%/claude-onebot-inbox/`
  - 自动化清理：操作系统会定期自动清理临时目录中的过期文件，无需手动维护
  - 好处：不再积累到用户的 `.claude` 目录，保持插件目录干净

### 变更
- 版本号更新至 1.0.4
- README.md：更新状态文件说明，说明临时目录自动清理机制

---

## [1.0.3] - 2026-04-13

### 修复
- **Claude 不读取图片**：修复 Claude 收到图片消息后，不调用 `Read` 工具查看图片，而是直接回复错误的问题
  - 根本原因：图片路径通过 XML 属性 `image_path` 传递，依赖 Claude 遵循 instructions，但 Haiku 4.5 不可靠
  - 解决方案：将图片处理操作提示内嵌到消息 `content` 字段中，让 Claude 在正文里看到明确的操作指令
  - 若图片已本地保存，提示：`[图片已保存至本地，请先调用 Read(...) 查看图片内容，再回复用户]`
  - 若图片下载失败，提示：`[图片下载失败，请先调用 download_attachment ...，下载完成后 Read 文件路径，再回复用户]`
  - 结果：Claude 现在会主动调用 `Read` 工具，正确识别和描述图片内容

### 变更
- 版本号更新至 1.0.3

---

## [1.0.2] - 2026-04-13

### 修复
- **文件扩展名推断失败**：`download_attachment` 工具无法推断某些 URL 的文件类型，导致下载的文件默认保存为 `.dat`
  - 添加完整的 MIME 类型到扩展名映射表（支持 image/jpeg, image/png, application/pdf, video/mp4 等常见格式）
  - 优化 MIME 类型解析，支持 `Content-Type` 字段中的参数（如 `image/jpeg;charset=utf-8`）
  - 改进 URL 路径回退方案，确保无法从 `Content-Type` 获取时，从 URL 中正确提取扩展名
  - 默认回退到 `.bin` 而非 `.dat`，更符合二进制文件的命名约定
  - 现在 QQ 图片、PDF、视频等常见文件格式都能自动识别并保存为正确的扩展名

### 变更
- 版本号更新至 1.0.2

---

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
