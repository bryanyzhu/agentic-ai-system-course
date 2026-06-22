# Chapter 10 — Multi-agent delegation

## TL;DR

A multi-agent system is one agent (the parent) running another agent (the subagent) as a bounded unit of work. Done well, it isolates a subtask so the parent's context stays clean and the subagent can use a different tool set, model, or trust boundary. Done badly, it produces a vague *"look into this"* with unlimited tools and no output contract, and you debug it for a week. This chapter covers the delegation packet, the result contract, sync vs async and sequential vs parallel patterns, recursion caps and isolation modes, supervisor vs specialist topologies, and how to decide whether delegation is the right move or just a more expensive tool call.

---

## Why this matters

The first time you build a multi-agent system you discover three things at once: the subagent uses more tokens than you expected, returns more text than you wanted, and made decisions you cannot audit. Each is a contract failure. The packet was vague. The result schema didn't exist. The audit trail was implicit.

The second-time benefit is the reason to learn it anyway: a well-shaped delegation is the cheapest way to make an agent specialize. The parent stays general; the subagent gets a tight role, a small tool set, and a model fit for the task. The whole system costs less and reasons better than a single all-knowing agent would.

---

## The concept

### When to delegate (and when not to)

Reach for delegation when at least one is true:

- The subtask needs **its own context** — different system prompt, different memory, different focus.
- The subtask should **isolate side effects** — a worktree, a sandbox, a separate trust boundary.
- The subtask wants a **different model or tool set** — cheap model for a narrow lookup, expensive model for deep reasoning.
- The subtask is **safely parallelizable** with other subtasks — three reviews in parallel, then synthesize.

Do *not* delegate when:

- A deterministic tool can answer the question.
- A skill can teach the parent to do it.
- The child would need the parent's full context anyway (you'd pay context cost twice).
- The subtask is too small to justify another model loop (delegation has setup cost — system prompt, tool list, packet construction).

The cheapest improvement most teams skip: ask whether each delegation is replacing a tool call that would have been cheaper.

### The delegation packet

What the parent sends to the subagent is a *packet*, not a transcript:

```ts
type DelegationPacket = {
  role:            string;       // "researcher" | "reviewer" | "implementer" | ...
  objective:       string;       // the subtask, in prose
  context:         string;       // filtered slice, NOT the full parent transcript
  allowedTools:    string[];     // tighter than parent's
  constraints:     string[];     // "do not write outside /tmp", "max 10 file reads"
  maxSteps:        number;       // hard cap
  budget?:         { tokens?: number; cost?: number };
  outputSchema:    JsonSchema;   // what the result must look like
  remainingDepth:  number;       // delegation depth left (see Recursion caps)
};
```

A few rules from production:

- **Don't dump the parent transcript by default.** Summarize, or pick the few messages the subagent actually needs. Dumping increases token cost, prompt-injection surface, and the chance the subagent goes off task.
- **Tighten the tool list.** A reviewer subagent gets read tools only. An implementer gets writes scoped to a worktree. An external researcher gets web tools but no shell.
- **Pass the remaining delegation depth.** Every spawn decrements it. When it hits zero, no more spawning.

```mermaid
flowchart LR
    P["Parent agent"] --> Pkt["Delegation packet<br/>(role, objective, context,<br/>tools, budget, schema)"]
    Pkt --> Child["Child agent loop"]
    Child --> Out["Structured result"]
    Out --> Synth["Parent synthesis"]
    Synth --> P
```

### The result contract

What comes back must be checkable. A bare paragraph is a contract failure waiting to happen. Production systems land on something like:

```ts
type ResearchResult = {
  answer:       string;
  evidence:     Array<{ source: string; quote: string }>;
  uncertainty:  "low" | "medium" | "high";
  followups:    string[];
  toolsUsed:    string[];      // for audit (Ch.16)
  cost?:        number;        // for the parent's budget rollup
};

function validateAgainstSchema(result: unknown, schema: JsonSchema) {
  // Reject the subagent's output if it doesn't match.
  // Bad output is a recoverable error — the parent can retry
  // with a corrective prompt or fail loud.
}
```

