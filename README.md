# Agentic System Course - Use Agent to Learn Agent

> English · [中文](README_zh.md)

**Join the [discord channel](https://discord.gg/dWSnHAFdpb) if you want to learn and build together!**

---

This is a 22-chapter skeleton course on how to design, build, and operate production AI agents — written to be read with your own AI partner at your side. **An agentic system** is an AI system that can autonomously pursue goals by planning, making decisions, using tools, adapting based on feedback, having memory, etc — instead of only responding to a single prompt. Similar to [Andrej Karpathy's idea file on LLM-wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), **this course is giving you the skeleton and your agent will help you put the muscles on it**.

**This course is**:

- A *skeleton* — load-bearing topics, patterns, and decisions, with trade-offs.
- Written to age slowly. Framework specifics rot fast; architectural patterns do not.
- A file pair (course + AGENTS.md) designed for AI consumption as much as human reading.

**This course is not**:

- A step-by-step tutorial. There is no walked-through project.
- Tied to one stack. The course never says "use LangChain" or "use Pydantic AI." Your AI partner suggests the stack that fits your project.
- A reference manual. When you need an exact API signature, ask your AI or read the docs.


---

## How to start

Clone the repo, open it in your usual IDE to view course content. At the same time, point your AI agent (Claude code/Codex) at the project root, and try one of these prompts when you study a chapter:

- *"Give me three real-world examples of where this matters."*
- *"Suppose you are interviewing me, quiz me on this topic with five follow-up questions, easy to hard."*
- *"What's a question I should be asking that I haven't?"*
- *"I just read about [pattern X]. I am building [your project]. Translate the pattern into the smallest version that works in whatever language and tools fit, and explain each piece as you write it."*
- *"Forget my project for a moment — show me how OpenCode (or Hermes Agent, or any leading coding agent) handles this, and what we should borrow from it."*

You can also just point your agent at Ch.22's design canvas and walk through it with your specific project in mind — that's the fastest path from "I have an idea" to "I have a spec."

---

## Built-in skills

### `agentic-system-reviewer`

Reviews PRDs, design docs, implementation plans, or agent code against the course. You can run this skill on **any agentic system** to get course-grounded feedback — your own project, an open-source agent you're studying, a PRD before any code exists, or a coworker's repo you want a second opinion on. The skill calibrates scope first (hobby / team tool / customer-facing), picks the chapters that matter for your archetype, reads them, and produces a findings-first report with severity, evidence, course citations, and concrete fixes — not a generic "looks good" or "add safety" review.

In Claude Code, just describe what you want — *"review this against the course"*, *"is this agent design good?"*, *"what chapters does this miss?"* — while the agent is pointed at the target repo or doc. The skill auto-loads when its description matches your intent.

**Codex users:** use Codex's official `skill-creator` skill to port this skill over.

---

## Course structure

| Chapters | Theme |
|---|---|
| **Ch.00** | How to use this course with your AI partner |
| **Ch.01–04** | Foundations: one tool call → the loop → tools as contract → prompts & cache |
| **Ch.05–08** | Memory and state: short-term → long-term → writing & curation → persistence |
| **Ch.09–11** | Coordination: planning → multi-agent delegation → the harness |
| **Ch.12–14** | External surface: human-in-the-loop → connectors/MCP → skills/MCP/subagents |
| **Ch.15–17** | Production scale: backend → observability → cost, latency, model strategy |
| **Ch.18–19** | Quality and ops: safety/adversarial inputs → operations and forward-deployed |
| **Ch.20–21** | Agency: proactive agents → self-evolving agents |
| **Ch.22** | Design canvas: designing your own agent |

### Chapter index (click to open)

- [Ch.00 · How to use this course](course/00-how-to-use-this-course.md)
- [Ch.01 · One tool call](course/01-function-calling.md)
- [Ch.02 · The agent loop](course/02-the-agent-loop.md)
- [Ch.03 · Tools the agent can trust](course/03-tools-validation.md)
- [Ch.04 · Prompts, context & cache](course/04-prompts-context-cache.md)
- [Ch.05 · Short-term memory](course/05-short-term-memory.md)
- [Ch.06 · Long-term recall](course/06-long-term-recall.md)
- [Ch.07 · Memory writing & curation](course/07-memory-writing.md)
- [Ch.08 · State and persistence](course/08-state-and-persistence.md)
- [Ch.09 · Planning patterns](course/09-planning-patterns.md)
- [Ch.10 · Multi-agent delegation](course/10-multi-agent-delegation.md)
- [Ch.11 · The agent harness](course/11-agent-harness.md)
- [Ch.12 · Human in the loop](course/12-human-in-the-loop.md)
- [Ch.13 · Connectors, MCP, IPC](course/13-connectors-mcp-ipc.md)
- [Ch.14 · Skills, MCP & subagents](course/14-skills-mcp-subagents.md)
- [Ch.15 · Backend infrastructure](course/15-backend-infrastructure.md)
- [Ch.16 · Observability](course/16-observability.md)
- [Ch.17 · Cost, latency & model strategy](course/17-cost-latency-model-strategy.md)
- [Ch.18 · Safety & adversarial inputs](course/18-safety-adversarial-inputs.md)
- [Ch.19 · Ops & forward-deployed](course/19-ops-and-forward-deployed.md)
- [Ch.20 · Proactive agents](course/20-proactive-agents.md)
- [Ch.21 · Self-evolving agents](course/21-self-evolving-agents.md)
- [Ch.22 · Designing your own agent](course/22-designing-your-own-agent.md)

The chapters are ordered so each one only assumes what came before. If you have a clear project, skim chapters that do not apply yet and come back when they do. Suggested learning paths by project shape (coding agent, personal assistant, multi-tenant tool, research agent, just exploring) are in `CLAUDE.md`.

The goal is not to finish the course. *The goal is to ship something you wanted to ship anyway, and to understand every line of it.*

---

## Optional: reference systems

The course occasionally points at four open-source systems for grounded examples:

- **OpenCode** — coding agent (terminal-first, typed tools, sessions, compaction)
- **Hermes Agent** — personal assistant (memory, skills, cron, channels)
- **OpenClaw** — self-hosted personal-assistant gateway (channel adapters)
- **Paperclip** — workflow control plane (multi-agent orchestration, durable Postgres state)

You do **not** need to clone any of these to get value from the course, these four are sanity checks. If you find yourself wanting grounded answers (*"how does X actually do Y in code?"*), your AI partner will offer to clone the relevant repo on demand, or you can run setup beforehand:

```bash
./setup.sh
```

This clones all four reference repos to `references/`. Idempotent (safe to re-run). Env-var-overridable if you want to point at forks or pinned commits. See the script for details.

If you want to ask questions about other sources, feel free to put them under `references/` and chat about it.

---

## Repository layout

```
course/         — 22 chapters (read-only by design — this is the skeleton)
CLAUDE.md       — Behavioral guide for your AI partner (read by Claude Code)
AGENTS.md       — Same content as CLAUDE.md, named for Codex / Cursor / other agents
README.md       — This file
setup.sh        — Optional: clone the four reference repos
.claude/skills/ — Built-in skills your AI partner can invoke (reviewer; more coming)
docs/           — Written notes your AI partner produces on request (created on demand)
questions/      — Daily Q&A log your AI partner writes automatically (created on demand)
workspace/      — Code, exercises, and project builds (created on demand)
references/     — The four reference repos if you (or your agent) clone them
```

`CLAUDE.md` and `AGENTS.md` are duplicates — same content under two filenames because different agents look for different files by default. Both are structured the same way the course is: a durable, opinionated file any AI assistant can read in one pass and turn into useful behavior.

---

## One last thing before you start

You do not need permission to start. You do not need to read every chapter before opening your agent. You do not need to know which framework you will end up using. The first thing you build will be small enough that none of those decisions matter — and once it's in front of you, working, the next decision becomes obvious.

The people shipping the most interesting agentic systems today are not the ones with the most experience. **They are the ones who got into a tight loop with their AI partner first and stayed in it longest.** This course exists so that when you are in that loop, **you know what to ask**.


---

## License

See `LICENSE`. The course content is open for educational use. Reference systems retain their original licenses.
