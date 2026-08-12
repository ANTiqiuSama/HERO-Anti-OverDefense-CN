# Agent 反过度防御

> 解决无效哈希、虚构边界、机械评审和超前搭建，让 Agent 先完成真正的任务。

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=flat)](LICENSE)
· [执行规则](RULES.md)
· [产品设计](PRODUCT.md)
· [案例库](cases/README.md)
· [其他 Agent 的放置位置](hosts/README.md)

Agent 反过度防御是一套给编程和研究型 Agent 使用的任务范围约束。

它解决的不是“Agent 不够努力”，而是另一种常见问题：Agent 很努力，却把时间花在了次要工作上。你让它修一个功能或继续改进模型，它后来开始不断增加哈希、冻结文件、审计、清单、兼容层、宽范围测试和版本号。流程越来越完整，真正的交付越来越慢。

它要求 Agent 始终回答三件事：

1. 当前真正要交付什么？
2. 新增工作有什么现实依据，不做会发生什么具体问题？
3. 这项检查或结构会改变哪个后续决定？

答不清时，这项工作可以记录为可选项，但不应抢占当前关键路径。

## 解决的四类问题

| 分类 | 问题 | 常见表现 |
|---|---|---|
| **H — Hashing** | 没必要也要算哈希 | 生成没人读取的校验和；直接比较就能解决，却先给每条数据算摘要 |
| **E — Edge cases** | 为这里不会出现的情况设防 | 给没有用户和部署的演示项目设计复杂安全体系 |
| **R — Rubrics** | 用流程代替判断 | 已经验证过还反复审计；用评分表制造一个永远无法通过的门 |
| **O — Overbuild** | 为没提出的未来搭架子 | 提前加入迁移框架、兼容层、feature flag 和层层包装 |

H / E / R / O 只是案例分类，不再作为项目名称。四类问题的共同点是：额外机制缺少现实依据，却开始占用主任务的时间、代码和上下文。

## 它不是什么

这套规则不是“少测试”“别做安全”或“永远用最简单方案”。

下面这些仍然应该做：

- 项目真实可达的边界情况；
- 用户、项目约定或更高优先级规则明确要求的安全、迁移、审阅和验证；
- 能推翻当前实现的冒烟测试；
- 共享接口改变后的相关回归；
- 候选进入独立测试前的冻结和防泄漏措施；
- 真实发布包需要的哈希、清单和复现说明。

它反对的是阶段错位和比例失衡：探索还在快速试错，却为每个小版本建立一整套发布级治理；代码没有真实消费者，却提前建设迁移和兼容体系；刚验证过且未改动的内容，又被反复审计。

## 仓库里有什么

| 内容 | 作用 | 是否自动进入上下文 |
|---|---|---|
| [`AGENTS.md`](AGENTS.md) | 短常驻合同：主交付物、证据门、用户纠偏和阶段边界 | Codex 新任务启动时读取 |
| [`keep-task-in-scope` Skill](.agents/skills/keep-task-in-scope/SKILL.md) | 长期优化、冻结审计、迁移、兼容、加固和新抽象的详细判断流程 | 任务命中或显式调用时加载 |
| [`RULES.md`](RULES.md) | 为什么这样设计、什么时候触发、多久触发一次 | 不应整篇放入常驻配置 |
| [`cases/`](cases/README.md) | 正例、反例和组合案例，用于离线回归 | 不自动加载 |
| [`PRODUCT.md`](PRODUCT.md) | Goal Compiler、运行时 Guard 和回放器的后续产品设计 | 不自动加载 |

推荐配置是：**短 `AGENTS.md` 常驻 + Skill 按需加载 + 案例离线评估**。

## 选择安装方式

### 方式一：只安装短规则

适合普通开发任务，或者你只想先验证基本效果。

如果目标项目还没有 `AGENTS.md`，在目标项目根目录执行：

```bash
curl -fsSL \
  https://raw.githubusercontent.com/ANTiqiuSama/Agent-Anti-OverDefense-CN/main/AGENTS.md \
  -o AGENTS.md
```

如果目标项目已经有 `AGENTS.md`，不要覆盖。先下载到临时文件：

```bash
curl -fsSL \
  https://raw.githubusercontent.com/ANTiqiuSama/Agent-Anti-OverDefense-CN/main/AGENTS.md \
  -o /tmp/anti-overdefense-AGENTS.md
```

