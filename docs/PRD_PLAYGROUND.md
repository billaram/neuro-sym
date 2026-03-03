# PRD: Neuro-Symbolic Playground

Author: Ramkumar Mohan
Status: Draft
Last Updated: 2026-03-03

---

## 1. Overview

### What

An interactive, CLI-driven playground that ships **6 toy neuro-symbolic applications**.
Each toy is small enough to understand in one sitting but demonstrates a real
architectural pattern. Every toy maps to a documented real-world application domain
with references to production systems and research.

### Why

- The neuro-sym core engine (parser + executor) needs concrete use cases to validate
- Toy apps make the abstract architecture tangible and demoable
- Mapping to real-world applications builds intuition for where neuro-symbolic
  approaches outperform pure-neural or pure-symbolic alternatives
- Serves as a portfolio piece: "here are 6 problems, here is why hybrid wins"

### Who

- Primary: The author — learning by building
- Secondary: Anyone exploring the repo — each toy is self-contained and runnable

---

## 2. Design Principles

1. **Each toy is self-contained** — runs with a single command, no external services
2. **Each toy uses the shared engine** — parser, DSL, executor from `src/`
3. **Each toy has a real-world mapping** — documented with references and trade-off
   analysis
4. **Neural and symbolic boundaries are visible** — logs show exactly which component
   handled which part of the pipeline
5. **Everything runs on CPU** — no GPU required for any toy

---

## 3. Toy Applications

### Toy 1 — Feature Rollout Advisor

**Domain:** Product engineering / feature flagging

**Scenario:**
A product team asks: "Should we roll out Smart Checkout to all users?" The system
takes natural language input, extracts constraints, evaluates them against live
metrics, and returns a decision with a full reasoning trace.

**Example interaction:**
```
$ uv run python -m playground.rollout_advisor

> Should we roll out Smart Checkout if latency < 3s and conversion lift > 5%?

Parsing...
  Parser: rule-based
  Intent: rollout_decision
  Constraints: {latency_ms: <3000, min_lift: >0.05}

Executing...
  Rule: latency_check    → PASS (actual: 2100ms < 3000ms)
  Rule: lift_check       → PASS (actual: 7.2% > 5.0%)
  Aggregation: AND(PASS, PASS) → PASS

Decision: ROLL_OUT
Confidence: deterministic
Trace: 2 rules evaluated, 0 failures, 3 operations, 0.4ms
```

**Neuro-symbolic split:**
| Stage        | Component | Why                                         |
|--------------|-----------|---------------------------------------------|
| Parse query  | Neural    | Natural language is unstructured             |
| Extract constraints | Neural/Rule | Map phrases to typed values          |
| Evaluate rules | Symbolic | Deterministic comparison, must be auditable |
| Aggregate    | Symbolic  | Boolean logic, must be deterministic         |

**Real-world mapping:**

| Production System         | What It Does                                    | Neuro-Symbolic Angle                          |
|---------------------------|-------------------------------------------------|-----------------------------------------------|
| LaunchDarkly              | Feature flags with targeting rules              | Rules are symbolic; user segments could be neural-classified |
| Statsig                   | Statistical gate checks for rollouts            | Constraint evaluation is symbolic; metric interpretation could use neural grounding |
| Internal rollout systems  | Multi-metric go/no-go decisions                 | Exactly this toy — parse intent, evaluate rules, explain decision |

**References:**
- Johari et al. — "Controlled Feature Rollouts in Large-Scale Systems" (Meta, 2020)
- Statsig documentation on sequential testing and gate rules

---

### Toy 2 — Loan Eligibility Engine

**Domain:** Financial services / credit decisioning

**Scenario:**
A loan officer asks: "Is this applicant eligible for a home loan?" The system takes
applicant data (possibly as natural language narrative), extracts financial attributes,
evaluates them against regulatory and business rules, and returns an eligibility
decision with compliance trace.

**Example interaction:**
```
$ uv run python -m playground.loan_engine

> 35 year old applicant, income 120k, credit score 740, debt-to-income 28%,
  requesting 400k home loan

Parsing...
  Parser: rule-based
  Intent: loan_eligibility
  Attributes: {age: 35, income: 120000, credit_score: 740, dti: 0.28, loan_amount: 400000}

Executing...
  Rule: min_credit_score  → PASS (740 >= 620)
  Rule: max_dti           → PASS (28% <= 43%)
  Rule: income_ratio      → PASS (loan/income = 3.33 <= 4.5)
  Rule: min_age           → PASS (35 >= 18)

Decision: ELIGIBLE
Confidence: deterministic
Compliance trace: 4 regulatory rules evaluated, all passed
```

