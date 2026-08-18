# The Missing Layer Beneath Intelligence and Agents: What is an Intelligence Agent Organization Runtime (IAOR)?

**Author**: W.Y.
**Date**: 2026-08-18

---

> I have been building a runtime that organizes multiple independent agents into a team — IAOR (Intelligence Agent Organization Runtime). After it first truly ran a coordinated experiment, I decided to write it down clearly. This article defines it; the ones that follow are things I kept finding as I went through the data.

## 1. A Definition: Intelligence and Agents Are Two Layers

Let me begin with a definition.

The same large language model, combined with different system prompts, different tool sets, and different ways of assembling and invoking those tools, becomes a completely different independent agent. The same model behaves like a rigorous engineer in company A's agent, a casual assistant in company B's agent, and something else again in your own agent.

Based on this observation, I distinguish two layers:

```
Intelligence — the capability of the model itself. Essential, reusable, shareable.
     ↓ "代理" (agent-ified) as
Agent — a carrier that agent-ifies intelligence into a concrete role.
    Binding a specific model, plus a specific prompt, a tool set, and a
    specific way of assembling and invoking those tools.
```

One **Intelligence** can be agent-ified into multiple **Agents** in different ways. Each Agent inherits the capability of the same intelligence, yet exhibits different behaviors, language, and abilities. **An Agent is never isolated — behind it always sits a shared layer of Intelligence.**

This phenomenon is not something I invented. It exists objectively. Every company building agents — including myself — practices it: the same model, agent-ified into different agents by different prompts, different tool sets, and different assembly methods.

The reason I separate "Intelligence" and "Agent" as two names is that they refer to different things in IAOR, as explained below.

## 2. What is IAOR

Given that "Intelligence" and "Agent" are two layers, a more precise question emerges:

**When organizing a group of agents into a collaborative team, what is actually being organized?**

The answer is not a pile of isolated agents. What is organized is **agents that have agent-ified a particular model** — those agents behind which a shared Intelligence sits. The runtime that organizes them is:

**Intelligence Agent Organization Runtime (IAOR)**

- **Intelligence** — the essence; the capability of the model; reusable and shareable
- **Agent** — the carrier that agent-ifies intelligence into a concrete role
- **Organization** — organizing multiple agents that have agent-ified intelligence
- **Runtime** — the infrastructure that carries this and can truly run

I place "Intelligence" first not for rhetorical effect, but because this layering is the starting point for understanding the entire category: **what you organize is not black-box agents, but agents that have agent-ified intelligence.**

## 3. The Layer I See as Blank: Beneath Agents

Looking at the current state of the technology stack as I understand it, I believe there is a gap here.

```
┌───────────────────────────────────────────────┐
│  Application Layer                             │
│  Codex, QoderWake, 杏林泛舟, Agent 365, etc.   │
├───────────────────────────────────────────────┤
│  ── the layer I believe is missing: IAOR ──   │
├───────────────────────────────────────────────┤
│  Orchestration Framework Layer                 │
│  CrewAI, LangGraph, AutoGen                    │
├───────────────────────────────────────────────┤
│  Model Routing Layer                           │
│  Fugu, LiteLLM, Portkey                        │
├───────────────────────────────────────────────┤
│  Operating System Layer                        │
│  ANOLISA, Step AOS, Linux, macOS               │
└───────────────────────────────────────────────┘
```

In 2026, there are two mainstream forces in this industry, but in my view, neither has clearly defined the "organization" layer:

**One force is interconnection standards.** In May 2026, China released the "Artificial Intelligence - Agent Interconnection" national standard series (GB/Z 185), with an open-sourced AIP (Agent Interconnection Protocol) and issued agent identity codes. It solves the **interface layer** problem — how agents from different vendors recognize each other, how they "speak," and how they interoperate under a unified protocol. It defines the "traffic rules" of communication.

**The other force is industry applications.** Some vendors build vertical deployments on top of standards — for example, 杏林泛舟 AI 2.0 in the life sciences domain, which splits compliance checks, pharmacovigilance, etc. into multiple business agents interconnected by standard.

But neither force, in my view, directly answers the more essential question: **When a group of agents is to form a truly collaborative organization, who defines their structure, roles, communication, governance, and memory?**

- Interconnection standards govern "how agents recognize each other and speak" — they do not define how agents are organized into teams.
- Industry applications govern "how a specific domain uses multi-agent" — they do not provide a general organization runtime.
- Orchestration frameworks (CrewAI, LangGraph) govern "how the flow of a one-off task is orchestrated" — they treat agents as pipeline nodes, not as organization members.

**What I believe has not been clearly defined is the "organization" layer.**

An IAOR does not care what model your agent uses, nor what task it executes. It cares about:

- **Structure** — how agents are organized into a team, what the hierarchy is, how communication links run
- **Coordination** — how tasks are decomposed, dispatched, and tracked
- **Governance** — what each agent can and cannot do, where the boundaries are
- **Memory** — what the organization as a whole knows, not lost across agent lifecycles
- **Resilience** — how the organization responds when APIs fail, rate limits trigger, or networks drop
- **Domain agnosticism** — the core engine is domain-neutral; domain knowledge is mounted on demand; the same engine can drive an engineering team, a legal research group, or a content creation team

Think of it as Kubernetes for agents. Kubernetes does not care what application you run — it cares about scheduling, scaling, networking, and fault tolerance. IAOR does not care what task your agent is doing — it cares about how they are organized, how they communicate, and how they recover from failure.

