# 放到哪里才会生效

HERO v2 分成短常驻规则和按需 Skill。不要把整个 `RULES.md`、`PRODUCT.md` 或案例库塞进 Agent 的常驻配置。

## 常驻规则

复制 [`../AGENTS.md`](../AGENTS.md) 中 `HERO-CONTRACT-START` 到 `HERO-CONTRACT-END` 的内容，放进对应工具会自动读取的项目文件：

| 工具 | 项目文件 |
|---|---|
| Codex / Antigravity | `AGENTS.md` |
| Claude Code | `CLAUDE.md` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Cursor | `.cursor/rules/*.mdc`，旧项目可能使用 `.cursorrules` |
| Windsurf | `.windsurfrules` |
| Gemini CLI | `GEMINI.md` |

如果文件已经存在，把这段放在项目约定附近即可，不要覆盖原内容。项目确有发布、安全、兼容或迁移要求时，以更具体的项目规则为准。

## Codex Skill

复杂任务再复制 Skill：

```bash
mkdir -p .agents/skills
cp -R /path/to/HERO-Anti-OverDefense-CN/.agents/skills/keep-task-in-scope .agents/skills/
```

Codex 开始工作前会读取 `AGENTS.md`。Skill 只先暴露名称和描述，任务命中后才加载完整内容，因此适合承载长期实验、冻结审计和迁移判断，而不用让所有简单任务支付同样的上下文成本。

## 生效时机

- 修改项目 `AGENTS.md` 后，对新启动的任务最可靠；
- 已经运行很久的任务，应在目标改变、恢复或上下文压缩后重新锚定；
- Skill 可由 Agent 根据描述自动选择，也可以显式使用 `$keep-task-in-scope`；
- 不要按固定分钟数重复发送 HERO 提醒。

关于 Codex 的实际加载规则，以 OpenAI 官方 [AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md) 和 [Skills](https://learn.chatgpt.com/docs/build-skills) 文档为准。
