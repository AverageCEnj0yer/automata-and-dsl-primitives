# CNF Grammar Normalizer

A C++17 implementation of a multi-stage transformation pipeline that converts arbitrary context-free grammars (CFGs) into **Chomsky Normal Form (CNF)**, accompanied by a strict CNF structural validator.

CNF constrains every production to one of two binary/terminal shapes (`A -> BC` or `A -> a`, plus `S -> ε` only if the start symbol never appears on a right-hand side), which serves as the prerequisite normalization pass for algorithms such as CYK parsing.

## Transformation Pipeline

`normalize(const Grammar&)` in `src/grammar.cpp` runs an ordered sequence of pure transformation passes where each stage preserves the language while establishing invariants required by the next:

1. **Start Symbol Isolation**: Introduces a collision-free fresh start symbol (`S0`, generated via `freshSymbol`) with `S0 -> S` so the start symbol never appears on any RHS.
2. **ε-Production Elimination (`removeEpsilon`)**: Computes all nullable nonterminals via fixed-point iteration (`computeNullable`), then expands each production across all nullable subset combinations (`rhsVariants`), retaining `S -> ε` only for the start symbol.
3. **Unit-Production Elimination (`removeUnit`)**: Builds the directed graph of unit rules (`A -> B`) and computes per-nonterminal BFS transitive closures (`unitClosure`) so cyclic chains (`A -> B -> A`) terminate cleanly, replacing unit chains with direct non-unit productions.
4. **Terminal Isolation (`isolateTerminals`)**: For rules with `|RHS| >= 2`, replaces embedded terminals with reusable proxy nonterminals (`T`, `T0`, `T1`, ...) and emits corresponding `Ti -> a` productions.
5. **Binarization (`binarise`)**: Breaks productions with `|RHS| > 2` into right-cascading binary rules using fresh intermediate nonterminals (`X`, `X0`, `X1`, ...).
6. **Dead-Symbol Pruning & Deduplication (`removeNonProductive`, `removeInaccessible`, `dedupe`)**: Runs a bottom-up fixed-point pass to drop non-generating symbols followed by a top-down reachability pass from `S`, deduplicating productions with a strict ordering comparator (`ProductionLess`).

## Validation

`isChomskyNormalForm(const Grammar&)` verifies that:
- Every production has a single nonterminal on the LHS.
- Every RHS is either two nonterminals (`A -> BC`), a single terminal (`A -> a`), or empty (`S -> ε` only when `lhs == S` and `S` appears on no RHS).

## Structure

```
cnf-normalizer/
├── CMakeLists.txt
├── main.cpp            # Verification suite across 4 CFG test cases
├── include/
│   └── grammar.h       # Grammar interface, normalize(), isChomskyNormalForm()
└── src/
    └── grammar.cpp     # 6-stage CNF normalization pipeline & validator
```

## Build & Run

Requires a C++17 compiler and CMake ≥ 3.10:

```bash
mkdir -p build && cd build
cmake ..
cmake --build .
./test
```

`main.cpp` validates 4 distinct CFGs covering nullable nonterminals, unit rules, mixed/long productions (up to length 5), and unreachable symbols—asserting `!isChomskyNormalForm(g)` before normalization and `isChomskyNormalForm(gn)` after.
