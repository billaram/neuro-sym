# Neuro-Symbolic AI — Project Structure

```
neuro-sym/
│
├── src/                            # Core source code
│   ├── dsl/                        # Domain-Specific Language definition
│   │   ├── __init__.py
│   │   ├── schema.py               # DSL grammar & schema definition
│   │   ├── tokens.py               # Token types for the DSL
│   │   └── validators.py           # DSL validation logic
│   │
│   ├── parser/                     # Neural + rule-based parsing
│   │   ├── __init__.py
│   │   ├── text_to_dsl.py          # Text → DSL conversion
│   │   ├── rule_parser.py          # Rule-based parser (regex/grammar)
│   │   ├── neural_parser.py        # LLM/Transformer-based parser
│   │   └── ast_builder.py          # AST construction from parsed output
│   │
│   ├── executor/                   # Symbolic reasoning engine (CPU-native)
│   │   ├── __init__.py
│   │   ├── rule_engine.py          # Rule evaluation engine
│   │   ├── constraint_solver.py    # Constraint satisfaction logic
│   │   ├── trace_logger.py         # Execution trace & explanation graph
│   │   └── decision_output.py      # Deterministic output generator
│   │
│   ├── grounding/                  # Neural grounding (Week 4)
│   │   ├── __init__.py
│   │   ├── embedder.py             # DistilBERT / small Transformer embeddings
│   │   ├── classifier.py           # Intent classification
│   │   └── fallback.py             # Neural fallback when rules fail
│   │
│   ├── pipeline/                   # End-to-end integration
│   │   ├── __init__.py
│   │   ├── orchestrator.py         # Parse → Execute → Output pipeline
│   │   ├── config.py               # Pipeline configuration
│   │   └── explain.py              # Explanation output (CLI / web)
│   │
│   └── utils/                      # Shared utilities
│       ├── __init__.py
│       ├── logging.py              # Structured logging
│       ├── timing.py               # CPU/latency benchmarking helpers
│       └── io.py                   # File I/O helpers
│
├── tests/                          # Test suite
│   ├── parser/
│   │   ├── test_text_to_dsl.py
│   │   ├── test_rule_parser.py
│   │   └── test_ast_builder.py
│   ├── executor/
│   │   ├── test_rule_engine.py
│   │   ├── test_constraint_solver.py
│   │   └── test_trace_logger.py
│   ├── grounding/
│   │   ├── test_embedder.py
│   │   └── test_fallback.py
│   ├── pipeline/
│   │   └── test_orchestrator.py
│   └── integration/
│       └── test_end_to_end.py
│
├── docs/                           # Weekly documentation & architecture
│   ├── week1/
│   │   └── foundations.md           # Third-Wave AI summary & notes
│   ├── week2/
│   │   └── parser_design.md        # DSL spec & parser architecture
│   ├── week3/
│   │   └── executor_design.md      # Rule engine & reasoning design
│   ├── week4/
│   │   └── integration_notes.md    # Hybrid system & benchmarks
│   └── architecture/
│       ├── system_overview.md       # High-level architecture document
│       ├── dsl_specification.md     # Formal DSL grammar & examples
│       └── diagrams/               # Architecture diagrams (PNG/SVG)
│           └── .gitkeep
│
├── research/                       # Research materials & reading notes
│   ├── papers/                     # Downloaded PDFs organized by topic
│   │   ├── week1/                  # Third-Wave, neural-symbolic foundations
│   │   │   └── .gitkeep
│   │   ├── week2/                  # Semantic parsing, program synthesis
│   │   │   └── .gitkeep
│   │   ├── week3/                  # ILP, conditional compute, retrieval
│   │   │   └── .gitkeep
│   │   └── week4/                  # NPI, hybrid architectures
│   │       └── .gitkeep
│   ├── notes/                      # Reading notes & annotations
│   │   ├── darpa_third_wave.md
│   │   ├── neural_symbolic_survey.md
│   │   ├── semantic_parsing.md
│   │   ├── chain_of_thought.md
│   │   └── program_synthesis.md
│   └── references/
│       └── bibliography.md         # Full reference list with links
│
├── benchmarks/                     # Performance measurement
│   ├── scripts/
│   │   ├── bench_parser.py         # Parser latency benchmarks
│   │   ├── bench_executor.py       # Executor CPU benchmarks
│   │   └── bench_pipeline.py       # End-to-end pipeline benchmarks
│   └── results/
│       └── .gitkeep                # Benchmark result CSVs / JSON
│
├── examples/                       # Usage examples & demos
│   ├── rollout_decision.py         # Smart Checkout rollout example
│   ├── sample_inputs.json          # Example text inputs
│   └── sample_outputs.json         # Expected DSL + decision outputs
│
├── configs/                        # Configuration files
│   ├── dsl_schema.yaml             # DSL schema definition
│   ├── rules.yaml                  # Rule definitions for executor
│   └── pipeline.yaml               # Pipeline runtime config
│
├── PLAN.md                         # 4-week implementation plan (detailed)
├── PROJECT_STRUCTURE.md            # This file
├── README.md                       # Project overview & setup instructions
├── pyproject.toml                  # Python project config
└── .gitignore                      # Git ignore rules
```

## Directory Purpose Summary

| Directory       | Purpose                                          | Active In  |
|-----------------|--------------------------------------------------|------------|
| `src/dsl/`      | DSL grammar, schema, validation                  | Week 2     |
| `src/parser/`   | Text-to-DSL conversion (rule + neural)           | Week 2     |
| `src/executor/` | Symbolic rule engine, constraints, traces         | Week 3     |
| `src/grounding/`| Neural embeddings & fallback reasoning            | Week 4     |
| `src/pipeline/` | End-to-end orchestration                          | Week 4     |
| `docs/`         | Weekly write-ups & architecture docs              | All weeks  |
| `research/`     | Papers, reading notes, bibliography               | All weeks  |
| `benchmarks/`   | CPU performance measurement scripts & results     | Week 3-4   |
| `tests/`        | Unit, integration, and end-to-end tests           | All weeks  |
| `examples/`     | Runnable demos and sample I/O                     | Week 2+    |
| `configs/`      | YAML configs for DSL, rules, pipeline             | Week 2+    |
