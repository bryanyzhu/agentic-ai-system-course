# Agentic System Course - Use Agent to Learn Agent

> 中文版 · [English](README.md)

**如果你想一起学习和构建，欢迎加入 [discord channel](https://discord.gg/dWSnHAFdpb)！**

---

这是一门 22 章的骨架（skeleton）课程，讲解如何设计、构建并运维生产级 AI agent —— 它被刻意写成「与你身边的 AI 伙伴一起阅读」的形式。**Agentic system（智能体系统）** 是一类 AI 系统：它能够通过 planning、做决策、使用 tool、根据 feedback 自我调整、拥有 memory 等方式，自主地追求目标 —— 而不只是对单条 prompt 做出回应。与 [Andrej Karpathy 关于 LLM-wiki 的 idea 文档](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) 的思路类似，**这门课给你的是骨架，而你的 agent 会帮你在上面长出肌肉**。

**这门课是**：

- 一副 *骨架（skeleton）* —— 承重的核心主题、模式与决策，并附带各自的 trade-off。
- 为「慢速老化」而写。框架的具体细节腐烂得很快；架构层面的模式不会。
- 一对文件（course + AGENTS.md），既为人类阅读而写，也同样为 AI 消费而设计。

**这门课不是**：

- 一份手把手的 tutorial。这里没有被一步步走完的项目。
- 绑定到某一套技术栈。课程里从不会说「用 LangChain」或「用 Pydantic AI」。该用什么栈，由你的 AI 伙伴根据你的项目来建议。
- 一本参考手册。当你需要某个精确的 API 签名时，去问你的 AI，或者直接读文档。


---

## 如何开始

Clone 这个 repo，用你常用的 IDE 打开来查看课程内容。与此同时，把你的 AI agent（Claude Code / Codex）指向项目根目录，在学习某一章时试试下面这些 prompt：

- *"给我三个现实世界中的例子，说明这件事为什么重要。"*
- *"假设你在面试我，就这个主题出五个由易到难的追问考考我。"*
- *"有什么我本该问、却还没问出来的问题？"*
- *"我刚读完 [某个 pattern X]。我正在构建 [你的项目]。把这个 pattern 翻译成在合适的语言与工具下能跑起来的最小版本，并在你写的时候逐块解释。"*
- *"先把我的项目放一边 —— 让我看看 OpenCode（或 Hermes Agent，或任何一个主流 coding agent）是怎么处理这件事的，以及我们应该从中借鉴什么。"*

你也可以直接把你的 agent 指向 Ch.22 的 design canvas，带着你具体的项目走一遍 —— 这是从「我有一个想法」到「我有一份 spec」最快的路径。

---

## 内置 skills

### `agentic-system-reviewer`

对照课程，review PRD、design doc、implementation plan 或 agent 代码。你可以在 **任何 agentic system** 上运行这个 skill，获得以课程为依据的反馈 —— 你自己的项目、你正在研究的某个开源 agent、还没写任何代码前的 PRD，或是你想要第二意见的同事的 repo。这个 skill 会先校准范围（hobby / team tool / customer-facing），挑出对你这个原型（archetype）最相关的章节，读完它们，然后产出一份「findings 优先」的报告 —— 带 severity、证据、课程引用和具体的修复建议，而不是泛泛的「看起来不错」或「加点 safety」式 review。

在 Claude Code 里，只要描述你想要什么 —— *"对照课程 review 一下这个"*、*"这个 agent 设计好吗？"*、*"它漏掉了哪些章节？"* —— 同时让 agent 指向目标 repo 或文档即可。当你的意图与它的描述匹配时，这个 skill 会自动加载。

**Codex 用户：** 使用 Codex 官方的 `skill-creator` skill 把这个 skill 移植过去。

---

## 课程结构

| 章节 | 主题 |
|---|---|
| **Ch.00** | 如何与你的 AI 伙伴一起使用这门课 |
| **Ch.01–04** | 基础：一次 tool call → the loop → tool 即 contract → prompt 与 cache |
| **Ch.05–08** | Memory 与 state：short-term → long-term → 写入与 curation → persistence |
| **Ch.09–11** | 协调：planning → multi-agent delegation → the harness |
| **Ch.12–14** | 对外接口：human-in-the-loop → connector/MCP → skill/MCP/subagent |
| **Ch.15–17** | 生产级规模：backend → observability → cost、latency、model strategy |
| **Ch.18–19** | 质量与运维：safety / 对抗性输入 → operations 与 forward-deployed |
| **Ch.20–21** | 自主性：proactive agent → self-evolving agent |
| **Ch.22** | Design canvas：设计你自己的 agent |

### 章节目录（点击直达中文译文）

- [Ch.00 · How to use this course](course/zh/00-how-to-use-this-course.md)
- [Ch.01 · One tool call](course/zh/01-function-calling.md)
- [Ch.02 · The agent loop](course/zh/02-the-agent-loop.md)
- [Ch.03 · Tools the agent can trust](course/zh/03-tools-validation.md)
- [Ch.04 · Prompts, context & cache](course/zh/04-prompts-context-cache.md)
- [Ch.05 · Short-term memory](course/zh/05-short-term-memory.md)
- [Ch.06 · Long-term recall](course/zh/06-long-term-recall.md)
- [Ch.07 · Memory writing & curation](course/zh/07-memory-writing.md)
- [Ch.08 · State and persistence](course/zh/08-state-and-persistence.md)
- [Ch.09 · Planning patterns](course/zh/09-planning-patterns.md)
- [Ch.10 · Multi-agent delegation](course/zh/10-multi-agent-delegation.md)
- [Ch.11 · The agent harness](course/zh/11-agent-harness.md)
- [Ch.12 · Human in the loop](course/zh/12-human-in-the-loop.md)
- [Ch.13 · Connectors, MCP, IPC](course/zh/13-connectors-mcp-ipc.md)
- [Ch.14 · Skills, MCP & subagents](course/zh/14-skills-mcp-subagents.md)
- [Ch.15 · Backend infrastructure](course/zh/15-backend-infrastructure.md)
- [Ch.16 · Observability](course/zh/16-observability.md)
- [Ch.17 · Cost, latency & model strategy](course/zh/17-cost-latency-model-strategy.md)
- [Ch.18 · Safety & adversarial inputs](course/zh/18-safety-adversarial-inputs.md)
- [Ch.19 · Ops & forward-deployed](course/zh/19-ops-and-forward-deployed.md)
- [Ch.20 · Proactive agents](course/zh/20-proactive-agents.md)
- [Ch.21 · Self-evolving agents](course/zh/21-self-evolving-agents.md)
- [Ch.22 · Designing your own agent](course/zh/22-designing-your-own-agent.md)

章节顺序经过编排，使得每一章只依赖它之前的内容。如果你已经有一个明确的项目，可以跳读暂时用不上的章节，等用得上时再回来。按项目形态（coding agent、personal assistant、multi-tenant tool、research agent、纯探索）划分的建议学习路径，见 `CLAUDE.md`。

目标不是把这门课「读完」。*目标是把你本来就想做出来的东西 ship 出去，并且理解它的每一行。*

---

## 可选：reference systems

课程偶尔会指向四个开源系统，作为有据可循的示例：

- **OpenCode** —— coding agent（terminal 优先、typed tool、session、compaction）
- **Hermes Agent** —— personal assistant（memory、skill、cron、channel）
- **OpenClaw** —— 自托管的 personal-assistant gateway（channel adapter）
- **Paperclip** —— workflow 控制平面（multi-agent orchestration、持久化的 Postgres state）

你 **不需要** clone 其中任何一个就能从课程中获益，这四个只是用来「对照核验」的。如果你发现自己想要有据可循的答案（*"X 在代码里到底是怎么做 Y 的？"*），你的 AI 伙伴会按需提议 clone 相关的 repo，你也可以提前运行 setup：

```bash
./setup.sh
```

这会把四个 reference repo 全部 clone 到 `references/`。幂等（可安全重跑）。支持用环境变量覆盖，如果你想指向 fork 或固定的 commit。细节见脚本。

如果你想就其他来源提问，尽管把它们放到 `references/` 下，然后聊它就好。

---

## 仓库布局

```
course/         — 22 章（设计上只读 —— 这就是那副骨架）
CLAUDE.md       — 给你 AI 伙伴的行为指南（由 Claude Code 读取）
AGENTS.md       — 与 CLAUDE.md 内容相同，命名是为了 Codex / Cursor / 其他 agent
README.md       — 英文版本文件
README_zh.md    — 本文件（中文）
course/zh/      — 课程章节的中文翻译
setup.sh        — 可选：clone 四个 reference repo
.claude/skills/ — 你 AI 伙伴可调用的内置 skill（reviewer；更多在路上）
docs/           — 你 AI 伙伴按需产出的书面笔记（按需创建）
questions/      — 你 AI 伙伴自动写入的每日 Q&A 日志（按需创建）
workspace/      — 代码、练习与项目构建（按需创建）
references/     — 如果你（或你的 agent）clone 了四个 reference repo，它们在这里
```

`CLAUDE.md` 和 `AGENTS.md` 是重复的 —— 同样的内容用两个文件名，因为不同的 agent 默认会去找不同的文件。两者的组织方式都和课程一样：一份持久、有观点的文件，任何 AI 助手都能一遍读完并转化为有用的行为。

---

## 开始之前的最后一件事

你不需要获得许可才能开始。你不需要在打开你的 agent 之前读完每一章。你不需要知道自己最终会用哪个框架。你构建的第一个东西会小到让这些决定都无关紧要 —— 而一旦它摆在你面前、能跑起来，下一个决定就会变得显而易见。

今天 ship 出最有意思的 agentic system 的人，并不是经验最多的那些人。**而是那些最先和自己的 AI 伙伴进入紧密循环、并且在其中停留最久的人。** 这门课存在的意义，就是当你身处那个循环里时，**你知道该问什么。**


---

## License

见 `LICENSE`。课程内容开放用于教育用途。Reference systems 保留它们各自原本的 license。
