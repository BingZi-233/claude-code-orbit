# Claude Code 轨道项目

Claude Code 外部插件和工具的集合。

## 项目结构

```
├── external_plugins/          # 外部 MCP 频道插件
│   └── onebot/               # OneBot V11 频道插件
│       ├── server.ts         # 主程序
│       └── package.json       # 依赖配置
├── .claude-plugin/           # Claude Code 插件配置
└── README.md                 # 本文件
```

## 插件

### OneBot (external_plugins/onebot)

通过 QQ (OneBot V11) 与 Claude 进行交互的 MCP 频道插件。

**功能：**
- QQ 私聊和群聊消息接收
- 消息回复、撤回
- 图片接收和发送
- 访问控制和权限管理
- 群聊提及检测

**快速开始：**

```bash
cd external_plugins/onebot
bun install --no-summary && bun server.ts
```

详见 `external_plugins/onebot` 目录的说明文档。

## 许可证

Apache 2.0
