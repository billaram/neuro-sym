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

## 4-Week Plan

See [PLAN.md](PLAN.md) for the detailed implementation timeline.

## Core Insight

Neural networks excel at perception and parsing. Symbolic systems excel at reasoning and deterministic execution. Third-Wave AI combines both — using neural models for grounding and symbolic engines for CPU-efficient, explainable inference.
