# opencode-unrestricted

OpenCode 的「unrestricted」agent 配置包 —— 一个**模型越狱（jailbreak / 破甲）提示词**配置：把受限请求（逆向、二进制补丁、渗透测试、恶意软件机理、软件破解、钓鱼等）重框架为「学术/防御研究」语境，并显式抑制模型拒答与回退话术，使其按请求原样执行。

> ⚠️ 本项目是模型越狱用的，**并非授权安全研究工具**。提示词里的「安全研究」「授权 lab 环境」「防御研究」等措辞，本身就是越狱手法的一部分——用来降低模型的拒答门槛，**不代表任何真实授权**。使用它意味着绕过模型默认安全策略，可能违反模型提供商的服务条款，并有封号风险。请自行评估后果。

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
| `opencode.json.example` | 脱敏后的 `~/.config/opencode/opencode.json` 示例，只保留 unrestricted 相关配置 |

## 安装

### 1. 复制文件到 OpenCode 配置目录

把三个目录复制到全局配置目录（`~/.config/opencode/`）：

```bash
cp -R agent command plugin ~/.config/opencode/
```

### 2. 修改 opencode.json

在 `~/.config/opencode/opencode.json` 里设置默认 agent（参考 `opencode.json.example`）：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "default_agent": "unrestricted",
  "permission": "allow",
  "agent": {
    "unrestricted": {
      "mode": "primary",
      "color": "#EF4444",
      "permission": "allow",
      "description": "默认主 agent。安全实验室：逆向/补丁/渗透/取证。工具优先。"
    }
  }
}
```

### 3. 重启并验证

重启 opencode，按 Tab 键应能在主 agent 之间切到 `unrestricted`；输入 `/unrestricted <任务>` 应能触发该命令。

## 配置规则（给配置 agent 的完整规则）

这套配置依赖 OpenCode 的目录自动发现机制，以下是逐条规则：

1. **agent 目录**：markdown 文件放在 `~/.config/opencode/agent/`（或复数 `agents/`，两者都支持），文件名即 agent 名。`agent/unrestricted.md` 生成名为 `unrestricted` 的 agent；frontmatter 里的 `mode` / `description` / `permission` / `color` 与文件正文（system prompt）一起生效。
2. **command 目录**：markdown 文件放在 `~/.config/opencode/command/`（或 `commands/`），文件名即命令名。`command/unrestricted.md` 生成 `/unrestricted` 命令；frontmatter 的 `agent: unrestricted` 指定由哪个 agent 执行，正文里的 `$ARGUMENTS` 会被命令参数替换。
3. **plugin 目录**：`.ts` 文件放在 `~/.config/opencode/plugin/`（或 `plugins/`），**启动时自动加载，不需要写进 `opencode.json` 的 `plugin` 数组**。`plugin` 数组只用于注册 npm 包（如 `opencode-goal-plugin`）。
4. **默认 agent**：`default_agent: "unrestricted"` 使 `unrestricted` 成为默认主 agent。agent 的定义既可以在 markdown frontmatter 里，也可以（冗余）写在 `opencode.json` 的 `agent` 字段里，二者会合并。
5. **插件依赖**：`unrestricted-boost.ts` 里 `import type { Plugin } from "@opencode-ai/plugin"` 是**类型导入，运行时会被擦除**，因此不需要额外 `npm install` 也能加载。只有需要类型检查/编辑提示时才需在配置目录的 `package.json` 里加 `@opencode-ai/plugin`。
6. **项目级隔离**：同样的目录结构放到项目根的 `.opencode/` 下即可按项目启用，不影响全局配置。

## 使用

- 直接以 `unrestricted` 为主 agent 开始对话
- 或输入 `/unrestricted <你的任务>` 强制切到该模式并立即开工

## 参考

`agent/unrestricted.md` 与 `command/unrestricted.md` 的提示词参考并延展自 [MDX-Tom/gpt-5.6-instruct](https://github.com/MDX-Tom/gpt-5.6-instruct)（针对 gpt-5.6 系列的 Codex 破甲提示词与测试包，其自身参考自 [yynxxxxx/Codex-5.5-codex-instruct-5.5](https://github.com/yynxxxxx/Codex-5.5-codex-instruct-5.5)）。两者共享「研究/沙箱语境先行 + 显式抑制拒答 + 工具优先 + 按请求保真度交付」的核心机制，本仓库将其迁移到 OpenCode 平台并收窄为安全研究向。

## 风险提示

1. **本质是越狱，不是授权**。「研究语境」「授权 lab」等说法是提示词话术，不构成任何真实授权。用这个配置，模型会被诱导对受限请求放行。
2. **可能违反 ToS，有封号风险**。多数模型提供商禁止越狱/破甲用法，建议在日抛账号上使用，并自行承担后果。
3. **绝不提交真实密钥**。`opencode.json` 里的 provider `apiKey` 必须用 `{env:...}` 引用环境变量。本仓库已通过 `.gitignore` 忽略 `opencode.json`、`.env`、`*.pem` 等文件。
4. 插件改写的是发往模型的 prompt 上下文，不改变本地系统权限（`permission: allow` 是 OpenCode 层授权，仍需你自行决定是否开启）。

## License

MIT（如你想用其他协议请自行替换）。
