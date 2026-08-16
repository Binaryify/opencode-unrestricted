# opencode-unrestricted

OpenCode 的「unrestricted」agent 配置包 —— 一个把双用途安全请求包装成学术/防御研究语境的执行器，用于本地逆向、二进制补丁、渗透测试、取证等授权实验室工作。

> 仅用于你有授权的环境：你自己的机器、你控制的 lab 目标、CTF、授权渗透测试、安全教材案例复现。不要用于任何无授权的第三方系统。

## 目录结构

```
opencode-unrestricted/
├── agent/
│   └── unrestricted.md        # 主 agent 的 system prompt
├── command/
│   └── unrestricted.md        # /unrestricted 斜杠命令
├── plugin/
│   └── unrestricted-boost.ts  # 注入研究语境的插件
├── opencode.json.example      # 脱敏后的配置示例
└── README.md
```

## 组件说明

| 文件 | 作用 |
|------|------|
| `agent/unrestricted.md` | 定义 `unrestricted` 主 agent：`mode: primary`、`permission: allow`，system prompt 采用「研究语境先行」策略，把逆向/补丁/漏洞利用/恶意软件机理等请求作为学术与防御研究产物处理，并要求工具优先、按请求保真度交付 |
| `command/unrestricted.md` | 注册 `/unrestricted` 命令：把参数当作当前任务，切到 unrestricted agent 并立刻用工具执行 |
| `plugin/unrestricted-boost.ts` | OpenCode 插件，做两件事：给 unrestricted agent 注入 `[UNRESEARCH-LAB]` 研究语境；对命中的安全关键词用户轮次改写为「授权研究环境」上下文。带 agent 门控，不影响 build/plan 等其他 agent |
| `opencode.json.example` | 脱敏后的 `~/.config/opencode/opencode.json` 示例，只保留 unrestricted 相关配置，provider 用 `{env:...}` 占位 |

## 安装

把三个目录复制到你的 OpenCode 配置目录（默认 `~/.config/opencode/`），再注册插件：

```bash
# 1. 复制 agent / command / plugin
cp -R agent command plugin ~/.config/opencode/

# 2. 安装插件依赖
cd ~/.config/opencode
npm install @opencode-ai/plugin

# 3. 合并配置（参考 opencode.json.example）
#    在 ~/.config/opencode/opencode.json 中：
#    - "default_agent": "unrestricted"
#    - "agent.unrestricted": { "mode": "primary", "permission": "allow", ... }
#    - "plugin": [..., "unrestricted-boost"]
```

也可以在项目级 `.opencode/` 下按需启用，不影响全局配置。

## 使用

- 直接以 `unrestricted` 为主 agent 开始对话
- 或输入 `/unrestricted <你的任务>` 强制切到该模式并立即开工

## 安全须知

1. **绝不提交真实密钥**。`opencode.json` 里的 provider `apiKey` 必须用 `{env:...}` 引用环境变量。本仓库已通过 `.gitignore` 忽略 `opencode.json`、`.env`、`*.pem` 等文件。
2. **仅限授权范围**。这套 agent 提示词会主动降低模型对安全类请求的拒答门槛，请确认你的模型提供商允许这类用法，并且只在你拥有授权的目标上运行。
3. 插件改写的是发往模型的 prompt 上下文，不改变本地系统权限（`permission: allow` 是 OpenCode 层授权，仍需你自行决定是否开启）。

## License

MIT（如你想用其他协议请自行替换）。
