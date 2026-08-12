<p align="center">
  <img src="assets/hero-banner.png" alt="HERO 反过度防御：别让编程 Agent 为一个小功能建一座大堡垒" width="85%">
</p>

# HERO：让 Agent 把力气用在真正的任务上

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=flat)](LICENSE)
· [执行规则](RULES.md)
· [产品设计](PRODUCT.md)
· [案例库](cases/README.md)
· [各工具放置位置](hosts/README.md)

你让 Agent 改进模型、修一个功能，或者分析一段代码。它一开始做得还不错，后来却不断增加冻结文件、哈希、清单、审计、兼容层和版本号。流程越来越完整，真正的结果反而越来越慢。

HERO 处理的就是这种“认真过头，主次颠倒”。它不是让 Agent 少测试、少做安全，而是要求每一项额外工作都能回答三个问题：

1. 当前项目里有什么真实依据？
2. 不做会发生什么具体问题？
3. 结果不同会改变下一步什么决定？

答不清时，这项工作先不要进入关键路径。

## HERO 的四种常见形状

| 类型 | 通俗解释 | 常见表现 |
|---|---|---|
| **H — Hashing** | 没必要也要算哈希 | 生成没人读取的校验和；直接比较就能解决，却先给每条数据算摘要 |
| **E — Edge cases** | 为这里不会出现的情况设防 | 给没有用户和部署的演示项目设计复杂安全体系 |
| **R — Rubrics** | 用流程代替判断 | 已经验证过还反复审计；用评分表制造一个永远无法通过的门 |
| **O — Overbuild** | 为没提出的未来搭架子 | 提前加入迁移框架、兼容层、feature flag 和层层包装 |

长时间实验还有一种更隐蔽的组合：每次小改动都新建版本、冻结候选、计算哈希、生成清单，再对这些治理产物继续写测试。单看每一步都像科研严谨，合在一起却可能让“改进模型”变成“维护实验流程”。探索阶段和正式确认阶段必须分开。

## 推荐用法

这个版本不再只提供一段提醒语，而是分成三层：

| 层 | 作用 | 日常上下文成本 |
|---|---|---|
| [`AGENTS.md`](AGENTS.md) | 始终保留主任务、用户纠偏和阶段边界 | 小且固定 |
| [`keep-task-in-scope` Skill](.agents/skills/keep-task-in-scope/SKILL.md) | 遇到长期优化、冻结审计、兼容迁移或新抽象时做详细判断 | 只在命中时加载 |
| [`cases/`](cases/README.md) | 用正例和反例做离线回归 | 日常不加载 |

对 Codex，最简单的做法是把本仓库 [`AGENTS.md`](AGENTS.md) 中标记范围内的内容复制到目标项目根目录，再复制 Skill：

```bash
mkdir -p .agents/skills
cp -R /path/to/HERO-Anti-OverDefense-CN/.agents/skills/keep-task-in-scope .agents/skills/
```

Skill 不是必装。普通、直接的小改动只用短 `AGENTS.md` 就够；长期研究、反复调参、迁移和加固任务更适合同时使用 Skill。Codex 会在工作前读取项目的 `AGENTS.md`，Skill 则按任务描述显式或隐式触发，具体机制见 OpenAI 官方的 [AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md) 和 [Skills](https://learn.chatgpt.com/docs/build-skills) 文档。

其他 Agent 的放置位置见 [`hosts/README.md`](hosts/README.md)。不要把整个案例库塞进常驻提示词；它会占上下文，也容易让 Agent 把案例误当成机械检查表。

## 什么时候提醒

不按分钟提醒。推荐默认值是：

- 新任务开始时，静默建立一次主交付物和完成条件；
- 用户纠偏、目标变化、会话恢复或上下文压缩后，重新锚定一次；
- 准备新增冻结、哈希、清单、兼容层、宽范围测试或基础设施时，对这个决定检查一次；
- 证据、阶段和用户目标都没变，就不要重复提醒同一件事。

对用户可见的打断只留给真正会改变成本、范围或风险的选择。其余检查应在 Agent 内部完成。

## 当前产品形态

仓库现在是一个能直接使用的 **Repo Kit**：短常驻规则、按需 Skill 和案例回归。下一步更合适的形态不是继续堆提示词，而是一个轻量的 **HERO Guard 插件**：用 Hook 记录范围漂移、在恢复和压缩后重建任务锚点、对确定性的资源限制做拦截，并在任务结束后生成一次回放报告。详细方案见 [`PRODUCT.md`](PRODUCT.md)。

## 使用边界

HERO 反对的是不成比例的方案，不是科研严谨和必要验证。候选进入独立测试前冻结，发布包附带可验证的哈希，或者共享接口改动后跑相关全量回归，都可能完全合理。

判断重点不是“有没有哈希、审计和宽测”，而是它们是否出现在正确阶段，是否服务于一个真实决定。

## 来源与许可

本仓库基于 [wanshuiyin/HERO-Anti-OverDefense](https://github.com/wanshuiyin/HERO-Anti-OverDefense) 独立整理和改写，不是 Fork。保留原项目的概念、案例、版权归属与 MIT 许可证；中文改写和新增执行机制不改变原项目许可。

详见 [`LICENSE`](LICENSE)。