**Neuro-symbolic split:**
| Stage            | Component | Why                                        |
|------------------|-----------|--------------------------------------------|
| Parse narrative  | Neural    | Free-text applicant descriptions            |
| Extract attributes | Neural  | NER-like extraction of financial values     |
| Evaluate rules   | Symbolic  | Regulatory rules must be deterministic      |
| Generate trace   | Symbolic  | Compliance requires auditable decision path |

**Real-world mapping:**

| Production System      | What It Does                                     | Neuro-Symbolic Angle                              |
|------------------------|--------------------------------------------------|----------------------------------------------------|
| Zest AI                | ML-based credit underwriting                     | Neural scoring + symbolic compliance rules          |
| Fannie Mae DU          | Desktop Underwriter for mortgage eligibility      | Rule-based core with statistical risk layering      |
| Upstart                | AI lending with explainability requirements       | Neural risk model + symbolic regulatory compliance  |

**References:**
- CFPB — Adverse Action Notice requirements (explainability mandate)
- Basel III — Credit risk standardized approach (rule-based capital requirements)
- Rudin, Cynthia — "Stop Explaining Black Box Models for High Stakes Decisions" (2019)

---

### Toy 3 — Medical Triage Assistant

**Domain:** Healthcare / clinical decision support

**Scenario:**
A nurse describes patient symptoms in plain language. The system extracts symptom
entities, evaluates them against triage protocols, and recommends an urgency level
with a full clinical reasoning trace.

**Example interaction:**
```
$ uv run python -m playground.triage

> 45 year old male, chest pain radiating to left arm, shortness of breath,
  onset 30 minutes ago, history of hypertension

Parsing...
  Parser: neural
  Intent: symptom_triage
  Symptoms: [chest_pain_radiating, dyspnea, acute_onset]
  History: [hypertension]
  Demographics: {age: 45, sex: male}

Executing...
  Rule: cardiac_red_flag   → TRIGGERED (chest_pain_radiating + dyspnea)
  Rule: time_urgency       → TRIGGERED (onset < 60min)
  Rule: risk_factors       → PRESENT (male, age>40, hypertension)
  Aggregation: ANY_TRIGGERED → escalate

Decision: TRIAGE_LEVEL_1 (Immediate)
Confidence: deterministic
Trace: cardiac protocol triggered, 3 red flags, recommend immediate evaluation
```

**Neuro-symbolic split:**
| Stage             | Component | Why                                         |
|-------------------|-----------|---------------------------------------------|
| Parse symptoms    | Neural    | Clinical language is varied and informal     |
| Entity extraction | Neural    | Map free text to standard symptom codes      |
| Triage rules      | Symbolic  | Protocols are deterministic and life-critical |
| Urgency scoring   | Symbolic  | Must follow established clinical algorithms   |

**Real-world mapping:**

| Production System      | What It Does                                   | Neuro-Symbolic Angle                              |
|------------------------|------------------------------------------------|---------------------------------------------------|
| Isabel Healthcare      | Diagnosis decision support                     | NLP symptom parsing + rule-based differential      |
| Epic Sepsis Model      | Sepsis early warning                           | Neural risk scoring + symbolic alert thresholds    |
| Babylon Health         | Symptom checker with triage                    | NLU + probabilistic reasoning + rule guardrails    |
| ESI (Emergency Severity Index) | Standard 5-level triage protocol       | Pure symbolic — candidate for neural front-end     |

**References:**
- Gilboy et al. — "Emergency Severity Index (ESI): A Triage Tool" (AHRQ)
- Topol, Eric — "High-Performance Medicine: The Convergence of Human and AI" (2019)
- WHO — "Integrated Management of Childhood Illness" (rule-based clinical protocol)

---

### Toy 4 — Policy Compliance Checker

**Domain:** Legal / regulatory compliance

**Scenario:**
A compliance officer asks: "Does this transaction comply with our AML policy?" The
system parses the transaction description, extracts relevant attributes, evaluates
them against policy rules, and returns a compliance verdict with a regulation-mapped
audit trail.

