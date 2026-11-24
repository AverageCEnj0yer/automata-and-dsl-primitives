# Automata & Formal Grammars

Core formal language theory primitives in C++17 implementing general formal grammars, Chomsky hierarchy classification, nondeterministic/deterministic finite automata (NFA/DFA), bidirectional grammar-automaton conversion, and subset construction with epsilon-closure.

## What's Implemented

### `Grammar` (`include/grammar.h`, `include/grammar.cpp`)
Represents a formal grammar `G = (V_T, V_N, S, P)` with arbitrary symbol sequences on both sides of each production (`LHS` and `RHS` as `std::vector<Symbol>`), supporting grammars across all levels of the Chomsky hierarchy.

- **`toFiniteAutomaton()`**: Converts a right-linear regular grammar into an equivalent NFA by introducing an accepting state `X` (`A -> aB` becomes `δ(A, a) ∋ B`; `A -> a` becomes `δ(A, a) ∋ X`; `A -> ε` becomes `δ(A, "") ∋ X`). The resulting automaton is lazily cached.
- **`generateWord()`**: Samples valid strings from the language by performing a bounded random walk (up to 20 steps) over the derived finite automaton using `std::mt19937`.
- **`classifyGrammar()`**: Classifies the grammar according to the Chomsky hierarchy:
  - **Type 3 (Regular)** — single nonterminal LHS, strictly right-linear or strictly left-linear (no mixed linearity or epsilon rules).
  - **Type 2 (Context-Free)** — single nonterminal LHS, unrestricted RHS.
  - **Type 1 (Context-Sensitive)** — at least one nonterminal on the LHS and `|RHS| >= |LHS|`, permitting `S -> ε` only if `S` does not appear on any RHS.
  - **Type 0 (Recursively Enumerable)** — unrestricted fallback.
- **`print()`**: Displays terminals, nonterminals, start symbol, and formatted production rules.

### `FiniteAutomaton` (`include/finiteAutomaton.h`, `include/finiteAutomaton.cpp`)
Represents a finite automaton `(Q, Σ, δ, q0, F)` backed by a transition map `std::map<std::pair<Symbol, Symbol>, std::set<Symbol>>`, supporting nondeterministic branching and epsilon transitions (`""`).

- **`stringBelongsToLanguage(std::string_view)`**: Simulates NFA execution over single-character input symbols by propagating active state sets and accepting if any final state is active upon exhaustion.
- **`isDeterministic()`**: Inspects the transition table to determine whether every `(state, symbol)` pair has at most one target state.
- **`toDFA()`**: Implements **subset (powerset) construction** with stack-based **ε-closure**, converting any NFA or ε-NFA into an equivalent DFA with sequentially named states (`Q0`, `Q1`, ...).
- **`toGrammar()`**: Reconstructs an equivalent right-linear regular grammar from the automaton's transition table, stripping the synthetic accepting state `X`.
- **`print()`**: Displays states, alphabet, initial/final states, and transitions.

## Structure

```
automata-and-grammars/
├── CMakeLists.txt
├── main.cpp                   # Demo driver & assertion-based membership tests
└── include/
    ├── grammar.h / .cpp       # Formal grammar model, classifier, word generator
    └── finiteAutomaton.h / .cpp # NFA/DFA model, simulator, subset construction
```

## Build & Run

Requires a C++17 compiler and CMake ≥ 3.10. Out-of-source builds are enforced by `CMakeLists.txt`:

```bash
mkdir -p build && cd build
cmake ..
cmake --build .
./test
```

`main.cpp` defines a sample regular grammar, prints the grammar and its derived automaton, generates 5 random words, and runs 18 `assert`-based membership checks (9 accepted, 9 rejected). Commented hooks at the bottom of `main.cpp` also exercise `classifyGrammar()`, `toGrammar()`, `isDeterministic()`, and `toDFA()`.
