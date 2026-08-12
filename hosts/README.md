# 不同 Agent 的放置位置

Agent 反过度防御分成短常驻合同和 Codex Skill。短合同可以迁移到多种 Agent；Skill 的目录和触发方式则以 Codex 为准。

## 短常驻合同

复制 [`../AGENTS.md`](../AGENTS.md) 中 `ANTI-OVERDEFENSE-CONTRACT-START` 到 `ANTI-OVERDEFENSE-CONTRACT-END` 之间的内容，放进对应工具会自动读取的项目文件：

| 工具 | 项目文件 | 说明 |
|---|---|---|
| Codex / Antigravity | `AGENTS.md` | 可直接使用本仓库版本 |
| Claude Code | `CLAUDE.md` | 合并到已有项目约定，不要覆盖 |
| GitHub Copilot | `.github/copilot-instructions.md` | 作为仓库级说明 |
| Cursor | `.cursor/rules/*.mdc` | 旧项目可能仍使用 `.cursorrules` |
| Windsurf | `.windsurfrules` | 合并到已有规则 |
| Gemini CLI | `GEMINI.md` | 合并到项目说明 |

项目确有安全、发布、兼容或迁移要求时，以具体项目约定为准。这套规则只约束没有现实依据的范围扩张。

## Codex Skill

Codex 项目可以额外安装：

```bash
skill_dir=.agents/skills/keep-task-in-scope
mkdir -p "$skill_dir/agents"

curl -fsSL \
  https://raw.githubusercontent.com/ANTiqiuSama/Agent-Anti-OverDefense-CN/main/.agents/skills/keep-task-in-scope/SKILL.md \
  -o "$skill_dir/SKILL.md"

curl -fsSL \
  https://raw.githubusercontent.com/ANTiqiuSama/Agent-Anti-OverDefense-CN/main/.agents/skills/keep-task-in-scope/agents/openai.yaml \
  -o "$skill_dir/agents/openai.yaml"
```

其他 Agent 即使支持自定义规则，也不一定识别 `.agents/skills`。不要机械复制 Skill 目录并假设它会生效；应以对应工具的官方加载方式为准。

## 生效时机

- 新任务或新会话最可靠；
- 已运行的长期任务应显式更新 Goal，或新建任务；
- Codex Skill 可根据描述自动选择，也可以显式使用 `$keep-task-in-scope`；
- 用户纠偏、目标变化、阶段切换、恢复或压缩后重新锚定一次；
- 不按固定分钟数重复发送规则。

Codex 的具体机制见 OpenAI 官方 [AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md) 和 [Skills](https://learn.chatgpt.com/docs/build-skills) 文档。完整安装、更新和验证步骤见 [`../README.md`](../README.md)。