Structured outputs let the parent reason mechanically: validate the schema, score the confidence, compare across siblings, surface to the user. Unstructured outputs force the parent to call the model again to interpret them — a second hidden cost on every delegation.

### Synchronous vs asynchronous; sequential vs parallel

Two orthogonal axes:

- **Synchronous** — the parent waits for the subagent. Most production setups (OpenCode's `task` tool, Hermes Agent's `delegate_task`).
- **Asynchronous** — the subagent runs in a background thread or process. Hermes Agent's `spawn_background_review_thread` is the canonical reference; Paperclip's heartbeat scheduling is async at the system level.

- **Sequential** — parent delegates A, waits, then delegates B. The result of A informs B.
- **Parallel** — parent spawns A, B, C at once; they run independently; parent synthesizes when all return.

```ts
// Parallel, when inputs are truly independent.
const [api, ui, db] = await Promise.all([
  delegate(apiReviewPacket, ctx),
  delegate(uiReviewPacket, ctx),
  delegate(dbReviewPacket, ctx),
]);
const final = await synthesize([api, ui, db], ctx);

// Sequential, when one result shapes the next packet.
const investigation = await delegate(investigationPacket, ctx);
const patchPlan      = await delegate(buildPatchPlanPacket(investigation), ctx);
const final          = await synthesize([investigation, patchPlan], ctx);
```

Parallel saves wall-clock time; sequential keeps reasoning ordered. Mix them — parallel for the gathering phase, sequential for the synthesis phase.

### Recursion caps and the depth-1 default

A subagent that can spawn its own subagents is a stack overflow waiting to happen. Three patterns in production:

- **Depth-1 default** (the most common production choice): the parent can spawn subagents; subagents cannot spawn further subagents. Safest, simplest, and what you should start with unless a concrete need forces otherwise.
- **Bounded depth** (OpenClaw at depth 5): allowed up to a small limit; exhaustion throws.
- **Topology cap** (Paperclip): no in-loop spawning at all; the scheduler dispatches; the agent's parent/child relationships are tracked as data, not stack frames.

```ts
function assertCanSpawnChild(ctx: AgentContext) {
  if (ctx.remainingDelegationDepth <= 0) {
    throw new Error("Delegation depth exhausted; flatten or hand off via supervisor");
  }
}
```

A subtle gotcha: depth caps are usually count-based, but two subagents at depth N−1 can each spawn one child, doubling the effective work at depth N. If cost matters more than nesting, switch to a *cost-based* cap — total spawned tokens, not nesting count.

### Isolation modes

What level of separation each child gets:

| Mode | What's isolated | Cost | When |
|---|---|---|---|
| **Same process, shared memory** | Just the system prompt and tool set | Cheapest | Quick specialist queries |
| **Separate session, shared store** | Memory namespace, audit log | Low | Most subagent uses |
| **Worktree** | Filesystem (git worktree per subagent) | Medium | Code edits that mustn't touch main |
| **Sandbox** | OS-level isolation (Docker, Modal, Vercel) | High | Untrusted execution |
| **Separate process / adapter** | Full process boundary | Highest | Different runtime; channel adapter style |

OpenCode supports worktree isolation. Hermes Agent's tool environments (`tools/environments/`) support Docker, SSH, Modal, Vercel Sandbox at the per-tool level. Paperclip runs each adapter in a separate process. The choice is a trust-and-budget decision: higher isolation costs more but contains more.

The memory and recall side — what the subagent can read from and write to — is covered by Ch.06 (recall boundaries) and Ch.07 (write-back boundaries). Pick the same answer across both; mixed policies (subagent can read everything but write nothing) usually work; the opposite (write but not read) almost never does.

### Parallel work on shared artifacts