**Example interaction:**
```
$ uv run python -m playground.compliance

> Wire transfer of $45,000 from a new account opened 3 days ago to an
  international recipient in a high-risk jurisdiction

Parsing...
  Parser: neural
  Intent: aml_compliance_check
  Attributes: {type: wire_transfer, amount: 45000, account_age_days: 3,
               recipient: international, jurisdiction_risk: high}

Executing...
  Rule: ctr_threshold      → PASS (45000 < 10000 threshold? NO — flag for review)
  Rule: new_account_flag   → TRIGGERED (account_age < 30 days)
  Rule: high_risk_jurisdiction → TRIGGERED
  Rule: structuring_check  → PASS (single transaction, no split pattern)
  Aggregation: 2 of 4 rules triggered → escalate

Decision: ESCALATE_TO_SAR_REVIEW
Confidence: deterministic
Audit trail: 4 AML rules evaluated, 2 triggered, BSA/AML regulation references attached
```

**Neuro-symbolic split:**
| Stage              | Component | Why                                         |
|--------------------|-----------|---------------------------------------------|
| Parse description  | Neural    | Transaction narratives are unstructured      |
| Extract attributes | Neural    | Map text to regulatory-relevant fields       |
| Policy evaluation  | Symbolic  | Regulations are explicit rules — no ambiguity |
| Audit trail        | Symbolic  | Regulators require deterministic, traceable decisions |

**Real-world mapping:**

| Production System      | What It Does                                   | Neuro-Symbolic Angle                                |
|------------------------|------------------------------------------------|------------------------------------------------------|
| Chainalysis            | Blockchain transaction monitoring              | Neural pattern detection + symbolic rule thresholds   |
| Featurespace (ARIC)    | Real-time fraud and AML detection              | Adaptive neural models + deterministic policy rules   |
| ComplyAdvantage        | AML screening and monitoring                   | NLP entity resolution + rule-based screening lists    |

**References:**
- FinCEN — Bank Secrecy Act / AML regulations
- FATF — Risk-Based Approach guidance (2014)
- EU 6th Anti-Money Laundering Directive (6AMLD)

---

### Toy 5 — Recipe Substitution Advisor

**Domain:** Consumer / knowledge reasoning

**Scenario:**
A home cook asks: "I'm making chocolate cake but I'm out of eggs and I'm lactose
intolerant — what can I substitute?" The system parses the query, identifies
constraints (dietary restrictions, missing ingredients), reasons over substitution
rules, and returns options with explanations.

**Example interaction:**
```
$ uv run python -m playground.recipe

> Making chocolate cake, no eggs available, lactose intolerant, want to keep it moist

Parsing...
  Parser: neural
  Intent: ingredient_substitution
  Recipe: chocolate_cake
  Missing: [eggs]
  Dietary: [lactose_free]
  Preferences: [moist_texture]

Executing...
  Rule: egg_substitute         → OPTIONS [applesauce, flax_egg, banana]
  Rule: lactose_filter         → REMOVE [butter → coconut_oil, milk → oat_milk]
  Rule: texture_preference     → RANK [applesauce > banana > flax_egg] (moisture)
  Aggregation: combine filtered + ranked options

Decision: SUBSTITUTE
  eggs      → applesauce (1/4 cup per egg) — adds moisture, mild sweetness
  butter    → coconut oil (1:1 ratio)
  milk      → oat milk (1:1 ratio)
Confidence: deterministic (substitution rules) + heuristic (texture ranking)
Trace: 3 substitution rules, 1 dietary filter, 1 preference ranking
```

**Neuro-symbolic split:**
| Stage               | Component | Why                                        |
|---------------------|-----------|--------------------------------------------|
| Parse query         | Neural    | Casual cooking language is highly varied     |
| Identify constraints| Neural    | Dietary needs expressed in many ways         |
| Substitution rules  | Symbolic  | Known 1:1 mappings, chemistry-based ratios   |
| Dietary filtering   | Symbolic  | Binary include/exclude against ingredient DB |
| Texture ranking     | Hybrid    | Heuristic rules informed by culinary knowledge |

**Real-world mapping:**

| Production System   | What It Does                                    | Neuro-Symbolic Angle                               |
|---------------------|-------------------------------------------------|----------------------------------------------------|
| Whisk (Samsung)     | Smart recipe management with substitutions      | NLP parsing + rule-based substitution engine         |
| Yummly              | Personalized recipe recommendations             | Neural taste profiling + symbolic dietary filters    |
| IBM Chef Watson     | Computational creativity in cooking             | Neural flavor pairing + symbolic recipe structure    |

**References:**
- Ahn et al. — "Flavor Network and the Principles of Food Pairing" (Scientific Reports, 2011)
- Marin et al. — "Recipe1M+: A Dataset for Learning Cross-Modal Embeddings for Cooking Recipes" (2019)

---

### Toy 6 — Puzzle Solver (Logic Grid)

**Domain:** Education / constraint satisfaction