## 4. Architecture: Three Roles, Four Systems

The architecture is the result of repeated trial and error. It is still evolving, but the core structure has stabilized enough to share.

The roles are shells — they define responsibilities and permissions, not capabilities. Capability comes from the domain knowledge mounted onto a role, the available tool sets, and the bound model.

### Three Roles

**Director** — the top-level manager. The Director stands outside the organization, holds a global view, and interacts with the user through natural language. It monitors organizational state, reports anomalies, analyzes problems, and executes user decisions. It can create or destroy Managers/Operators, adjust policies, and intervene when necessary. *(Currently, the Director has joined the architecture design; the full three-layer collaboration chain is under development.)*

**Manager** — the coordinator. Receives high-level tasks, decomposes them into subtasks, creates Operators with the appropriate capabilities, dispatches work, tracks progress, and stores results into the Archive. A Manager can recursively manage other Managers.

**Operator** — the executor. Each Operator has a clear capability boundary (tool set, read/write permissions, model binding) and autonomously decides how to complete assigned tasks. It understands intent and decides its own execution path.

### Four Systems

**Policy** — the governance layer. Defines under what conditions each Operator may do what; dynamically overridable, escalating to Director approval.

**Channel** — the communication infrastructure. Operators and Managers communicate through the Channel rather than direct function calls, supporting message routing, broadcasting, and audit logs.

**Archive** — the organizational memory. Every task result, every decision, every error is stored in the Archive, forming a queryable structured knowledge base.

**Selector** — the model selection policy. Evaluates tasks based on capability needs, latency sensitivity, cost constraints, and historical success rates, selecting the optimal model and self-optimizing.

## 5. Proof It Runs: A Real Experiment

I have implemented and run a series of experiments on a self-developed engine.

### The Core Experiment

A **1 Manager + 2 Operators** team implemented a TaskFlow kanban application from a product brief. It was a real development task with real build requirements.

**One premise matters.** The three members — the Manager and the two Operators — each hold their own independent LLM API key. This is not a single large model spawning parallel sub-agents internally. These are three genuinely independent members, each with its own model connection, organized into one team by the runtime.

What happened, in 6 minutes and 26 seconds:

| Metric | Result |
|--------|--------|
| LLM calls — Manager | 25 |
| LLM calls — Operator 003 (frontend) | 23 |
| LLM calls — Operator 004 (infrastructure) | 18 |
| **Total LLM calls** | **66** |
| Files produced | 25 source files (5 infrastructure + 20 frontend) |
| Execution mode | Two operators working in parallel |
| File conflicts | 0 |
| `npm run build` | Passed, 53 modules, zero errors |
| Manual intervention | 0 |

66 calls for 25 files, zero conflicts. Whether that is a lot or a little, I leave to you. Along the way, the two Operators also coordinated directly — at one point, the infrastructure Operator handed its complete interface documentation to the frontend Operator without routing through the Manager. Why this organization is worth even a few extra calls — and what that direct coordination means — is exactly what the next articles unpack.

<figure>
  <img src="article-images/fig1-planning.png" alt="Manager planning 25 files and dispatching two operators in parallel">
  <figcaption><b>Figure 1 — Planning and dispatch.</b> The Manager reads the scaffold and splits the task into 25 files — 5 infrastructure files to Operator 004, 20 frontend files to Operator 003 — judging that the two can run in parallel. Notice the links: the purple line from the Manager to 004 has just turned from dashed to solid, a glowing pulse travelling from the Manager toward 004's panel — the first dispatch message in flight — while the pale-blue line between 004 and 003 marks a peer link still waiting. Inside the panels, 004 is already ordering the 5 infrastructure files by dependency; 003 is digesting the frontend spec.</figcaption>
</figure>

<figure>
  <img src="article-images/fig2-complete.png" alt="Project COMPLETE — both operators idle, build passed">
  <figcaption><b>Figure 2 — Delivery.</b> The Manager marks the project <code>COMPLETE</code>: both Operators idle, 25 source files merged with zero conflicts, <code>npm run build</code> passing 53 modules with zero TypeScript errors. The dispatch pulse is over — the active links fall back to dashed — and only the pale-blue peer line between 004 and 003 remains, a trace of the cross-member coordination along the way.</figcaption>
</figure>

<figure>
  <img src="article-images/fig3-product.png" alt="The TaskFlow kanban app running in the browser">
  <figcaption><b>Figure 3 — The product.</b> What those 25 files compile into is a real kanban app running at localhost:5173 — task creation, filtering, search, stats, and a three-column status board. It is the final landing point of this coordination: a usable product, not just a COMPLETE badge on a management panel.</figcaption>
</figure>

## Conclusion

This article makes a single point: beneath Intelligence and Agents, an "organization" layer is missing.

A single Agent, however capable, is still one person. But when you want to organize a group of genuinely independent agents — each with its own model connection — into a team that actually collaborates, who defines their structure, roles, communication, governance, and memory? As far as I can tell, no clear answer exists yet. IAOR is my answer to that missing layer.

And it is not a concept — it runs. Three members, each holding an independent model connection, were organized into one team by a runtime: 6 minutes 26 seconds, 25 files, zero conflicts. Why this organization is worth it, and what it can do that no single agent can, is what the coming articles answer.

---

*W.Y. · 2026-08-18*