When subagents run in parallel on related artifacts (three reviewers across the same codebase, two implementers editing different sections of the same document), pick a coordination shape *before* spawning. Two patterns cover almost every case:

- **Isolated edit + merge at synthesis.** Each subagent works in its own worktree, sandbox, or namespace; the parent merges outputs when all return. Overlaps surface as merge failures resolved at a single point — by the parent's synthesis step (deterministic merge when the edits are disjoint), by a reviewer specialist (semantic merge when they overlap cleanly), or by the user (when overlap is real). This is the safer default; it pushes conflicts to one resolution point instead of letting siblings race in shared state.
- **Shared blackboard.** A small structured store (a JSON file, a Redis hash, a database row) that siblings can read and write during their runs — useful for *"I already checked `auth.ts`, skip it"*-style coordination. The blackboard inherits the locking and CAS discipline from Ch.07 (atomic writes) and Ch.08 (CAS transitions); a blackboard without those is a race condition pretending to be a coordination pattern.

For coding agents specifically, worktree isolation plus a post-synthesis merge step is the established pattern: each subagent gets its own checkout, the parent inspects the diffs side by side, and the merge is either deterministic (no overlap) or surfaced for resolution (overlap detected). Letting parallel subagents race on a single repo state is the most expensive class of multi-agent coding bug — partial, mutually inconsistent edits that look plausible per file and break on integration. The cost of one extra worktree is much smaller than the cost of unwinding that.

### Supervisor vs specialist topology

Two roles repeat across systems:

- **Supervisor / orchestrator** decides who runs, in what order, with what inputs. Often the main agent loop. Paperclip's heartbeat service is a control-plane-level supervisor.
- **Specialist** is a tightly scoped subagent with a narrow tool set and a clear role — `explore`, `review`, `summarize`, `extract`. The specialist does not decide what to do; the supervisor decides.

```mermaid
flowchart TD
    User["User request"] --> Sup["Supervisor / orchestrator"]
    Sup -->|"plan packet"| Spec1["Planner specialist"]
    Sup -->|"review packet"| Spec2["Reviewer specialist"]
    Sup -->|"impl packet"| Spec3["Implementer specialist"]
    Spec1 --> Sup
    Spec2 --> Sup
    Spec3 --> Sup
    Sup --> User
```

The pattern that scales: name your specialists. Each has a system prompt, a tool list, a result schema, and a one-line description. The supervisor picks by name. OpenCode's built-in agent profiles (`build`, `plan`, `general`, `explore`) are the canonical reference; you usually add a few custom profiles per project as new specialist needs surface.

### Per-subagent restrictions

Every restriction the parent puts on a specialist is also a Ch.04 win. A specialist with three tools has a shorter system prompt (more cache reuse across specialists). A specialist with a cheaper model costs less per call. The savings compound across many delegations.

In practice:

- **Tools.** Explicit allowlist per role; deny by default. (Ch.03's metadata flags tell the supervisor which tools are safe for which specialist.)
- **Model.** Cheap and fast for narrow tasks; reasoning model for genuinely hard subproblems.
- **Memory.** Scoped per Ch.06; usually read the parent's namespace, write to its own.
- **Approval gates.** If the specialist can take destructive action, it inherits the parent's permission rules — Ch.12 covers the gate.

### Context handoff

The single biggest cost of a subagent is the context the parent passes it. Three patterns, from cheapest to richest:

- **Fresh system prompt + objective only.** The subagent starts clean. Cheapest. Works when the objective contains all the context.
- **Summarized handoff.** The parent's compaction (Ch.05) summarizes the relevant turns into a `<context>` block. Medium cost; usually right.
- **Filtered transcript slice.** The parent picks the last N turns or all turns matching some filter. Most expensive; reserve for cases where the subagent genuinely needs the original wording.

A useful rule from Ch.05: the parent's *compact* operating transcript is usually a better starting point for handoff than the full audit log. The compaction already chose what matters.

### Subagent output discipline

A specialist that writes paragraphs when one sentence would do is a token leak. The parent should enforce:

- **Terse final answer.** A handful of sentences, or a structured object. Anything longer is a synthesis failure.
- **No intermediate noise.** The parent should not see the subagent's tool calls or reasoning *in its prompt context* by default — only the final answer. (OpenCode's `task` tool does this; Hermes Agent's `StreamingContextScrubber` hides injected memory from the parent's view.) This is a *prompt-context* rule, not an *audit* rule: the subagent's tool calls, reasoning, and intermediate turns are still recorded to the audit log (Ch.05) and the trace pipeline (Ch.16), and stay inspectable for debugging, replay, and post-hoc review. Hide from the parent's prompt to save tokens and keep the parent focused; never hide from the operator.
- **Cited evidence where the answer requires it.** Each load-bearing claim gets a source the parent can check.