**Scenario:**
A user describes a logic grid puzzle in natural language. The system parses the clues
into formal constraints, solves the constraint satisfaction problem symbolically, and
returns the solution with a step-by-step deduction trace.

**Example interaction:**
```
$ uv run python -m playground.puzzle

> There are 3 people: Alice, Bob, Carol.
  They each have a different pet: cat, dog, fish.
  Alice doesn't have a cat. Bob doesn't have a fish. Carol has a dog.

Parsing...
  Parser: neural
  Intent: logic_grid_puzzle
  Entities: {people: [alice, bob, carol], pets: [cat, dog, fish]}
  Constraints:
    - NOT(alice, cat)
    - NOT(bob, fish)
    - IS(carol, dog)

Executing...
  Step 1: IS(carol, dog)         → assign carol=dog, eliminate dog from others
  Step 2: NOT(alice, cat)        → alice ∈ {fish}  (dog eliminated by step 1)
  Step 3: alice=fish             → assign, eliminate fish from others
  Step 4: NOT(bob, fish)         → already satisfied
  Step 5: bob ∈ {cat}           → assign bob=cat

Decision: SOLVED
  Alice → Fish
  Bob   → Cat
  Carol → Dog
Confidence: deterministic (unique solution found)
Trace: 5 deduction steps, 3 assignments, 0 backtrack
```

**Neuro-symbolic split:**
| Stage            | Component | Why                                          |
|------------------|-----------|----------------------------------------------|
| Parse clues      | Neural    | Natural language clues have varied phrasing    |
| Formalize constraints | Neural/Rule | Map "doesn't have" → NOT(), "has" → IS() |
| Solve CSP        | Symbolic  | Constraint propagation is pure logic          |
| Generate trace   | Symbolic  | Each deduction step is explicit               |

**Real-world mapping:**

| Production System / Domain | What It Does                              | Neuro-Symbolic Angle                              |
|----------------------------|-------------------------------------------|----------------------------------------------------|
| Google OR-Tools            | Constraint programming solver              | Pure symbolic — candidate for neural front-end      |
| Automated scheduling       | Staff/room/resource allocation             | NLP requirement parsing + symbolic CSP solving      |
| Hardware verification      | Formal verification of chip designs        | Specification parsing + symbolic model checking     |
| Sudoku / crossword solvers | Puzzle solving with constraint propagation | Well-studied CSP domain                            |

**References:**
- Russell & Norvig — "AI: A Modern Approach" Ch. 6 (Constraint Satisfaction Problems)
- Rossi, van Beek, Walsh — "Handbook of Constraint Programming" (2006)

---

## 4. Architecture — How Toys Share the Core Engine

Each toy is a thin configuration layer over the shared `src/` engine:

```
playground/
├── __init__.py
├── rollout_advisor.py      ← Toy 1
├── loan_engine.py          ← Toy 2
├── triage.py               ← Toy 3
├── compliance.py           ← Toy 4
├── recipe.py               ← Toy 5
├── puzzle.py               ← Toy 6
└── common/
    ├── __init__.py
    ├── cli.py              ← Shared CLI formatting & colors
    └── runner.py           ← Shared pipeline invocation

configs/
├── toys/
│   ├── rollout_advisor.yaml
│   ├── loan_engine.yaml
│   ├── triage.yaml
│   ├── compliance.yaml
│   ├── recipe.yaml
│   └── puzzle.yaml
```

Each toy YAML defines:
```yaml
name: rollout_advisor
description: Feature rollout go/no-go decisions

parser:
  primary: rule          # rule | neural
  fallback: neural       # used if primary fails

dsl:
  intent: rollout_decision
  schema: rollout_decision.schema.yaml

executor:
  rules: rollout_rules.yaml
  trace: true
  mode: deterministic    # deterministic | probabilistic | hybrid

display:
  show_parse: true
  show_trace: true
  show_timing: true
```

**Data flow (same for all toys):**
```
User text
  → src/parser/     (configured per toy: rule or neural)
  → src/dsl/        (validated against toy-specific schema)
  → src/executor/   (evaluated against toy-specific rules)
  → playground CLI   (formatted output with trace)
```

---

## 5. Documentation Strategy — Linking Toys to Real World

Each toy ships with a companion doc in `docs/playground/`:

```
docs/playground/
├── overview.md                  ← This playground explained
├── toy1_rollout_advisor.md      ← Deep dive + real-world mapping
├── toy2_loan_engine.md
├── toy3_triage.md
├── toy4_compliance.md
├── toy5_recipe.md
├── toy6_puzzle.md
└── real_world_patterns.md       ← Cross-cutting pattern analysis
```

