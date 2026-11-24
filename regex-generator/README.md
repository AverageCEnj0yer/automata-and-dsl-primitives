# Regex AST & String Generator

A C++17 regular expression front-end and polymorphic AST evaluator that parses regex patterns into an expression tree and samples random matching strings—serving as the syntax and AST foundation for a future NFA-backed regex engine.

## Architecture

The implementation is split into three decoupled compiler-style stages:

1. **Lexer (`include/token.h`, `include/lexer.h`, `src/lexer.cpp`)**
   Tokenizes a regex pattern string into `LPAREN`, `RPAREN`, `PIPE`, `STAR`, `PLUS`, `QUESTION`, `POWER` (`^N` with multi-digit integer lexeme), `LITERAL`, and `END`, skipping whitespace.
2. **Recursive-Descent Parser (`include/parser.h`, `src/parser.cpp`)**
   Builds an AST using a 4-level precedence hierarchy (lowest to highest):
   - `parseExpression()` — alternation (`|`) → `AlternationNode`
   - `parseSequence()` — implicit concatenation → `SequenceNode`
   - `parseFactor()` — postfix quantifiers (`+`, `*`, `?`, `^N`) → `RepeatNode`
   - `parseBase()` — grouped sub-expressions `(...)` and literals → `LiteralNode`
3. **Polymorphic AST & Generator (`include/nodes.h`, `src/nodes.cpp`)**
   Defines an abstract `Node` interface (`generate()`, `printStructure(int depth = 0)`, virtual destructor) implemented by `LiteralNode`, `SequenceNode`, `AlternationNode`, and `RepeatNode`, with recursive destructor cleanup of owned child nodes. Uses `std::mt19937` seeded via `std::random_device`; unbounded quantifiers (`*`, `+`) are capped at 5 repetitions during sampling to guarantee bounded output.

## Supported Pattern Syntax

| Construct | Meaning | Generation Bounds |
| :--- | :--- | :--- |
| `a`, `0`, ... | Single-character literal | Exact character |
| `AB` | Concatenation | In-order composition |
| `A\|B` | Alternation | Uniform random branch |
| `(A)` | Grouping / precedence override | Evaluates inner sub-tree |
| `A?` | Optional | 0 to 1 repetitions |
| `A*` | Kleene star | 0 to 5 repetitions (capped) |
| `A+` | One or more | 1 to 5 repetitions (capped) |
| `A^N` | Exact repetition count | Exactly `N` repetitions |

## Structure

```
regex-generator/
├── CMakeLists.txt
├── main.cpp              # Test harness printing ASTs and generated samples
├── include/
│   ├── token.h           # Regex TokenType enum & token printer
│   ├── lexer.h           # Regex scanner interface
│   ├── nodes.h           # Polymorphic AST node hierarchy
│   └── parser.h          # Recursive-descent parser interface
└── src/
    ├── lexer.cpp         # Lexer implementation
    ├── nodes.cpp         # AST traversal, pretty-printing & random generation
    └── parser.cpp        # Precedence-climbing recursive descent
```

## Build & Run

Requires a C++17 compiler and CMake ≥ 3.10:

```bash
mkdir -p build && cd build
cmake ..
cmake --build .
./test
```

`main.cpp` parses three multi-feature regex patterns (`(a|b)(c|d)E+G?`, `P(Q|R|S)T(UV|W|X)*Z+`, and `1(0|1)*2(3|4)^5 36`), prints each resulting AST, and generates 5 random matching strings per pattern.

### Example Output

```text
=== TEST: Nr 1 ===
Regex: (a|b)(c|d)E+G?
Tree Structure:
Sequence:
  Alternation (Choice):
    Literal: 'a'
    Literal: 'b'
  Alternation (Choice):
    Literal: 'c'
    Literal: 'd'
  Repeat (1 to 5):
    Literal: 'E'
  Repeat (0 to 1):
    Literal: 'G'
Generated Strings:
  1: acEEEG
  2: bdEG
  ...
```