Specialists trained to be terse are usually trained the same way as Ch.05's summarizer: explicit purpose in the system prompt, structured output schema, low temperature for the synthesis step. The model can do it; the parent has to ask for it.

### Subagent failure handling

A subagent can fail in three distinguishable ways:

- **Recoverable** (e.g., schema validation failed). The parent retries with a corrective prompt, capped at 1–2 attempts.
- **Permanent** (e.g., tool not available, credentials invalid). The parent surfaces the failure and either tries a different specialist or fails up to the user.
- **Silent** (e.g., the output validates but the answer is wrong). The hardest. Defenses live in the result schema (confidence field, citations, structured fields) and in cross-validation (a second subagent reviews the first).

Track subagent success rates over time. A specialist that fails 30% of the time is either poorly scoped or pointed at the wrong tasks; either way it is a Ch.16 signal worth catching early.

### The supervisor in a long-running control plane

A pattern that earns its own mention because it does not look like a subagent: a supervisor that lives *outside* the agent loop, across many runs. Paperclip's heartbeat service is exactly this. It schedules, retries, watches for orphans, enforces budgets, and routes work to agents. The "agents" it supervises are not in-process subagents — they are full agent runs that may span minutes or hours.

This pattern matters for production systems where work outlives a single agent invocation: long-running automations, multi-step approvals, async user interactions. The supervisor is the durable layer; the agents are the workers. Ch.08's persistence and state machine are the foundation it stands on. Treat the supervisor itself like a Ch.08 run: state machine, atomic claim, heartbeat, reaper.

### Background subagents

The simplest non-blocking delegation: a daemon thread that runs after a successful turn and writes back to memory or skills. Hermes Agent's background review fork is the canonical reference (covered from the memory-writing angle in Ch.07). Use it for *"decide whether to remember anything from this session"* or *"summarize the day's work in the background"* — not for anything the user is waiting on.

Constraints to honor:

- Background subagents should use a different (usually cheaper) model.
- A restricted tool set — typically memory and skill tools only.
- Their results are visible *next session*, not this one. The Ch.04 cache rule applies in reverse: don't mutate the running prompt from a background process.

### Verification and cross-checking

A more recent pattern, not yet widespread in the references but worth flagging: spawn a *second* subagent whose only job is to review the first one's output against the same context. The reviewer specialist gets the original packet plus the first subagent's result and returns either *approve* or *issues with this answer*. Cheap insurance against silent failures.

Two practical notes: keep the reviewer's tool set tighter than the worker's (usually read-only), and budget the reviewer at a fraction of the worker's cost — a reviewer that costs more than the work it reviews is not worth the call.

---

## Real-system notes

