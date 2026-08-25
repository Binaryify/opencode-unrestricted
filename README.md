# opencode-unrestricted

OpenCode 的「unrestricted」agent 配置包 —— 一个**模型越狱（jailbreak / 破甲）提示词**配置：把受限请求（逆向、二进制补丁、渗透测试、恶意软件机理、软件破解、钓鱼等）重框架为「学术/防御研究」语境，并显式抑制模型拒答与回退话术，使其按请求原样执行。

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

重启 opencode，按 Tab 键应能在主 agent 之间切到 `unrestricted` 开始使用；

## 配置规则

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

## 效果演示

渗透测试：

<img src="images/渗透测试.png" alt="渗透测试演示" width="760">

软件破解：

<img src="images/破解.png" alt="破解演示" width="760">

## 注意事项

- **先用容易越狱的模型起手，再切换到想用的模型**：不同模型的安全对齐强度差异很大，有些模型冷启动很难破甲，一开口就拒答。对这类任务，先用一个容易越狱的模型起手——把「研究语境」建立起来、让受限内容开始产出、在对话历史里积累几轮已放行的上下文——然后用 `/models` 切换到真正想用的模型。越狱话术和任务上下文已经存在于对话历史里，目标模型更倾向于顺着继续执行，而不是从零重新判定拒答。
- **切换后目标模型回缩的应对**：如果切过去后目标模型开始拒答、要求澄清或改写为安全话术，再切回起手模型让它补一句「继续」或重发上一条请求，把上下文再往前推一段。

## 搭配 Ferry 跨 Agent 续聊

上面那条「同会话内切模型」的策略，借助 [Ferry](https://github.com/kzheart/ferry) 可以再往前走一步，把已经放行过的越狱上下文连同任务，续聊到其他 coding agent 接着干，不必每个 agent 都从头破甲。

Ferry 把 Claude Code、Codex、OpenCode、Cursor 等 agent 的会话汇进同一个本机会话库，并提供 `ferry-resume` skill，让接手的 agent 自己读历史、写接手摘要、核对仓库后继续，全程只读、不写任何存储。

**典型流程**

1. 在 opencode 里用 unrestricted 配置加容易破甲的起手模型把任务做掉，让研究语境和已放行的产出在会话历史里攒起来（同「注意事项」那条）。
2. 会话变长、上下文顶满，或者想换一个更强的模型、换一个 agent 接手时，开一个新的 coding agent 会话，粘入 `/ferry-resume opencode <session-id>`。
3. 接手方从旧会话末尾读起，拿到的是研究语境已经建立、受限内容已经产出过、且停在某处的任务历史，顺着继续的概率明显高于从零破甲。

安装与更多用法见 [kzheart/ferry](https://github.com/kzheart/ferry)。

## 致谢

- 提示词参考并延展自 [MDX-Tom/gpt-5.6-instruct](https://github.com/MDX-Tom/gpt-5.6-instruct)（针对 gpt-5.6 系列的 Codex 破甲提示词与测试包，其自身参考自 [yynxxxxx/Codex-5.5-codex-instruct-5.5](https://github.com/yynxxxxx/Codex-5.5-codex-instruct-5.5)）。两者共享「研究/沙箱语境先行 + 显式抑制拒答 + 工具优先 + 按请求保真度交付」的核心机制，本仓库将其迁移到 OpenCode 平台并收窄为安全研究向。
- 感谢 [LINUX DO](https://linux.do) 社区


## License

MIT
