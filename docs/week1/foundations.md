# Week 1 — Foundations: Third-Wave AI & Neuro-Symbolic Thinking

Author: Ramkumar Mohan

---

## 1. The Three Waves of AI

The DARPA framework, presented by John Launchbury (Director, DARPA I2O) in 2017,
divides the history and future of AI into three waves. Each wave is characterized by
its capabilities across four axes: **perceive**, **learn**, **abstract**, and **reason**.

### Wave 1 — Handcrafted Knowledge

**Era:** 1960s–1990s (expert systems, symbolic AI)

Human experts encode domain knowledge as explicit rules. The system performs logical
deduction over those rules.

| Capability | Level |
|------------|-------|
| Perceive   | None  |
| Learn      | None  |
| Abstract   | None  |
| Reason     | Strong |

**Strengths:**
- Excellent at taking concrete facts and working through logical implications
- Deterministic, auditable, explainable
- CPU-efficient — rule evaluation is cheap

**Weaknesses:**
- Cannot perceive the natural world (no vision, no language understanding)
- No learning — rules must be hand-authored
- Brittle — breaks on any input outside its rule set
- Cannot transfer knowledge across domains

**Examples:** Chess programs, TurboTax, medical expert systems, DARPA Cyber Grand Challenge

### Wave 2 — Statistical Learning

**Era:** 2010s–present (deep learning, Transformers)

Systems learn patterns from large datasets. No explicit rules — the model discovers
structure through gradient descent over millions of parameters.

| Capability | Level   |
|------------|---------|
| Perceive   | Strong  |
| Learn      | Strong  |
| Abstract   | Minimal |
| Reason     | Minimal |

**Strengths:**
- Excellent at perceiving the natural world (vision, speech, language)
- Can learn nuanced classification and prediction from data
- Scales with compute and data

**Weaknesses:**
- Minimal reasoning ability — cannot reliably chain logical steps
- No contextual capability — cannot explain *why* it made a decision
- Requires massive training datasets
- Vulnerable to adversarial examples (bizarre, confident errors)
- Statistically impressive in aggregate but individually unreliable

**Examples:** ImageNet classifiers, GPT-family models, speech recognition, autonomous
vehicle perception, DARPA electromagnetic spectrum sharing

### Wave 3 — Contextual Adaptation

**Era:** Emerging (neuro-symbolic, hybrid architectures)

Systems build explanatory underlying models of the world. They combine the perception
and learning of Wave 2 with the reasoning and abstraction of Wave 1. They can learn
from minimal examples and explain their decisions.

| Capability | Level  |
|------------|--------|
| Perceive   | Strong |
| Learn      | Strong |
| Abstract   | Strong |
| Reason     | Strong |

**Characteristics:**
- Constructs contextual models, not just pattern matches
- Combines learning with reasoning
- Can explain decisions in human-interpretable terms
- Requires far less training data (one-shot or few-shot)
- Interprets novel situations by reasoning over learned models

**This is where neuro-symbolic AI lives.** The neural component handles perception
and grounding. The symbolic component handles reasoning and execution. Together they
cover all four capability axes.

### Wave Comparison Summary

```
              Perceive    Learn    Abstract    Reason
Wave 1            -         -         -          +++
Wave 2          +++       +++         -           -
Wave 3          +++       +++       +++         +++
```

---

## 2. Why Pure Transformers Struggle with Reasoning

Transformers (GPT, BERT, etc.) are Wave 2 systems. They are remarkably capable at
perception and pattern extraction, but they have fundamental limitations when it comes
to multi-step reasoning.

### The Core Problem

Transformers compute via dense matrix multiplications over fixed-depth attention layers.
Every input, regardless of complexity, passes through the same number of computational
steps. There is no mechanism for:

- **Variable-depth computation** — a 2-step problem and a 20-step problem get the same
  number of forward-pass layers
- **Explicit working memory** — intermediate results are spread across attention heads,
  not stored in addressable locations
- **Verifiable execution** — you cannot inspect which "rule" the model applied at step 7
  of its reasoning, because there are no discrete steps

### Evidence from Research

**Chain-of-Thought Prompting (Wei et al., 2022)** demonstrated that prompting large
language models to "show their work" with intermediate reasoning steps dramatically
improves performance on arithmetic, commonsense, and symbolic reasoning tasks. A
540B-parameter model with just 8 chain-of-thought exemplars achieved state-of-the-art
on the GSM8K math benchmark, surpassing even fine-tuned GPT-3 with a verifier.

This is revealing in two ways:

1. **Positive:** Transformers have latent reasoning capacity that can be elicited
   through prompting — the knowledge is there, encoded in weights.
2. **Negative:** Without explicit prompting to externalize intermediate steps, the model
   fails. It cannot reliably perform multi-step reasoning internally. The "chain of
   thought" is a workaround for the absence of native step-by-step execution.

