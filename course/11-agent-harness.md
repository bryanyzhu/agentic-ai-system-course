# Chapter 11 — The agent harness

## TL;DR

The harness is the runtime around the model. Every chapter from Ch.01–10 has been about one piece of it: the loop, the tools, the prompt, the memory, the persistence, the planning, the delegation. This chapter is about composing those pieces into a single program with a clear lifecycle (bootstrap → tick → shutdown), a well-defined hook surface for extension, a configuration model that does not leak secrets, and a clean boundary between the harness itself and the application code that uses it. The model brings judgment; the harness brings structure. By the end of this chapter you should be able to look at any production agent and name its components, its lifecycle, and what plugs into where.

---

## Why this matters

Three failure modes you can avoid by knowing what a harness is.

The first: you write the tool dispatcher inline in the loop, write the prompt builder inline in the dispatcher, write the memory layer inline in the prompt builder. Six weeks later you cannot extend any one piece without breaking the others. The harness exists so that each chapter's component has a clean interface and a known place to live.

The second: you have great components but no lifecycle. The DB connects after the first tool call, the plugin loader runs after the first model invocation, the heartbeat starts before the migration finishes. The harness defines a startup sequence so this stops being a surprise.

The third — Anthropic puts it well in their long-running-apps essay: *every component in a harness encodes an assumption about what the model can't do on its own.* Without that framing, harnesses accrete features long after the underlying model has stopped needing them. The harness is not a permanent monument; it is scaffolding that should evolve as the model evolves.

---

## The concept

### What a harness is, and what it is not

The harness owns: the loop, the prompt builder, the tool registry and dispatcher, the memory manager, the persistence layer, the hook system, the bus, the model router. Plus the lifecycle that wires all of them together.

The harness does *not* own: what the agent believes, the specific skills it accumulates, the prompts of specific tools, the business logic of which tasks to solve. Those are application code. The same harness should be able to host an explore agent today, a customer-support agent tomorrow, and an analyst agent next week — without any of the harness changing.

A useful rule: if removing a feature would break *what task can this system solve*, it is application code. If removing it would break *how this system runs at all*, it is harness. Paperclip is the cleanest reference for this split — Paperclip itself does not call models; it spawns adapter processes (the application) and orchestrates them. OpenCode splits server / services (harness) from agent definitions (application) the same way.

### The component inventory

Ten services every production harness has, with a couple of optional ones:

```mermaid
flowchart LR
    subgraph CORE["Core loop"]
        LC["Loop controller"]
        TR["Tool registry"]
        TD["Tool dispatcher"]
    end
    subgraph PROMPT["Prompt + context"]
        PB["Prompt builder"]
        MM["Memory manager"]
        CC["Context compactor"]
    end
    subgraph STATE["State + durability"]
        PS["Persistence"]
        RR["Run state machine"]
        CK["Checkpoint store"]
    end
    subgraph PLUMBING["Cross-cutting"]
        HS["Hook surface"]
        BS["Event bus"]
        MR["Model router"]
        TS["Trace sink"]
        CFG["Config + secrets"]
    end
    CORE --> PROMPT
    CORE --> STATE
    PLUMBING --> CORE
    PLUMBING --> PROMPT
    PLUMBING --> STATE
```

Each block is a chapter you have already read. Ch.01 — loop body; Ch.02 — loop controller; Ch.03 — tool registry + dispatcher; Ch.04 — prompt builder; Ch.05 — memory manager + compactor; Ch.06–07 — memory store + writer; Ch.08 — persistence + run state + checkpoint store; Ch.09 — planner (a layer on top of the loop); Ch.10 — delegation (the supervisor lives in the loop, specialists are spawned by it). Hooks, bus, router, trace sink, and config are the cross-cutting plumbing — covered next.

The harness is the diagram. The chapters were the pieces.

### Composition: how the services wire

Three patterns appear across production harnesses, in roughly increasing order of formality:

- **Closure factories.** Each service is a function that takes its dependencies and returns an object of methods. Wiring happens in `main` / `app.ts` once. Paperclip uses this — small, explicit, easy to test by passing fakes.
- **Service registry.** Components register themselves in a typed registry at boot; consumers look up by name. Useful when there are many similar things (tools, agents, providers).
- **Layered DI.** Each service declares its dependencies via type signatures; the runtime resolves them in order. OpenCode uses Effect's `Layer.effect` for exactly this.

Pick one and stick with it. The worst harnesses are the ones that mix all three — some services injected, some registered, some imported as singletons. The same goes for whether services are async at construction or synchronous: pick a convention and hold it.