Each toy doc follows this structure:

1. **Problem statement** — What does this toy solve?
2. **Why neuro-symbolic?** — Why can't pure neural or pure symbolic handle this alone?
3. **Architecture walkthrough** — Which component does what, with diagrams
4. **Real-world systems** — 2-3 production systems that solve similar problems,
   with analysis of their architecture
5. **Trade-off analysis** — When would you use neural-only, symbolic-only, or hybrid?
6. **References** — Papers, docs, regulatory sources

### `real_world_patterns.md` — Cross-Cutting Analysis

This document maps the patterns seen across all 6 toys:

| Pattern                          | Toys Using It    | Real-World Domain               |
|----------------------------------|------------------|----------------------------------|
| NL → structured constraint       | 1, 2, 3, 4, 5, 6 | Any system with user-facing NLU |
| Deterministic rule evaluation    | 1, 2, 3, 4       | Compliance, safety, finance     |
| Constraint satisfaction          | 5, 6              | Scheduling, planning, logistics |
| Audit trail / explainability     | 1, 2, 3, 4       | Regulated industries            |
| Dietary/preference filtering     | 5                 | Recommendation systems          |
| Neural fallback on rule miss     | 1, 2, 3          | Hybrid production systems       |

---

## 6. Implementation Phases

### Phase 1 — Foundation (aligns with Week 2-3)
- [ ] Create `playground/` directory and `common/` utilities
- [ ] Implement **Toy 1 (Rollout Advisor)** — simplest, validates the pipeline
- [ ] Implement **Toy 6 (Puzzle Solver)** — pure symbolic, validates executor depth
- [ ] Write companion docs for both

### Phase 2 — Breadth (aligns with Week 3-4)
- [ ] Implement **Toy 2 (Loan Engine)** — adds regulatory rule complexity
- [ ] Implement **Toy 4 (Compliance Checker)** — adds multi-rule escalation logic
- [ ] Write companion docs for both

### Phase 3 — Neural Integration (aligns with Week 4)
- [ ] Implement **Toy 3 (Triage)** — requires neural parsing of clinical language
- [ ] Implement **Toy 5 (Recipe)** — requires hybrid ranking (rules + heuristics)
- [ ] Write companion docs for both
- [ ] Write `real_world_patterns.md` cross-cutting analysis

### Phase 4 — Polish
- [ ] Add `--verbose` and `--quiet` flags to all toys
- [ ] Add `--json` output mode for programmatic use
- [ ] Add timing benchmarks to each toy
- [ ] Ensure all toys work with `uv run python -m playground.<toy>`

---

## 7. Success Criteria

| Criteria                                          | Target                    |
|---------------------------------------------------|---------------------------|
| All 6 toys runnable with a single command          | Yes                       |
| Each toy produces a reasoning trace                | Yes                       |
| Each toy has a companion doc with real-world mapping | Yes                     |
| No toy requires GPU                                | Yes                       |
| Average toy execution time (CPU)                   | < 50ms (symbolic path)    |
| Cross-cutting pattern doc complete                 | Yes                       |
| Each toy uses the shared `src/` engine             | Yes                       |

---

## 8. Non-Goals

- **Production-grade NLP** — toy parsers can be simple regex or single-prompt LLM calls
- **Real data** — all toys use synthetic/hardcoded sample data
- **Web UI** — CLI only (stretch: simple Streamlit for demo day)
- **Multi-turn conversation** — single query → single decision per invocation
- **Model training** — no fine-tuning; use pretrained models or rule-based parsing only

---

## 9. Reference Summary

### Papers
| Paper | Relevant Toys | Key Idea |
|-------|---------------|----------|
| Wei et al. — Chain-of-Thought (2022) | All | Externalizing reasoning steps improves accuracy |
| Reed & de Freitas — NPI (2016) | 6 | Neural models can learn to compose programs |
| Garcez et al. — Neural-Symbolic Survey (2017) | All | Taxonomy of hybrid integration approaches |
| Rudin — Stop Explaining Black Boxes (2019) | 2, 3, 4 | Interpretable models over post-hoc explanations |
| Russell & Norvig — AIMA Ch. 6 | 6 | Constraint satisfaction problem formalization |

### Regulatory / Standards
| Source | Relevant Toys |
|--------|---------------|
| CFPB Adverse Action Notices | 2 |
| Basel III Credit Risk | 2 |
| FinCEN BSA/AML | 4 |
| FATF Risk-Based Approach | 4 |
| AHRQ Emergency Severity Index | 3 |
| WHO IMCI Protocol | 3 |
