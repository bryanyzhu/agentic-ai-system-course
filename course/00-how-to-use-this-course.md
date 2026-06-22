# Chapter 00 — How to use this course

## TL;DR

You are about to learn how production-grade agentic systems are designed — not by reading thousands of lines of someone else's code, but by walking a map of the patterns and pairing with your own AI agent to apply them to whatever you actually want to build. This chapter explains why the course is written the way it is, how to read it with an agent at your side, and how to personalize every chapter into something you can ship. Everything is about you and your project.

---

## Why this course exists

In early 2026, I have watched the same thing happen over and over. Someone who has never written a line of code opens Claude or Codex, describes a problem, and forty minutes later is staring at a working prototype they understand line by line. A senior engineer who used to spend a week reading a new framework now skims the docs, asks their agent five sharp questions, and is shipping the same afternoon. The starting points differ. The loop is the same: a curious human, an honest question, an AI partner that explains, scaffolds, and runs the code right there in the conversation.

This way of learning is roughly **ten times faster** than watching videos or following a step-by-step tutorial. Videos are passive and frozen in time. Tutorials over-fit to one stack and rot the moment a new model is released. An AI agent is the opposite — interactive, current, and tailored to you. The bottleneck is no longer information. The bottleneck is knowing what to ask.

That is what this course is for. It is the map an agent cannot hand you on its own. Vendors will not give you the map either, because every vendor wants you to think their framework **is** the map. After reading the source of the most-discussed production agents in the field — OpenCode, Hermes Agent, OpenClaw, Paperclip, etc — what becomes obvious is that they share the same dozen or so ideas. Different names, different file structures, the same patterns underneath. This course is those patterns, written down once, in plain language, with the trade-offs that matter.

## What this course is, and is not

This course is a **spine** — the load-bearing topics, patterns, and decisions that show up in every serious agentic system. It includes diagrams, vocabulary, failure modes, design choices, and notes from real production systems. It is opinionated about *what to think about* and almost silent about *which library to import*.

This course is **not** a step-by-step tutorial. There is no "first install this, then paste that." There is no project we walk through together top to bottom. There are no quizzes, no capstone repos to clone, no notebooks that only run on one stack.

That is on purpose. The details that age fastest in a course — SDK versions, model names, framework APIs, prices — are the very details your agent already has the most up-to-date answers about. The details that age slowest — what a tool registry really is, when to use a checklist vs. a graph for planning, how memory poisoning happens, why caches break, when to put a human in the loop — are what you will find here.

Think of it like this: I am giving you the skeleton. Your agent will help you put the muscles on it.

## How to actually use this course

The course works the same whether you have ten years of engineering experience or have never touched a terminal. The difference is which prompts you give your agent. Pick the ones that match where you are right now — and feel free to use the simplest ones for every chapter, no matter your background. *"Explain it to me like I'm 13"* never goes out of style.

**To understand a chapter** (works for any reader, technical or not):

- *"Explain this chapter to me like I don't know any of the jargon. Define each term the first time you use it."*
- *"Give me three real-world examples of where this matters."*
- *"What's the one thing I should remember from this chapter?"*
- *"What's a question I should be asking that I haven't?"*

**To apply it to your project** (once you have something in mind):

- *"I just read about [pattern X]. I am building [your project]. Translate the pattern into the smallest version that actually works in whatever language and tools fit, and explain each piece as you write it."*
- *"Here are the failure modes [Ch.NN] warned about. Look at what we built earlier and tell me which of these are already a risk."*
- *"Show me two ways to do this and the trade-offs in three bullets each — not paragraphs."*

**To compare against real systems** (when you want to ground an idea):

- *"Forget my project for a moment — show me how OpenCode (or Hermes, or Codex, or Claude Code) handles this, and what we should borrow from it."*

**To test yourself**:

- *"Quiz me on this chapter. Ask me five questions, starting easy and getting harder."*

If you are not technical, the prompts at the top of this list are the ones you'll use most. Your agent writes the code; your job is to read what it produced, decide whether it makes sense, and ask the next question. The words on the page stop feeling like jargon faster than you'd expect.

## How this turns into your own project

Personalization is not a separate step at the end of the course. It happens chapter by chapter, in the conversations you have with your agent.

By Ch.02 you will have a tiny loop that calls one tool. By Ch.05 it will remember things between turns. By Ch.10 it can delegate to a smaller specialist. By Ch.16 you can see what it cost and where it spent its time. By Ch.22 — the final chapter — you sit down with your agent, walk through a design canvas, and decide what your real system needs and what it does not.

You can also flip the order. If you already know your project — a research assistant, an internal coding agent, a customer-support workflow, a personal life-admin bot — pick the chapters that match what it needs first, and treat the rest as background reading. Ch.22 will help you choose.

The goal is not to finish the course. The goal is to ship something you wanted to ship anyway, and to understand every line of it.


## The shape of the rest of the course

There are twenty-one more chapters. They build from the smallest mechanism up to the largest concerns.

- **Ch.01–02** — the agent loop itself: one tool call, then many.
- **Ch.03–04** — tools and prompts: the contract with the model.
- **Ch.05–07** — memory: short-term, long-term, and how to write it safely.
- **Ch.08** — state and persistence: surviving a crash.
- **Ch.09–10** — planning and multi-agent delegation.
- **Ch.11** — putting it all together into a harness.
- **Ch.12–14** — humans, connectors, and skills/MCP/subagents: the surface the agent presents to the outside world.
- **Ch.15–17** — backend, observability, cost: the production layer.
- **Ch.18–19** — safety and operations.
- **Ch.20–22** — proactive agents, self-evolving agents, and finally designing your own.

The chapters are ordered so each one only assumes what came before. If you have a clear project, you can skim chapters that do not apply yet and come back when they do.

## Reference systems used throughout

When a chapter says "real system note," it is pointing at one of these:

- **OpenCode** — a coding-agent harness with typed tools, permissions, sessions, compaction, and a terminal UI.
- **Hermes Agent** — a personal assistant with memory, skills, cron, messaging integrations, and a background review loop.
- **OpenClaw** — a self-hosted assistant gateway, strongest at channel adapters fanning many platforms into one assistant boundary.
- **Paperclip** — a workflow control plane: agents, issues, budgets, approvals, secrets, durable Postgres state, audit logs.

These are not templates to copy. They are sanity checks. If a pattern in this course matches what two or more of them do in production, it is load-bearing. If only one does it, the trade-offs are spelled out.

---

## One last thing before you start

You do not need permission to start. You do not need to read every chapter before opening your agent. You do not need to know which framework you will end up using. The first thing you build will be small enough that none of those decisions matter — and once it is in front of you, working, the next decision becomes obvious.

The people shipping the most interesting agentic systems today are not the ones with the most experience. They are the ones who got into a tight loop with their AI partner first and stayed in it longest. This course exists so that when you are in that loop, you know what to ask.

Turn the page. Ch.01 is one tool call.

---

<!-- nav-footer -->
<div align="center">

[📖 Course index](../README.md) · [Next: Ch.01 One tool call ➡️](01-function-calling.md)

</div>
