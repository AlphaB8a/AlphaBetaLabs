# Genesis Stack Topology (Locked)

```txt
User / Genesis CLI
        |
        v
┌──────────────────────────────────────────┐
│ Overseer (Cognitive Authority)           │
│ Provider priority:                       │
│   1) Google services (Vertex AI / Gemini)│
│   2) OpenAI                              │
│   3) NVIDIA                              │
│ Responsibilities: plans / critiques / QA │
└───────────────┬──────────────────────────┘
                |
                v
┌──────────────────────────────────────────┐
│ Guardrails (Structural Authority)        │
│ - deterministic policy checks first      │
│ - optional NeMo Guardrails when enabled  │
│ - halts on policy violations             │
└───────────────┬──────────────────────────┘
                |
                v
┌──────────────────────────────────────────┐
│ Local Workers (Execution)                │
│ - qwen3-coder                            │
│ - deepseek-r1                            │
│ - wizard-math                            │
│ - qwen3-vl                               │
└──────────────────────────────────────────┘
```

| Layer      | Responsibility      | Failure Mode Prevented            |
| ---------- | ------------------- | --------------------------------- |
| Genesis    | State + dispatch    | Tool confusion                    |
| Overseer   | Judgment            | Bad plans / weak critiques        |
| Guardrails | Enforcement         | Runaway agents / policy bypass    |
| Workers    | Execution           | Slow / unsafe / low-quality work  |

## File References

- Entry point: `Agent_Core/genesis.py`
- Preflight: `Agent_Core/preflight.py`
- Engineer (aider wrapper): `Agent_Core/nodes/engineer/engineer_cli.py`
- Engineer (aider config): `Agent_Core/nodes/engineer/.aider.conf.yml`
- Orchestration (graph wiring): `Agent_Core/orchestration/graph.py`
- Orchestration (routing/supervisor): `Agent_Core/orchestration/supervisor.py`
- NeMo guardrails (protocols): `Agent_Core/orchestration/guardrails/config.co`
- NeMo guardrails (model binding): `Agent_Core/orchestration/guardrails/config.yml`
- NeMo guardrails (prompt templates): `Agent_Core/orchestration/guardrails/prompts.yml`
- NeMo guardrails (actions): `Agent_Core/orchestration/guardrails/actions.py`
- OpenAI overseer helper: `Agent_Core/orchestration/overseer.py`
- Orchestration (state): `Agent_Core/orchestration/state.py`
- Legacy orchestration (checkpointing hooks): `Agent_Core/python/orchestration/graph.py`
- Legacy supervisor (retry logic): `Agent_Core/python/orchestration/supervisor.py`
- Training runner node: `Agent_Core/python/nodes/trainer.py`
- Training job (long-running): `Agent_Core/python/training/nightly_evolution.py`
- Auditor hook helper: `Agent_Core/hooks/git_auditor.py`
- Git post-commit hook: `Workspace/active_projects/.git/hooks/post-commit`

## Concrete Places To Insert NeMo (Practical)

### 1️⃣ Agent Orchestration Layer

Use NeMo to:

- Define agent graphs
- Specify allowed transitions
- Prevent “agent drift”

This replaces ad-hoc agent loops.

Concrete insertion points:

- `Agent_Core/orchestration/graph.py`
- `Agent_Core/orchestration/supervisor.py`
- `Agent_Core/python/orchestration/graph.py`
- `Agent_Core/python/orchestration/supervisor.py`

### 2️⃣ Execution Supervisor

NeMo watches:

- Which tool is called
- In what order
- With what constraints

If something violates policy → halt.

Concrete insertion points:

- `Agent_Core/orchestration/supervisor.py`
- `Agent_Core/genesis.py`

### 3️⃣ Long-Running Tasks

Training jobs, data pipelines, batch analysis:

- NeMo handles retries
- NeMo enforces checkpoints
- NeMo guarantees reproducibility

Overseer should never be responsible for this.

Concrete insertion points:

- `Agent_Core/python/nodes/trainer.py`
- `Agent_Core/python/training/nightly_evolution.py`
- `Agent_Core/python/orchestration/graph.py`

## When NOT To Use NeMo (Important)

❌ Single-step tasks  
❌ Interactive coding (Aider already handles this)  
❌ Early prototyping  
❌ Anything you might change hourly  

NeMo shines when the workflow is settled.

## A Simple Decision Rule (Keep This)

If the problem is “What should we do?” → Overseer (Google → OpenAI → NVIDIA).  
If the problem is “Did we follow the rules?” → Guardrails (deterministic checks; NeMo optional).

## Personal Reference

Flow: `Genesis → Overseer → Guardrails → Workers`

Recommendation (clear):

- Keep Google services as primary overseer (Vertex AI / Gemini)
- Enable NeMo guardrails only when workflows stabilize and reproducibility matters

You’re early-right, not late.
