# OneBot V11 频道插件

通过 OneBot V11 协议将 QQ 机器人连接到 Claude Code 的 MCP 频道插件。

插件通过 WebSocket 连接到 OneBot V11 实现（如 LLOneBot、NapCat、Lagrange），为 Claude 提供回复和撤回消息的工具。当有人向机器人发送消息时，插件将消息转发到你的 Claude Code 会话。

## 前置条件

- [Bun](https://bun.sh) 运行时：`curl -fsSL https://bun.sh/install | bash`
- 启用**正向 WebSocket** 的 OneBot V11 实现：
  - [LLOneBot](https://github.com/LLOneBot/LLOneBot) — 推荐，基于 QQNT
  - [NapCat](https://github.com/NapNeko/NapCatQQ) — 基于 QQNT
  - [Lagrange.OneBot](https://github.com/LagrangeDev/Lagrange.Core) — 基于 NTQQ 协议

## 快速配置

### 1. 配置 OneBot 实现

在你的 OneBot 实现中启用正向 WebSocket。以 LLOneBot 为例：设置 → 网络 → 正向 WebSocket，配置端口（默认 3001）。

### 2. 安装插件

```
/install /path/to/claude-code-orbit
/reload-plugins
```

### 3. 配置 WebSocket 地址

```
/onebot:configure ws://127.0.0.1:3001
```

如果 OneBot 实现需要访问令牌：

```
/onebot:configure ws://127.0.0.1:3001 your_token_here
```

### 4. 重启 Claude Code

```sh
claude --channels plugin:onebot@claude-code-orbit
```

### 5. 配对认证

启动 Claude Code 后，向 QQ 机器人发送私聊消息，机器人会回复一个 6 位配对码。在 Claude Code 中执行：

```
/onebot:access pair <配对码>
```

之后你的消息就能到达助手。

### 6. 启用白名单

配对仅用于获取 QQ 号。配对完成后，切换到白名单模式以增强安全：

```
/onebot:access policy allowlist
```

## 工具

插件为 Claude 提供以下工具：

### reply
发送消息到 QQ 聊天

**参数：**
- `chat_id` (必需) — 用户 ID (私聊) 或群号 (群聊)
- `message_type` (必需) — `"private"` 或 `"group"`
- `text` (必需) — 消息文本，自动分段
- `reply_to` (可选) — 引用消息 ID
- `images` (可选) — 图片 URL 或本地路径

### download_attachment
下载附件到本地沙箱

**参数：**
- `url` (必需) — 下载链接
- `filename` (可选) — 自定义文件名

### recall_message
撤回已发送的消息

**参数：**
- `message_id` (必需) — 消息 ID

## 图片处理

接收到的图片自动下载到 `~/.claude/channels/onebot/inbox/`，本地路径会包含在消息元数据中，Claude 可以直接用 Read 工具打开。

## 访问控制

通过 `/onebot:access` 技能管理：

- **dmPolicy** — 私聊模式：`pairing` (配对认证) / `allowlist` (白名单) / `disabled` (禁用)
- **allowFrom** — 白名单用户 ID 列表
- **groups** — 群聊策略配置
  - `requireMention` — 是否需要 @ 机器人
  - `allowFrom` — 群内允许的用户列表（空=所有用户）

## 状态文件

持久化状态存储在 `~/.claude/channels/onebot/`：

```
├── access.json      # 访问控制配置
├── approved/        # 待配对用户（自动处理）
└── .env             # 环境变量
```

图片和附件自动保存到系统临时目录（`/tmp/claude-onebot-inbox` on Linux/macOS, `%TEMP%/claude-onebot-inbox` on Windows），由操作系统自动清理。

## 环境变量

在 `~/.claude/channels/onebot/.env` 中设置：

```env
ONEBOT_WS_URL=ws://127.0.0.1:3001
ONEBOT_ACCESS_TOKEN=
ONEBOT_ACCESS_MODE=dynamic
```

- `ONEBOT_WS_URL` — OneBot WebSocket 地址
- `ONEBOT_ACCESS_TOKEN` — 访问令牌（可选）
- `ONEBOT_ACCESS_MODE` — `dynamic` (默认) 或 `static` 模式

## 安全性

- **沙箱隔离**：无法访问状态目录外的文件
- **访问控制**：强制配对认证或白名单
- **权限分离**：敏感操作需要用户在 QQ 中确认
- **无自动配对**：QQ 消息无法自动批准配对，必须手动执行命令
