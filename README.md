# Automata Theory Primitives & Language Front-Ends (C++17)

A collection of from-scratch, zero-dependency C++17 implementations of **formal language theory primitives**, **grammar transformations**, and **recursive-descent front-ends**—built as foundational building blocks toward a full NFA-based **regular expression engine** and a custom **domain-specific language (DSL)**.

## Overview

Each module in this repository isolates a core theoretical or compiler-engineering layer so it can be built, tested, and reasoned about independently:

```
Formal Language Primitives                Language Processing Front-Ends
──────────────────────────                ──────────────────────────────
automata-and-grammars/                    regex-generator/
  • Formal Grammars & Chomsky Classifier    • Regex Lexer & Precedence Parser
  • NFA/DFA & Subset Construction (ε-NFA)   • Polymorphic Regex AST & Sampler
  • Grammar ↔ Finite Automaton Conversion
                                          dsl-frontend/
cnf-normalizer/                             • Typed Imperative DSL Lexer
  • 6-Stage CFG → CNF Pipeline              • LL(3) Recursive-Descent Parser
  • Strict CNF Structural Validator         • RAII Polymorphic AST (16 node types)
```

## Modules

| Module | Summary | Key Algorithms & Patterns | Build |
| :--- | :--- | :--- | :--- |
| [`automata-and-grammars/`](./automata-and-grammars/README.md) | Formal grammar & finite automaton primitives | Chomsky hierarchy classification (Types 0–3), NFA simulation, subset construction with ε-closure, Grammar ↔ FA conversion | CMake (C++17) |
| [`cnf-normalizer/`](./cnf-normalizer/README.md) | Context-Free Grammar to Chomsky Normal Form converter | Fixed-point nullable analysis, BFS unit-production closure, terminal isolation, binarization, dead-symbol pruning, CNF validator | CMake (C++17) |
| [`regex-generator/`](./regex-generator/README.md) | Regular expression parser, AST, and random string sampler | Lexer, 4-level recursive-descent parser, polymorphic AST (`Literal`, `Sequence`, `Alternation`, `Repeat`), bounded random generation | CMake (C++17) |
| [`dsl-frontend/`](./dsl-frontend/README.md) | Compiler front-end for a typed imperative mini-language | 36-token scanner, LL(3) lookahead recursive-descent parser, 9-level operator precedence (right-assoc `=` and `^`), `std::unique_ptr` AST | `g++` / `clang++` (C++17) |

## Engineering Highlights

- **Pure C++17 Standard Library** — zero third-party dependencies; relies directly on STL containers, `std::unique_ptr`, `std::string_view`, and `<random>`.
- **Core CS Algorithms From Scratch** — NFA/ε-NFA to DFA subset construction, Chomsky hierarchy classification, full 6-pass CNF normalization with cycle-safe BFS closure, and LL(3) recursive-descent parsing.
- **Self-Verifying Entry Points** — assertion-based test suites validate NFA membership (`automata-and-grammars`) and CNF invariants across multiple grammar families (`cnf-normalizer`), while `regex-generator` and `dsl-frontend` include executable test harnesses and sample programs.

## Quick Start

### Prerequisites
- A C++17-compliant compiler (`g++`, `clang++`, or MSVC)
- CMake ≥ 3.10 (for the three CMake-configured modules)

### Building & Running Each Module

**1. Automata & Formal Grammars**
```bash
cd automata-and-grammars
mkdir -p build && cd build
cmake .. && cmake --build .
./test
```

**2. CNF Grammar Normalizer**
```bash
cd cnf-normalizer
mkdir -p build && cd build
cmake .. && cmake --build .
./test
```

**3. Regex AST & String Generator**
```bash
cd regex-generator
mkdir -p build && cd build
cmake .. && cmake --build .
./test
```

**4. DSL Compiler Front-End**
```bash
cd dsl-frontend
g++ -std=c++17 main.cpp lexer.cpp parser.cpp -o dsl_parser
./dsl_parser sample.txt
```

## Roadmap

These standalone primitives are designed to converge into two larger systems:

1. **Full Regex Engine** — lower the AST produced by `regex-generator/` into Thompson NFAs using `automata-and-grammars/`, enabling both NFA simulation and DFA compilation via subset construction.
2. **Complete DSL Toolchain** — extend `dsl-frontend/` with a semantic analysis pass (symbol tables and static type checking), control-flow constructs (`if` / `while`), and a tree-walk interpreter or bytecode backend, plus a CYK parser atop `cnf-normalizer/`.