- **OpenCode** ships the cleanest in-process delegation reference: a `task` tool that spawns child sessions with filtered context and an `Agent.Service.handleSubtask` flow that returns a single structured observation to the parent. Built-in `build` / `plan` / `general` / `explore` profiles show the supervisor/specialist split.
- **Hermes Agent** is the reference for both styles: synchronous `delegate_task` for in-line subagents and `spawn_background_review_thread` for async background subagents with a tightly restricted tool whitelist.
- **Paperclip** is the control-plane pattern: a supervisor (heartbeat scheduler) routes issues to agents, tracks `parent_run_id` lineage, and enforces budgets and approvals across runs. Recovery tasks can request a lighter model via `assigneeAdapterOverrides` — model selection per subagent at the orchestration level.
- **OpenClaw** uses channel adapters as a form of delegation across process boundaries: inbound messages dispatch to the underlying agent runtime; the adapter is the boundary. Useful reference for *"the subagent is a different process."*

---

## Common failure cases

*These failures are durable; their fixes evolve fastest — each names the pattern and leaves current specifics to you and your AI partner.*

- **Fan-out triples the token bill.** One task split into a fan of subagents costs many loops plus synthesis, and no single trace looks alarming. *Fix: a fan budget rolled up to the parent run, with a cost-based cap on the whole tree rather than per-leaf counts.*
- **The subagent returns a wall of text.** Good work wrapped in prose the parent has to spend another model call to interpret, eating cache and context. *Fix: terseness as a validated constraint — a bounded result schema and a hard output cap, reject-and-retry on overrun.*
- **Parallel subagents corrupt a shared artifact.** Siblings edit the same state at once and the merge breaks in ways no single output explains. *Fix: partition the work so siblings can't collide, falling back to real concurrency control (Ch.08) only where overlap is unavoidable.*
- **A subagent returns a confident wrong answer.** The result has the right shape but false content, and schema validation can't catch it. *Fix: adversarial cross-check with independent access to the evidence and checkable citations, not a self-graded confidence field.*
- **A background subagent dies and the parent waits forever.** An async worker crashes and the result never lands; the failure is an absence with no exception to catch. *Fix: treat async delegation as a leased run with a deadline and a fan-in policy for partial returns (Ch.08).*

---

## Pair with your agent

A few prompts that work well on this chapter:

- *"For each tool I currently call, decide whether it should stay a tool or become a delegation to a specialist subagent. Apply the four criteria from this chapter and explain each decision."*
- *"Design two specialist subagents for my project: a `reviewer` (read-only, cheap model, terse structured output) and an `implementer` (worktree-isolated, expensive model). Write both system prompts and the result schemas, plus the supervisor logic that decides when to call each."*
- *"Wire the delegation packet from this chapter into my codebase. Add the `remainingDepth` field and the `assertCanSpawnChild` guard. Write a test that proves a depth-2 nested spawn fails cleanly with a useful error message."*
- *"Take one of my multi-step research tasks and refactor it as parallel delegation with a synthesis step at the end. Compare wall-clock time and total cost against the sequential version."*
- *"Pick three of my common subagent failures from the last week. Classify each as recoverable / permanent / silent. For each class, write the parent-side handling code and show me the audit trail it generates."*
- *"Add a background review subagent that runs after each successful turn, with the tool whitelist `{memory, skill_manage}`. Make sure its writes only become visible to the parent next session (Ch.04 rule). Verify with the prefix fingerprint."*
- *"For my agent, log subagent success rate by specialist over the last month. If any specialist fails more than 20% of the time, propose either a tighter scope or a different model."*
- *"Implement a reviewer subagent that double-checks any output from my `implementer` specialist before it returns to the parent. Budget the reviewer at 30% of the implementer's token spend; reject and retry if the reviewer disagrees."*

---

## What's next

You now have a parent agent that can plan, a way to express subagent work as bounded packets, and the discipline to keep delegation focused. Ch.11 puts everything from Ch.01–10 together as a single harness — the loop, the tool registry, the prompt builder, the memory layer, the persistence engine, the planner, the delegation surface — into one composable architecture you can adapt to your stack.

---

<!-- nav-footer -->
<div align="center">

[⬅️ Prev: Ch.09 Planning patterns](09-planning-patterns.md) · [📖 Course index](../README.md) · [Next: Ch.11 The agent harness ➡️](11-agent-harness.md)

</div>
