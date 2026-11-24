# DSL Compiler Front-End (Lexer, LL(3) Parser & AST)

A C++17 front-end for a statically typed imperative domain-specific language (DSL). It reads source files into memory, tokenizes them via a hand-written lexer, parses the token stream with an **LL(3) recursive-descent parser**, and constructs a strongly owned polymorphic **Abstract Syntax Tree (AST)** using `std::unique_ptr`.

## Architecture

### 1. Lexical Analysis (`token.h`, `lexer.h`, `lexer.cpp`)
- Loads the input file into memory and emits a stream of 36 `TokenType` variants.
- Recognizes:
  - **Type keywords**: `int`, `float`, `string`, `bool`, `void`
  - **Keywords & logical operators**: `true`, `false`, `return`, `AND`, `OR`, `NOT`
  - **Literals & identifiers**: integers, floats (classifying malformed tokens like `1.2.3` or `123abc` as `UNKNOWN`), double-quoted strings, and alphanumeric/underscore identifiers
  - **Operators & delimiters**: `+`, `-`, `*`, `/`, `^`, `=`, `==`, `!=`, `<`, `<=`, `>`, `>=`, `(`, `)`, `,`, `;`, `{`, `}`
- Unrecognized characters emit single-character `UNKNOWN` tokens without stalling the scanner.

### 2. LL(3) Recursive-Descent Parser (`parser.h`, `parser.cpp`)
- Maintains a **3-token lookahead window** (`m_current`, `m_next`, `m_nextNext`) to disambiguate top-level function declarations (`TYPE IDENTIFIER "("`) from variable declarations (`TYPE IDENTIFIER ("=" | ";")`), and 2-token lookahead to distinguish assignment (`IDENTIFIER "="`) from equality (`IDENTIFIER "=="`).
- Reports syntax errors by throwing `ParseError` (`std::runtime_error`).

#### Supported Grammar (lowest to highest expression precedence)

```ebnf
program     -> (funcDecl | statement)* EOF
funcDecl    -> TYPE IDENTIFIER "(" paramList ")" block
statement   -> varDecl | returnStmt | block | exprStmt
varDecl     -> TYPE IDENTIFIER ("=" expression)? ";"
returnStmt  -> "return" expression? ";"
block       -> "{" statement* "}"
exprStmt    -> expression ";"

expression  -> assignment
assignment  -> IDENTIFIER "=" expression            (* right-associative *)
             | logicalOr
logicalOr   -> logicalAnd  ("OR"  logicalAnd)*
logicalAnd  -> equality    ("AND" equality)*
equality    -> comparison  (("==" | "!=") comparison)*
comparison  -> term        (("<" | ">" | "<=" | ">=") term)*
term        -> factor      (("+" | "-") factor)*
factor      -> power       (("*" | "/") power)*
power       -> unary       ("^" power)?             (* right-associative *)
unary       -> ("-" | "NOT") unary | primary
primary     -> INT | FLOAT | STRING | "true" | "false"
             | IDENTIFIER "(" argList ")"
             | IDENTIFIER
             | "(" expression ")"
```

### 3. RAII Abstract Syntax Tree (`ast.h`)
Header-only hierarchy rooted at `ASTNode` with 16 concrete node types owned via `std::unique_ptr<ASTNode>` (supporting nullable optional children such as uninitialized declarations and bare `return;` statements):
- **Root & Declarations**: `ProgramNode`, `FunctionDeclNode`, `ParameterNode`, `VarDeclNode`
- **Statements**: `BlockNode`, `AssignNode`, `ReturnNode`, `ExprStmtNode`
- **Expressions**: `BinaryOpNode`, `UnaryOpNode`, `FunctionCallNode`
- **Leaves**: `IntLiteralNode`, `FloatLiteralNode`, `StringLiteralNode`, `BoolLiteralNode`, `IdentifierNode`

## Structure

```
dsl-frontend/
├── token.h          # TokenType enum, Token struct, token formatting helpers
├── lexer.h / .cpp   # File-backed lexical scanner
├── ast.h            # Polymorphic RAII AST node hierarchy
├── parser.h / .cpp  # LL(3) recursive-descent parser & ParseError
├── main.cpp         # CLI driver: parses a source file and prints its AST
├── sample.txt       # Full program demonstrating functions, blocks, calls & expressions
└── tests/
    ├── test1.lex    # Multiple typed function declarations
    ├── test2.lex    # Whitespace-insensitive declarations & logical operators
    ├── test3.lex    # Intentional invalid characters (lexer error-handling sample)
    └── test4.lex    # All primitive literal types
```

## Build & Run

Requires a C++17 compiler (`g++` or `clang++`) with zero external dependencies:

```bash
g++ -std=c++17 main.cpp lexer.cpp parser.cpp -o dsl_parser
./dsl_parser sample.txt
```

You can also parse the additional valid test inputs:

```bash
./dsl_parser tests/test1.lex
./dsl_parser tests/test2.lex
./dsl_parser tests/test4.lex
```

*(Note: `tests/test3.lex` contains intentionally invalid symbols such as `$`, `#`, `[`, `]`, and `@` used for lexer error-handling verification, so the parser raises a `ParseError` on it.)*

### Example AST Output (`sample.txt`)

```text
Program
  FunctionDecl(int main)
    Params
    Block
      VarDecl(int x)
        IntLiteral(10)
      ...
      VarDecl(bool result)
        BinaryOp(AND)
          BinaryOp(>)
            Identifier(x)
            IntLiteral(5)
          UnaryOp(NOT)
            BoolLiteral(false)
      VarDecl(int z)
        FunctionCall(add)
          Identifier(x)
          BinaryOp(^)
            Identifier(y)
            IntLiteral(2)
```