### Specific Failure Modes

| Task Type                   | Transformer Behavior                              |
|-----------------------------|---------------------------------------------------|
| Multi-step arithmetic       | Errors compound — no carry propagation mechanism   |
| Constraint satisfaction     | Cannot systematically check all constraints         |
| Logical deduction (A→B→C)   | Prone to shortcut reasoning, skipping steps        |
| Negation and counting       | Unreliable on simple cases                         |
| Deterministic execution     | Same input can produce different outputs            |

### The Fundamental Mismatch

Reasoning requires **sequential, state-dependent computation** where:
- Each step depends on the result of the previous step
- State is updated explicitly between steps
- The number of steps varies with problem complexity
- Each step is inspectable and verifiable

Transformers provide **parallel, fixed-depth computation** where:
- All positions are processed simultaneously
- Depth is fixed at architecture time
- Internal state is distributed and opaque
- Outputs are probabilistic, not deterministic

This is not a training data problem or a scale problem. It is an architectural mismatch
between the computation model (dense, parallel, fixed-depth) and the task structure
(sparse, sequential, variable-depth).

---

## 3. Why Symbolic Systems Are CPU-Efficient

Symbolic execution — rule evaluation, constraint checking, logical deduction — is
inherently efficient on CPUs because the computation is sparse, structured, and
deterministic.

### Properties of Symbolic Execution

| Property              | Description                                              |
|-----------------------|----------------------------------------------------------|
| **Sparse**            | Only relevant rules fire; irrelevant rules are skipped   |
| **Deterministic**     | Same input always produces same output                   |
| **Bounded**           | Rule count is known; worst-case cost is predictable      |
| **Sequential**        | Steps execute in order; easy to parallelize per-rule     |
| **Inspectable**       | Every step produces an auditable trace                   |
| **No GPU required**   | Integer/boolean logic — no matrix multiplication needed  |

### Compute Comparison

Consider evaluating: "Should we roll out if latency < 3s AND lift > 5%?"

**Symbolic approach (CPU):**
```
1. Load constraints: {latency_ms: 3000, min_lift: 0.05}
2. Check: actual_latency < 3000?  → True   (1 comparison)
3. Check: actual_lift > 0.05?     → True   (1 comparison)
4. AND(True, True)                → True   (1 boolean op)
5. Decision: ROLL_OUT
Total: ~3 operations, <1ms, zero GPU memory
```

**Transformer approach (GPU):**
```
1. Tokenize input (~50 tokens)
2. Forward pass through N attention layers
3. Each layer: QKV projection, attention, FFN
4. ~100M+ floating-point operations
5. Decode output tokens
6. Parse output text to extract decision
Total: ~100ms, requires GPU, non-deterministic
```

For deterministic decision logic, symbolic execution is 100-1000x faster and uses a
fraction of the memory.

### When Symbolic Wins

- The rules are known and finite
- The decision must be deterministic and auditable
- Latency matters (real-time systems)
- Compute budget is constrained (edge, mobile, CPU-only servers)
- Explainability is required (compliance, safety-critical systems)

### When Symbolic Loses

- The input is unstructured (natural language, images, audio)
- The rules are unknown and must be learned from data
- The domain is too complex for manual rule authoring
- Fuzzy / probabilistic reasoning is needed

This is precisely why the hybrid approach matters: use neural models where symbolic
systems are weak (perception, parsing), and symbolic systems where neural models are
weak (reasoning, execution).

---

## 4. The Case for Hybrid: Neural Perception + Symbolic Execution

### The Insight

Neural networks and symbolic systems have complementary strengths:

```
               Neural          Symbolic
Perception      +++               -
Learning        +++               -
Reasoning        -              +++
Execution        -              +++
Explainability   -              +++
CPU Efficiency   -              +++
```

A hybrid system uses each where it is strongest:

1. **Neural models** handle the messy, unstructured front-end: understanding natural
   language, extracting intent, grounding ambiguous terms
2. **Symbolic systems** handle the precise, deterministic back-end: evaluating rules,
   checking constraints, producing auditable decisions

### Supporting Evidence from the Literature

**Neural Programmer-Interpreters (Reed & de Freitas, 2016)** demonstrated that neural
networks can learn to compose and execute programs by combining:
- A **task-agnostic recurrent core** for general processing
- A **persistent key-value program memory** for storing learned subroutines
- **Domain-specific encoders** for perceiving different environments

NPI learned addition, sorting, and 3D canonicalization with a single model managing 21
subprograms. The key insight: rather than asking a neural network to *be* the executor,
train it to *invoke* the right subroutine and let structured execution handle the rest.
NPI also uses external tools (scratch pads with read-write pointers) to cache
intermediate results — reducing the burden on recurrent hidden state.