```ts
// A typed harness — services as fields, all dependencies explicit.
type Harness = {
  config:        Config;
  bus:           EventBus;
  hooks:         HookRunner;
  tracer:        TraceSink;
  prompt:        PromptBuilder;     // Ch.04
  memory:        MemoryManager;     // Ch.05–07
  tools:         ToolRegistry;      // Ch.03
  loop:          LoopController;    // Ch.02
  state:         RunStateStore;     // Ch.08
  checkpoints:   CheckpointStore;   // Ch.08
  router:        ModelRouter;       // Ch.17 (forward)
};
```

### The lifecycle: bootstrap, tick, shutdown

```mermaid
stateDiagram-v2
    [*] --> Boot
    Boot --> Ready : services initialized + health green
    Ready --> Tick : user message / scheduled trigger
    Tick --> Tick : next request
    Tick --> Draining : SIGINT / SIGTERM
    Ready --> Draining : SIGINT / SIGTERM
    Draining --> Shutdown : in-flight runs drained or deadline hit
    Shutdown --> [*]
```

Three phases, each with its own rules. Most harness bugs live at the boundaries between them — services used before boot finished, requests accepted after drain started, shutdown that does not wait for the run state machine to checkpoint.

### Bootstrap order

The boot sequence is not arbitrary — each step depends on the previous. The order that works across production systems:

