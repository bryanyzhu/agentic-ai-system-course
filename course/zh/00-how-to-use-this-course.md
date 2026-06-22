# Chapter 00 — How to use this course

## TL;DR

你即将学习的是 production-grade（生产级）agentic 系统是如何设计出来的——不是靠通读别人成千上万行代码，而是沿着一张模式地图走一遍，再与你自己的 AI agent 结对，把这些模式应用到你真正想做的任何项目上。本章解释这门课为什么要这样写、如何在 agent 的陪伴下阅读它，以及如何把每一章个性化成你能真正交付的东西。一切都围绕着你和你的项目展开。

---

## Why this course exists

在 2026 年初，我一遍又一遍地看到同一件事发生。一个从没写过一行代码的人打开 Claude 或 Codex，描述一个问题，四十分钟后就盯着一个自己逐行都看得懂的可用原型。一位资深工程师过去要花一周时间去啃一个新框架，现在只是粗读文档、向自己的 agent 抛出五个犀利的问题，当天下午就能交付同样的东西。起点各不相同，但 loop 是一样的：一个好奇的人、一个真诚的问题、一个会在对话里当场解释、搭建脚手架并运行代码的 AI 伙伴。

这种学习方式大约比看视频或跟着分步教程走**快十倍**。视频是被动的，而且被冻结在某个时间点上。教程过度贴合某一种技术栈，新模型一发布就开始腐烂。AI agent 恰恰相反——它是交互式的、与时俱进的、为你量身定制的。瓶颈不再是信息。瓶颈在于知道该问什么。

这正是这门课存在的意义。它是 agent 自己交不到你手上的那张地图。厂商也不会把这张地图给你，因为每家厂商都想让你以为他们的框架**就是**那张地图。读完业界讨论最多的几个 production agent 的源码——OpenCode、Hermes Agent、OpenClaw、Paperclip 等等——有一件事变得显而易见：它们共享着同样的十几个核心思想。名字不同、文件结构不同，底层却是同样的模式。这门课就是把这些模式一次性写下来，用平实的语言，连同那些真正要紧的 trade-off。

## What this course is, and is not

这门课是一根**脊柱（spine）**——是每一个严肃的 agentic 系统里都会出现的、承重的主题、模式和决策。它包含图示、术语、失败模式、设计取舍，以及来自真实 production 系统的笔记。它对**该思考什么**有鲜明的立场，对**该 import 哪个库**几乎只字不提。

这门课**不是**一份分步教程。这里没有"先装这个，再粘那个"。没有一个我们从头到尾一起走完的项目。没有测验，没有要克隆的结业仓库，没有只能在某一种技术栈上跑起来的 notebook。

这是有意为之。一门课里老化最快的细节——SDK 版本、模型名称、框架 API、价格——恰恰是你的 agent 已经掌握了最新答案的那些细节。而老化最慢的细节——tool registry 到底是什么、planning 时该用 checklist 还是 graph、memory poisoning 是怎么发生的、cache 为什么会失效、什么时候该把 human 放进 loop 里——才是你能在这里找到的东西。

可以这样理解：我给你骨架，你的 agent 会帮你把肌肉填上去。

## How to actually use this course

无论你有十年工程经验，还是从没碰过 terminal，这门课的用法都是一样的。区别只在于你给 agent 什么样的 prompt。挑那些符合你当下处境的来用——而且别客气，无论你的背景如何，每一章都可以用最简单的那几个。*"把它讲给我听，就当我 13 岁"* 这一招永远不会过时。

**要理解一章**（适用于任何读者，无论是否技术背景）：

- *"把这一章讲给我听，假设我完全不懂这些行话。每个术语第一次出现时都给我下个定义。"*
- *"给我三个现实世界里这件事很重要的例子。"*
- *"这一章里我唯一该记住的事情是什么？"*
- *"有什么我本该问、但还没问出口的问题？"*

**要把它应用到你的项目上**（当你心里已经有了点想法）：

- *"我刚读到了 [模式 X]。我正在做 [你的项目]。把这个模式翻译成在合适的语言和工具下真正能跑起来的最小版本，并在你写的时候逐块解释。"*
- *"这里是 [Ch.NN] 警告过的那些失败模式。看看我们早先做的东西，告诉我其中哪些已经构成风险。"*
- *"给我两种做法，每种用三个要点列出 trade-off——别写成大段大段的。"*