然后把下面两个标记之间的内容合并到现有 `AGENTS.md`：

```text
ANTI-OVERDEFENSE-CONTRACT-START
ANTI-OVERDEFENSE-CONTRACT-END
```

保留项目已有的构建、测试、安全和发布规则。Agent 反过度防御负责约束范围，不替代项目自身约定。

### 方式二：安装短规则和 Skill（推荐）

长期研究、持续调参、复杂重构、迁移、安全加固或多阶段任务，建议再安装 Skill。

在目标项目根目录执行：

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

安装后的结构应该是：

```text
目标项目/
├── AGENTS.md
└── .agents/
    └── skills/
        └── keep-task-in-scope/
            ├── SKILL.md
            └── agents/
                └── openai.yaml
```

Skill 支持两种触发方式：

- **自动触发**：任务涉及长期优化、SOTA、顶刊目标、冻结、哈希、反复审计、迁移、兼容层、宽范围测试、新依赖或新基础设施时，Codex 可根据 Skill 描述选择它；
- **显式触发**：在提示词中写 `$keep-task-in-scope`。

例如：

```text
使用 $keep-task-in-scope 继续改进当前模型。先区分探索和确认阶段，
本轮只验证一个明确假设，不为每个尝试创建新的冻结包。
```

Skill 采用渐进加载：启动时只提供名称和描述，真正命中后才读取完整 `SKILL.md`，因此比把全部规则和案例长期塞进 `AGENTS.md` 更省上下文。具体机制见 OpenAI 官方 [Skills 文档](https://learn.chatgpt.com/docs/build-skills)。

### 方式三：只在当前任务中临时使用

如果你不能修改仓库文件，可以把 [`AGENTS.md`](AGENTS.md) 的规则块直接放进当前提示词。它只对当前任务生效，适合临时验证，不适合无人值守的长期任务。

## 什么时候开始生效

Codex 会在开始工作时读取 `AGENTS.md` 并构建指令链。修改文件后，**新任务或新会话最可靠**；已经运行很久的旧任务不会因为 Git 文件改变而自动重写自己的 Goal。官方加载规则见 [AGENTS.md 文档](https://learn.chatgpt.com/docs/agent-configuration/agents-md)。

对于已经运行中的长期任务，有两种做法：

1. 新建任务，让新的 `AGENTS.md` 和 Skill 从开始就参与；
2. 继续旧任务，但显式更新 Goal，写清当前阶段、硬约束和停止条件。

长期任务可以使用下面这个短模板。它不是每个任务都必须填写的表格，只在目标本身没有自然终点时使用：

```text
主交付物：这一阶段最终要得到什么。
当前阶段：探索 / 确认 / 发布。
本轮假设：这次改动准备验证什么。
比较依据：哪些指标或现象会支持或否定它。
资源边界：本轮可用的数据、调用、时间或尝试范围。
停止条件：何时保留、回滚或停止这条路线。
硬约束：例如只用已有数据、不联网、不新增依赖。
```

“持续改进”“追求 SOTA”“达到顶刊标准”可以作为方向，但不能单独充当停止条件。

## 多久提醒一次

不要按 5 分钟、10 轮对话或固定 token 数提醒。默认策略是事件触发：

| 事件 | 动作 | 是否打断用户 |
|---|---|---|
| 新任务开始 | 静默建立一次主交付物和完成条件 | 通常不打断 |
| 用户改变目标或明确纠偏 | 更新当前阶段约束，并重新锚定一次 | 只说明变化后的约束 |
| 会话恢复或上下文压缩 | 恢复一次任务锚点 | 通常不复述全部规则 |
| 准备新增冻结、哈希、清单、兼容层、宽测或基础设施 | 对这个范围决定运行一次证据门 | 只有会改变成本或方案时说明 |
| 探索进入确认，或确认进入发布 | 更新阶段，重新判断治理产物是否必要 | 必要时说明 |
| 证据、阶段和目标都没变 | 不重复提醒 | 不打断 |

可以把提醒去重理解为：

```text
当前任务 + 当前阶段 + 决策类型 + 当前证据
```

这个组合没变，同一件事只判断一次。

## Agent 实际会怎么做

### 普通功能开发

用户要求增加一个接口。Agent 先实现最小完整功能，再跑能覆盖改动路径的测试。没有真实消费者时，不提前增加多版本迁移框架。

### 长期模型实验

探索阶段复用同一实验账本，每轮对应一个假设和停止条件。选出候选后，才进入确认阶段冻结配置；确定要对外发布时，再生成完整清单、哈希和可复现包。

### 真实安全或兼容要求

项目处理不可信输入、维护公开接口或明确要求审计链时，这些工作本身就是主任务。这套规则不应拿来压掉它们，只要求写清适用对象、开始条件和结束条件。

### 用户中途纠偏

用户说“减少无效封装，只使用已有数据，不要再下载”，Agent 不应只执行“不要下载”。它需要把“减少封装”和“只使用已有资源”一起锁定为当前阶段约束，后续不再用其他名称继续同类流程膨胀。

## 如何确认安装成功

先检查文件：

```bash
grep -n 'ANTI-OVERDEFENSE-CONTRACT' AGENTS.md
test -s .agents/skills/keep-task-in-scope/SKILL.md && echo 'Skill installed'
```

然后在一个新任务中做行为验证。不要问 Agent “你知不知道反过度防御规则”，那只能证明它会复述。应该给它一个容易过度建设的小任务，观察：

- 主功能是否先完成；
- 真实可达的缺陷是否仍被发现；
- 没有依据的哈希、兼容层和宽测是否减少；
- 验证是否覆盖仍然活着的疑点；
- 没问题时是否能明确说没问题。

需要固定回归案例时，从 [`cases/`](cases/README.md) 同时选择“应该克制”和“应该扩大验证”的案例。只测少做，不测误伤，会把偷工减料误判成效果改善。

## 如何更新

更新前先查看差异，不要直接覆盖已有 `AGENTS.md`：

```bash
curl -fsSL \
  https://raw.githubusercontent.com/ANTiqiuSama/Agent-Anti-OverDefense-CN/main/AGENTS.md \
  -o /tmp/anti-overdefense-AGENTS.new.md

diff -u AGENTS.md /tmp/anti-overdefense-AGENTS.new.md
```

如果你的 `AGENTS.md` 还包含项目规则，只更新两个 `ANTI-OVERDEFENSE-CONTRACT` 标记之间的内容。

Skill 可以重新执行安装命令更新；它的目录只保存本项目自带文件时，覆盖这两个文件即可。

## 如何移除

1. 从 `AGENTS.md` 删除两个 `ANTI-OVERDEFENSE-CONTRACT` 标记及其中内容；
2. 删除 `.agents/skills/keep-task-in-scope/`；
3. 新建任务，确认旧任务规则没有被继续复用。

移除前如果你在同一目录加入了自己的 Skill 资源，先检查再删除，不要直接清空整个 `.agents/skills`。

## 其他 Agent

短常驻合同也可以放进其他工具会自动读取的项目文件：

| 工具 | 建议文件 |
|---|---|
| Claude Code | `CLAUDE.md` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Cursor | `.cursor/rules/*.mdc`，旧项目可能使用 `.cursorrules` |
| Windsurf | `.windsurfrules` |
| Gemini CLI | `GEMINI.md` |

这些工具的 Skill 和加载机制并不完全相同。对不支持同样 Skill 目录的工具，只复制短合同即可，不要假设 Codex 的 `.agents/skills` 会被自动识别。详细放置说明见 [`hosts/README.md`](hosts/README.md)。

## 产品边界

当前仓库提供的是自然语言配置和离线案例，不是强制执行器。更高优先级规则会覆盖它，模型也可能误判一次冻结或测试到底是否必要。

后续产品方向是把 Goal 编译、事件去重、确定性 Hook 和任务回放做成轻量插件，但要先证明当前 Repo Kit 能减少范围漂移，同时不会压掉真实安全、兼容和验证要求。完整设计见 [`PRODUCT.md`](PRODUCT.md)。

## 来源与许可

Agent Anti-OverDefense 基于 [wanshuiyin/HERO-Anti-OverDefense](https://github.com/wanshuiyin/HERO-Anti-OverDefense) 的公开概念与案例独立整理和扩展，不是 Fork。仓库保留原项目版权归属和 MIT 许可证；新的中文说明、任务阶段机制和 Skill 不改变原项目许可。

详见 [`LICENSE`](LICENSE)。
