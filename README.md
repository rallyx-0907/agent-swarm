# agent-swarm

A portal where a small team works with its agents.

Every agent the team needs lives behind one front door. You open the portal,
see who is there, and talk to them. Chat is the surface; the actual work —
repos, pipelines, deploys, documents, decisions — happens behind it.

The premise: a small team does great work not by having *many* agents but by
having *well-harnessed* ones. Agents do the execution; a few very strong
people do the direction, judgment, and escalation. The scarce resource stops
being headcount and becomes *AI engineering skill* — how well the swarm is
designed, fed, bounded, and measured.

## What an agent is here

An agent is not a chat wrapper. It is a durable operating unit:

| Capability | What it means concretely |
|---|---|
| Memory | Durable per-agent memory + shared org knowledge, with write policy, decay, and conflict resolution when two agents disagree |
| Tools | A governed tool registry — every tool declares its blast radius and who may call it |
| Model routing | Per-task tier selection under a budget ceiling; escalate the model on failure, not by default |
| Repo genesis | An agent can scaffold, name, and initialize a new repository with CI and conventions already correct |
| Product incubation | Take an idea from brief → spec → repo → running product, with human gates at the decision points |
| Agent genesis | An agent can bring a new agent into existence *(shape still open — see Open questions)* |

Every one of these is reached through the portal — it is the entry point to the
swarm, not one feature inside it.

## Architecture

Orchestration runs on **LangGraph** — a stateful graph runtime, not a prompt
chain. Graph state is checkpointed, so runs are resumable, inspectable, and
interruptible at a human gate.

```mermaid
flowchart TB
    subgraph surface["The portal"]
        roster["Agent roster<br/>who exists, what each is for"]
        chat["Threads<br/>humans + agents, one context"]
        queue["Approvals + in-flight runs"]
    end

    subgraph core["Orchestration — LangGraph"]
        router["Intent router"]
        graph["Agent graphs<br/>stateful, checkpointed, resumable"]
        gate{{"Human gate<br/>irreversible actions"}}
    end

    subgraph substrate["Substrate"]
        mem[("Memory<br/>per-agent + org knowledge graph")]
        tools["Tool registry<br/>blast radius declared per tool"]
        models["Model router<br/>tier selection + budget ceiling"]
    end

    subgraph control["Control plane"]
        evals["Evals<br/>per-role regression"]
        traces["Traces<br/>across agent handoffs"]
        authz["Authorization<br/>who may do what, irreversibly"]
    end

    roster --> chat --> router --> graph
    graph --> gate --> queue
    queue -->|approved| graph
    graph <--> mem
    graph <--> tools
    graph <--> models
    graph --> traces
    tools --> authz
    evals -.judges.-> graph
    traces -.feeds.-> evals
```

The control plane is not optional garnish. Once agents create agents, it is the
only thing that tells you whether the swarm is getting better or quietly worse.

## Core disciplines

The organization's real capital. Everything above is an expression of these.

**Foundations**
- **Context engineering** — what enters the window, in what order, and what is deliberately left out
- **RAG** — retrieval that is grounded, current, and attributable
- **Harness engineering** — the loop around the model: tools, permissions, feedback, recovery
- **Graph engineering** — knowledge and workflow as graphs, not as flat text
- **Loop engineering** — when an agent iterates, when it stops, when it escalates

**Equally load-bearing, easier to skip**
- **Evaluation engineering** — per-role regression evals. Without them, "agents spawning agents" has no error signal at all.
- **Memory engineering** — distinct from RAG. Retrieval is reading; this is deciding what to write, what to forget, and who wins a contradiction.
- **Authorization & blast-radius design** — tiering actions by reversibility. Agents reaching prod, spend, and outbound comms need a gate that is designed, not incidental.
- **Observability & trace engineering** — a swarm cannot be debugged from logs; you need one trace per run, unbroken across handoffs.
- **Escalation design** — engineered paths to the humans. Otherwise "a few strong people" become a bottleneck queue.
- **Cost & model-routing policy** — the discipline behind model autonomy: routing by task class and confidence, under a hard ceiling.

## Scope

v1 targets **full organizational operations** — dev work, internal ops, and
outbound/production actions. That ambition is exactly why authorization,
evals, and traces are in the architecture from the start rather than bolted on
after the first expensive mistake. It also makes the approvals queue core
portal UI rather than an admin afterthought — it is where those human gates
actually get answered.

## Open questions

- **Agent genesis** — does creating an agent produce a durable, committed, versioned agent definition, or an ephemeral worker for one task? A promotion path between the two is plausible, but it depends on evals being good enough to judge what deserves promotion.
- **Memory topology** — per-agent stores, a shared vector index, a knowledge graph, or a layered combination.
- **Portal reach** — is the portal the only surface, or do Slack and other chat clients federate into it?
- **Human gate granularity** — per action, per budget threshold, or per agent trust tier.

---

*Draft. Nothing here is built yet.*