**要与真实系统对照**（当你想给一个想法找个落脚点）：

- *"先把我的项目放一边——给我看看 OpenCode（或 Hermes、Codex、Claude Code）是怎么处理这个的，以及我们该从中借鉴什么。"*

**要测验自己**：

- *"就这一章考考我。问我五个问题，从简单到难。"*

如果你不是技术出身，这份清单顶部的那几个 prompt 是你用得最多的。agent 来写代码；你的任务是读它写出来的东西、判断它是否说得通、然后问下一个问题。纸面上的那些词会比你预想的更快地不再像行话。

## How this turns into your own project

个性化不是课程结尾才做的一个独立步骤。它是一章接一章地发生的，就在你和 agent 的对话里。

到 Ch.02，你会有一个调用单个 tool 的微型 loop。到 Ch.05，它会在多轮之间记住一些东西。到 Ch.10，它能把任务委派给一个更小的专家。到 Ch.16，你能看到它花了多少成本、时间都耗在了哪里。到 Ch.22——最后一章——你和你的 agent 坐下来，走一遍一张设计画布，决定你真正的系统需要什么、不需要什么。

你也可以打乱顺序。如果你已经明确了自己的项目——一个 research assistant、一个内部 coding agent、一个客服工作流、一个打理个人琐事的机器人——先挑出符合它需要的那些章节，把其余的当作背景阅读。Ch.22 会帮你做选择。

目标不是读完这门课。目标是交付一个你本来就想交付的东西，并且理解它的每一行。


## The shape of the rest of the course

后面还有二十一章。它们从最小的机制一路搭建到最大的关切。

- **Ch.01–02** —— agent loop 本身：先一个 tool call，再许多个。
- **Ch.03–04** —— tools 和 prompts：与模型之间的契约。
- **Ch.05–07** —— memory：short-term、long-term，以及如何安全地写入它。
- **Ch.08** —— state 和 persistence：从崩溃中幸存。
- **Ch.09–10** —— planning 和多 agent 委派。
- **Ch.11** —— 把这一切组装进一个 harness。
- **Ch.12–14** —— humans、connectors，以及 skills/MCP/subagents：agent 向外部世界呈现的那一面。
- **Ch.15–17** —— backend、observability、cost：production 层。
- **Ch.18–19** —— safety 和 operations。
- **Ch.20–22** —— proactive agents、self-evolving agents，以及最后，设计你自己的 agent。

章节的排序保证每一章只依赖它之前出现过的内容。如果你已经有了明确的项目，可以略过暂时用不上的章节，等用得上时再回来。

## Reference systems used throughout

当某一章说到 "real system note"（真实系统笔记）时，它指的就是下面这几个之一：

- **OpenCode** —— 一个 coding-agent harness，带有 typed tools、permissions、sessions、compaction 和一个 terminal UI。
- **Hermes Agent** —— 一个 personal assistant，带有 memory、skills、cron、消息集成，以及一个后台 review loop。
- **OpenClaw** —— 一个自托管的 assistant gateway，最擅长用 channel adapters 把多个平台汇聚进单一的 assistant 边界。
- **Paperclip** —— 一个工作流控制平面：agents、issues、budgets、approvals、secrets、持久化的 Postgres state、audit logs。

它们不是供你照抄的模板。它们是用来做合理性检查（sanity check）的。如果这门课里的某个模式与其中两个或更多在 production 中的做法相符，那它就是承重的。如果只有一个这样做，那相应的 trade-off 会被写清楚。

---

## One last thing before you start

你不需要谁的许可才能开始。你不需要在打开 agent 之前读完每一章。你不需要知道自己最终会用哪个框架。你做的第一个东西会小到这些决定都还无关紧要——而一旦它摆在你面前、跑起来了，下一个决定就变得显而易见了。

今天交付最有意思的 agentic 系统的那些人，并不是经验最丰富的人。他们是最先和自己的 AI 伙伴进入一个紧密 loop、并且在里面待得最久的人。这门课存在的意义，就是让你身处那个 loop 中时，知道该问什么。

翻页吧。Ch.01 是一个 tool call。

---

<!-- nav-footer -->
<div align="center">

[📖 课程目录](../../README_zh.md) · [下一章：Ch.01 One tool call ➡️](01-function-calling.md)

</div>
