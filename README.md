# Neuro-Symbolic AI

A working neuro-symbolic prototype combining neural parsing with symbolic reasoning for CPU-native, explainable decision-making.

## Architecture

```
Text Input → Neural Parser → Symbolic IR (DSL) → Rule Engine → Decision + Trace
```

- **Neural Parser** — Converts natural language to structured DSL
- **Symbolic Executor** — CPU-native rule evaluation with full reasoning traces
- **Hybrid Fallback** — Neural reasoning when symbolic rules are insufficient

## Project Structure

See [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md) for the full directory layout.

## Setup

```bash
# Install core dependencies
uv sync

# Install dev tools
uv sync --extra dev

# Install neural components (requires PyTorch)
uv sync --extra neural

# Install everything
uv sync --all-extras
```

## Usage

```bash
# Run tests
uv run pytest

# Run example
uv run python examples/rollout_decision.py
```

## "Isn't This Just Agentic AI?"

No. Agents and neuro-symbolic systems look similar on the surface (LLM + tools), but
they differ in **who controls the reasoning**.

| Dimension        | Agentic AI              | Neuro-Symbolic (this project)  |
|------------------|-------------------------|--------------------------------|
| Who reasons      | The LLM                 | The symbolic engine            |
| LLM role         | Orchestrator + reasoner | Parser / translator only       |
| Determinism      | No — probabilistic      | Yes — same input, same output  |
| Explainability   | Post-hoc rationalization| Actual computation trace       |
| LLM calls/decision | 3-5+                 | 0-1                           |
| Reasoning latency| Seconds                 | < 1ms (CPU)                    |
| Best for         | Open-ended, creative    | Rule-based, auditable decisions|

**Agents** are best when the reasoning itself is the hard part and you need a system
that can improvise. **Neuro-symbolic** is best when rules are known and decisions
must be deterministic, auditable, and fast.

Full analysis: [docs/neuro_symbolic_vs_agentic.md](docs/neuro_symbolic_vs_agentic.md)

## 4-Week Plan

See [PLAN.md](PLAN.md) for the detailed implementation timeline.

## Playground

6 toy applications that demonstrate neuro-symbolic patterns mapped to real-world
domains (finance, healthcare, compliance, and more).

See [docs/PRD_PLAYGROUND.md](docs/PRD_PLAYGROUND.md) for the full PRD.

## Core Insight

Neural networks excel at perception and parsing. Symbolic systems excel at reasoning and deterministic execution. Third-Wave AI combines both — using neural models for grounding and symbolic engines for CPU-efficient, explainable inference.