1. Load and parse the config file (with env-var overrides).
2. Validate the config schema; fail fast on errors, surfacing *all* of them.
3. Substitute env vars and resolve `$secret:` references.
4. Open the database; run any pending migrations.
5. Initialize storage services (sessions, transcripts, memory store).
6. **Discover plugins** from bundled and user paths; load each plugin's *manifest* — the tools, agent profiles, hook handlers, and commands it contributes — without yet activating it.
7. Build the tool registry in deterministic order: built-ins, then plugin contributions, then config-declared (Ch.04's cache rule applies — the order is fixed at boot and does not change).
8. Build the agent registry the same way: built-in profiles, then plugin profiles, then config profiles.
9. **Activate plugin hooks** against the now-stable registries; this is the second pass.
10. Start the optional subsystems (scheduler, MCP server, WebSocket bus, cron).
11. Run health checks — DB reachable, model provider reachable, plugin handshakes OK.
12. Flip the readiness flag; start accepting traffic.

The two-pass shape is the load-bearing detail. Plugins contribute *to* the tool and agent registries, so the registries cannot be built before plugin manifests are loaded; but plugin hooks need to *fire against* a stable registry, so they cannot be activated until the registries are built. Splitting plugin loading into discover-manifest (step 6) and activate-hooks (step 9) is the simplest way to resolve the dependency without making the registry mutable at runtime (which would break Ch.04's cache stability).

Two flags worth distinguishing: *liveness* (is the process alive?) and *readiness* (is it accepting traffic?). They are separate signals to a load balancer or supervisor. Conflating them is the source of half the deploy-time outages in agent systems.

### One tick, end to end

One tick = one user message → one final answer. Every chapter's contribution shows up:

```mermaid
sequenceDiagram
    participant U as User
    participant H as Harness
    participant P as Prompt builder (Ch.04)
    participant M as Memory (Ch.05–07)
    participant T as Tools (Ch.03)
    participant S as State store (Ch.08)
    participant Pr as Provider

    U->>H: user message
    H->>S: claim run (CAS)
    H->>M: prefetch memory
    loop step boundary (Ch.02)
        H->>P: build prompt (Ch.04 cache rules)
        P-->>H: stable prefix + volatile tail
        H->>Pr: stream model call
        Pr-->>H: tool calls or final answer
        alt tool calls
            H->>T: validate + dispatch
            T-->>H: tool results (Ch.03 envelope)
            H->>S: checkpoint
        else final answer
            H->>S: mark completed
        end
    end
    H-->>U: final answer
    H->>M: background review (Ch.07)
```

Every arrow is a hook point. Pre- and post-LLM hooks bracket the model call. Pre- and post-tool hooks bracket the dispatch. Session-start and session-end hooks bracket the whole tick. Plugins extend the harness by registering handlers at these points without modifying the loop.

### Graceful shutdown

A signal handler — SIGINT or SIGTERM — flips the harness into draining mode. In drain:

- New requests are refused (or queued, depending on policy).
- In-flight runs get a deadline (typically a few minutes) to reach a step boundary and checkpoint cleanly.
- After the deadline, surviving runs are marked `cancelled` in the state machine (Ch.08); their leases will be reaped by the next instance.
- Pending background-review forks are joined or marked abandoned.
- The database connection pool drains; the bus shuts down; the process exits.

The cost of skipping graceful shutdown is invisible until the day a deploy interrupts ten long-running agent sessions; the next instance then has to figure out what happened. Ch.08's reaper covers the recovery; this chapter covers the *prevention*.

### The hook surface

Hooks are the harness's extension API. Six lifecycle points cover most production needs:

| Hook | Fires when | What it is for |
|---|---|---|
| `pre_session` | Once at session start | Inject identity, set namespaces, kick off prefetch |
| `pre_llm_call` | Before each model call | Last-chance prompt mutation, gates, redaction |
| `post_llm_call` | After each model call | Token counting, redaction, plan extraction |
| `pre_tool_call` | Before each tool dispatch | Permission check (Ch.12), arg transform |
| `post_tool_call` | After each tool returns | Redact secrets, attach metadata, log |
| `post_session` | Once at session end | Background review (Ch.07), cost rollup, archive |

The harness fires each hook in registration order, passing a typed context object. Plugins return a directive (`continue`, `modify`, `deny`) and any side effects (logs, events) that go through the harness rather than mutating shared state directly. Hermes Agent and OpenClaw both register hooks this way; OpenCode's bus-event model is a close cousin.

Two rules from production:

- **Hooks must be idempotent.** A retried step (Ch.08) fires the same hooks again. If a hook writes a counter, increment with an idempotency key.
- **Fail-open vs fail-closed depends on the hook's job.** *Observational* hooks (tracing, metrics, plain logging, post-hoc transforms) are fail-open: a failure is logged and the loop continues. *Gating* hooks — security (Ch.18), approval (Ch.12), redaction, policy — must be fail-closed: a failed approval hook means the action is *not* approved; a failed redaction hook means the unredacted bytes never reach the next stage; a failed policy hook means the operation is denied. Tag each hook at registration with its failure semantics; the harness routes the failure based on the tag. Defaulting all hooks to fail-open is a vulnerability disguised as resilience.

### Provider abstraction (and its leaks)

The harness wraps providers behind a uniform interface so the loop, tools, and prompts do not care which one. In practice this is a *leaky* abstraction with three known holes:

- **Tool schema format** differs by provider (Anthropic uses `input_schema`; OpenAI uses `function.parameters`). The adapter normalizes on the way in.
- **Streaming events** differ (Anthropic emits `content_block_delta` and `tool_use`; OpenAI emits `choice.delta.tool_calls[i].function.arguments` fragments). Each provider has its own transport adapter.
- **Cache control syntax** is provider-specific (Ch.04 covers the Anthropic explicit-marker and OpenAI automatic-prefix shapes in detail). Apply it only inside the adapter that owns it; pass through for providers that do not support the marker.

```ts
// A clean provider interface lives behind every harness's loop.
// metadata() is capability negotiation — the harness asks what the
// provider supports and adapts requests, rather than hard-coding it.
interface ModelProvider {
  stream(req: ModelRequest): AsyncIterable<ProviderEvent>;
  countTokens(text: string): number;
  metadata(): {
    contextWindow:             number;
    maxOutput:                 number;
    supportsCacheControl:      boolean;
    supportsParallelToolCalls: boolean;
    supportsStructuredOutputs: boolean;
    supportsHostedTools:       boolean;
    refusalShape:              "block" | "finish_reason" | "none";
  };
}
```

This is *capability negotiation*: rather than hard-coding what each provider supports, the harness reads the metadata at boot (and on configuration reload) and routes/adapts accordingly. New provider capabilities arrive without code changes; missing capabilities surface as the router declining to route that request to that provider, rather than as a runtime failure deep in the loop.

The harness picks a provider via the model router (Ch.17 territory); the loop only sees the interface. When a provider fails, the router falls back to the next *compatible* one — same tool-schema dialect, at least the context window this turn needs, and reasoning and policy parity (Ch.02 covered this discipline in the loop's error-handling rules). A fallback that lacks the primary's capabilities is not a fallback; it is a different failure mode. Credential pools (rotating API keys on 429s) live in the router too — Hermes Agent and Paperclip both implement this.

### Configuration

The harness's configuration surface usually looks like this:

- **File.** YAML, JSON, or TOML; loaded once at startup. Hot-reload is optional and risky — it can break the cache by mutating tool descriptions mid-run (Ch.04).
- **Env-var overrides.** Every key can be overridden by an env var. Env wins over file. Use a documented, prefixed naming convention; random unprefixed env vars become debugging traps.
- **Secret references.** Sensitive values stored elsewhere — keychain, AWS Secrets Manager, encrypted file. Config holds `$secret:NAME` pointers resolved at runtime; secrets never appear in the loaded config object.
- **Schema validation.** Pydantic, zod, JSON Schema — pick one. Fail at startup on validation errors, surfacing *all* errors at once. The agent should not start if the config is invalid.
- **Plugin contributions.** Plugins can extend the schema with their own keys, merged at load time.

A common bug worth pre-empting: writing a config value to disk that contains a resolved secret. The serializer should re-emit `$secret:` references, never the resolved value. Test this with a unit check — serialize and grep for known secret material.

### Session, run, subagent — the vocabulary

Four work-unit terms recur across systems; pin their meanings to keep code and docs aligned:

- **Session** — a conversation thread with one participant on one channel in one workspace. Has a stable ID; persists transcript + state; can be resumed (Ch.08).
- **Run** — one invocation of the loop. Has a start, an end, a final state (succeeded / failed / cancelled). One session contains many runs over its lifetime.
- **Subagent** — a child run spawned by a parent (Ch.10). Sees a filtered slice of the parent's context; returns a single observation.
- **Heartbeat** — a wakeup tick used by control planes (Paperclip): the supervisor wakes up periodically and checks each session for work to do. A heartbeat may or may not result in a run.

OpenCode's `SessionID` and `RunID` branded types are the cleanest reference for keeping these straight; Paperclip's `issues` / `heartbeat_runs` / `agent_task_sessions` schema is the most thorough.

### Instance state and tenant scoping

A harness that serves more than one project, user, or tenant needs *instance state* — services scoped per project rather than globally. OpenCode's `InstanceState.make()` is the pattern: services are constructed lazily per `(project, agent)` combination and cached. Paperclip's multi-tenancy goes further — every table has a `company_id` and every query carries it.

The shape that scales: at the boundary of every harness operation, look up the instance for the current `(tenant, project, agent)` and route through it. Never reach into a global service from a request handler. The leak that comes back to bite you is one user seeing another user's memory because a global singleton was shared. Ch.06's namespace rule and Ch.08's tenant-scoped state machine both rely on this discipline.

### The bus and the streaming surface

Two adjacent concerns that production harnesses keep separate:

- The **internal event bus** lets plugins and observability subscribe to harness events (`session_started`, `tool_completed`, `run_failed`) without mutating shared state. Most harnesses run a simple in-process pub/sub; the bus is *not* durable by default — events that need to survive a restart get persisted separately (Ch.08).
- The **streaming surface** delivers tokens, tool events, and status updates to UIs (TUI, web, CLI). Server-sent events and WebSocket are both common. The harness fans bus events to connected clients filtered by session.

Keep the two separate. The bus is for in-process pub/sub; the streaming surface is the network face. Mixing them produces awkward coupling — every UI event becomes a global bus event, and the bus becomes a serialization point under load.

### Health and readiness

Two probes worth shipping from day one:

- **Liveness** — is the process alive at all? Cheap: a simple HTTP 200 with no dependencies.
- **Readiness** — is the harness ready to serve real traffic? Checks the DB, the model provider (with a tiny test call cached for a minute to avoid hammering it), plugin handshakes, and any critical hook errors at startup.

Three metrics that pay for themselves in the first month: number of active runs, queue depth, error rate per minute. These belong in Ch.16's trace pipeline but are worth wiring at the harness level from the start.

### Simpler harnesses age better

Anthropic's *Harness design for long-running agentic applications* essay names a useful rule: *every component in a harness encodes an assumption about what the model can't do on its own.* As models improve, those assumptions weaken. Components that earned their place last quarter may be unnecessary overhead this quarter.

Two practical consequences:

- **Audit your harness annually.** For each component, ask: *does the current model still need this?* Remove what no longer pays for itself. Anthropic notes they removed their "sprint" decomposition layer when a stronger model could handle longer coherent work without it.
- **Add complexity with the same discipline.** Each new harness component should solve a *measured* failure mode, not a theoretical one. Components added speculatively almost never come back out.

The goal is not the most sophisticated harness. It is the simplest harness that reliably handles your workload. The patterns in this chapter are a checklist of what is *available*, not a checklist of what must be *present*.

---

## Real-system notes

- **OpenCode** is the strongest end-to-end reference for an embedded harness: typed service composition with Effect Layers, a clean session/run separation, provider transport adapters per family, an SSE event bus, and a per-project `InstanceState` pattern. Read it as the "default" harness shape for a coding agent.
- **Hermes Agent** is the reference for harness + gateway separation: the inner agent loop is independent of the channel adapters (Telegram, CLI, cron), so the same harness serves many surfaces. The plugin hook surface (`pre_llm_call`, `post_tool_call`, and friends) is well-shaped and worth borrowing.
- **Paperclip** is the control-plane harness: it does not call models directly; it orchestrates *other* harnesses (adapter processes) through a heartbeat scheduler with explicit run-state machines, atomic claim, and reapers (Ch.08). Strongest reference for multi-tenant, multi-process production deployments.
- **OpenClaw** ships the cleanest channel-gateway abstraction over a personal-assistant harness — useful study for the gateway/harness boundary specifically.

A pointer outside the open-source repos: Anthropic's *"Harness design for long-running agentic applications"* (anthropic.com/engineering) is the best short read on context resets vs. compaction (Ch.05's territory), evaluator agents (Ch.10's verification pattern), and the principle that harness sophistication should track model capability.

---

## Common failure cases

*These failures are durable; their fixes evolve fastest — each names the pattern and leaves current specifics to you and your AI partner.*

- **Used before its dependency is ready.** The first request after a fresh boot fails, the second works, and you can never reproduce it locally. *Fix: an enforced readiness gate that refuses to flip until every declared dependency reports healthy — with the two-pass plugin shape, discover before you build registries, activate hooks after.*
- **Liveness and readiness are the same probe.** A slow dependency makes a healthy process look crashed, and the supervisor restart-loops the whole fleet. *Fix: liveness-depends-on-nothing, readiness-depends-on-everything, wired to different consumers.*
- **A gating hook fails open.** A blocked action goes through, with only a logged hook error nobody alarmed on. *Fix: fail-closed by required, explicit tag at registration — gating hooks must declare it, observational hooks opt into fail-open (Ch.12).*
- **One tenant sees another's session.** A global singleton slips into the request path and leaks state across tenants under load. *Fix: per-tenant instance resolution at every operation boundary with no global escape hatch (Ch.15).*
- **The harness keeps scaffolding the model outgrew.** A component still costs latency and tokens but no longer protects against anything. *Fix: a scheduled harness audit with evidence — tag each component with the measured failure it solves and A/B its removal against the eval suite (Ch.16).*

---

## Pair with your agent

A few prompts that work well on this chapter:

- *"Draw the component diagram of my current agent code. Identify which Ch.01–10 chapter each component implements, and flag anything that is implementing two chapters' worth of concerns inside one file."*
- *"Take my agent's startup code and reorder it into the bootstrap sequence from this chapter. Verify that health and readiness can fail independently — show me a failing readiness check that does not kill the process."*
- *"Wire the six lifecycle hooks (`pre_session`, `pre_llm_call`, `post_llm_call`, `pre_tool_call`, `post_tool_call`, `post_session`). Add a sample plugin that logs each event with timing. Verify the plugin can be added without modifying the loop."*
- *"Implement graceful shutdown: SIGINT triggers drain mode, in-flight runs get up to 60 seconds to finish, anything still running gets marked cancelled in the run state machine (Ch.08). Verify with a deliberately stuck run."*
- *"Refactor my provider integration into a `ModelProvider` interface with one adapter per family. Confirm the loop now compiles against a mock provider that has no network access. Use the mock for unit tests."*
- *"Audit my harness against Anthropic's rule: 'every component encodes an assumption about what the model can't do.' For each component, name the assumption. Propose one component to remove or simplify based on what the current frontier model can do reliably."*
- *"Add tenant scoping: every service that touches state takes a tenant context. Write a test that proves a request for tenant A cannot reach tenant B's session, memory, or run state."*
- *"Set up the harness's event bus and an SSE streaming endpoint that listens to it. Show me a session whose tokens stream live to a browser while plugins simultaneously subscribe to the same events on the bus."*

---

## What's next

You now have the architecture, the lifecycle, and the extension surface. The remaining chapters add the layers that production agents need to ship: human-in-the-loop approvals (Ch.12), connectors and MCP (Ch.13), skills and subagent design as a unit (Ch.14), backend infrastructure (Ch.15), observability (Ch.16), cost and latency strategy (Ch.17), safety and adversarial inputs (Ch.18), and operations (Ch.19). Each is a component or concern that bolts onto the harness shape you now have.

Ch.12 is next: the gate that pauses the loop and asks a human before taking high-risk actions.

---

<!-- nav-footer -->
<div align="center">

[⬅️ Prev: Ch.10 Multi-agent delegation](10-multi-agent-delegation.md) · [📖 Course index](../README.md) · [Next: Ch.12 Human in the loop ➡️](12-human-in-the-loop.md)

</div>