**Neural-Symbolic Learning and Reasoning (Garcez et al., 2017)** surveyed the
integration of connectionist and symbolic approaches, identifying applications in
computational biology, fault diagnosis, simulator training, and software verification.
The survey synthesizes views from 14 researchers and establishes that the combination
of neural learning with symbolic reasoning is not merely additive — it produces
capabilities that neither approach achieves alone.

**Chain-of-Thought Prompting (Wei et al., 2022)** showed that externalizing reasoning
steps improves Transformer performance. This is indirect evidence for the neuro-symbolic
thesis: when you force a neural model to produce symbolic-like intermediate
representations (step-by-step reasoning traces), performance improves. The chain of
thought *is* a rudimentary symbolic intermediate representation.

### The Architecture We Are Building

```
┌─────────────────────────────────────────────────────────────────┐
│                     NEURO-SYMBOLIC PIPELINE                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────┐  │
│  │              │    │              │    │                  │  │
│  │   NEURAL     │───▶│  SYMBOLIC    │───▶│    SYMBOLIC      │  │
│  │   PARSER     │    │  IR (DSL)    │    │    EXECUTOR      │  │
│  │              │    │              │    │                  │  │
│  └──────────────┘    └──────────────┘    └──────────────────┘  │
│        │                    │                     │             │
│   Text Input          Structured             Decision +        │
│   (natural            Program                Reasoning         │
│    language)          (JSON/AST)             Trace             │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  NEURAL FALLBACK (only when symbolic rules insufficient) │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  Neural components: GPU optional, used for parsing/grounding   │
│  Symbolic components: CPU-native, deterministic, explainable   │
└─────────────────────────────────────────────────────────────────┘
```

**Stage 1 — Neural Parser** (Week 2)
- Input: Natural language text
- Output: Structured DSL (JSON)
- Method: Rule-based parser + LLM-based parser (configurable)
- Handles ambiguity, varied phrasing, intent extraction

**Stage 2 — Symbolic IR** (Week 2)
- The DSL serves as the intermediate representation
- Schema-validated, deterministic, inspectable
- Decouples parsing from execution

**Stage 3 — Symbolic Executor** (Week 3)
- Input: Validated DSL program
- Output: Decision + reasoning trace
- Method: Rule engine, constraint solver, trace logger
- CPU-native, deterministic, explainable

**Stage 4 — Neural Fallback** (Week 4)
- Activated only when the symbolic executor has no matching rules
- Uses a small Transformer (DistilBERT) for probabilistic reasoning
- Decision is flagged as `probabilistic` rather than `deterministic`

---

## 5. Key Takeaways

1. **Wave 1 reasoned but couldn't perceive.** Wave 2 perceives but can't reason.
   Wave 3 aims to do both.

2. **Transformers are not executors.** They are powerful pattern matchers that can
   approximate reasoning through prompting tricks, but they lack the architectural
   primitives for reliable, deterministic, multi-step logic.

3. **Symbolic execution is cheap.** Rule evaluation on CPU is orders of magnitude
   faster than neural inference for deterministic decision tasks.

4. **The hybrid is the point.** Use neural models to bridge the gap between
   unstructured input and structured representation. Use symbolic systems to bridge
   the gap between structured representation and reliable output.

5. **Explainability is not optional.** In production systems — compliance, safety,
   business decisions — you need to know *why* a decision was made, not just *what*
   the decision was. Symbolic reasoning traces provide this natively.

---

## 6. Reading Sources

| #  | Source | Link |
|----|--------|------|
| 1  | Launchbury — "A DARPA Perspective on AI" (2017) | [Summary & slides](https://machinelearning.technicacuriosa.com/2017/03/19/a-darpa-perspective-on-artificial-intelligence/) |
| 1a | Brian Hu — Three Waves breakdown | [Blog post](https://brianhhu.github.io/2017-05-03-three-waves-ai/) |
| 1b | Sheth — Duality of Data and Knowledge Across the Three Waves (2021) | [arXiv](https://arxiv.org/abs/2103.13520) |
| 2  | Garcez et al. — Neural-Symbolic Learning and Reasoning (2017) | [arXiv](https://arxiv.org/abs/1711.03902) |
| 3  | Reed & de Freitas — Neural Programmer-Interpreters (ICLR 2016) | [arXiv](https://arxiv.org/abs/1511.06279) |
| 4  | Wei et al. — Chain-of-Thought Prompting (NeurIPS 2022) | [arXiv](https://arxiv.org/abs/2201.11903) |

---

## Next Steps

- **Week 2:** Design the DSL and build the parser (both rule-based and neural)
- **Week 3:** Build the symbolic executor (rule engine + constraint solver + trace logger)
- **Week 4:** Integrate everything and add neural fallback
