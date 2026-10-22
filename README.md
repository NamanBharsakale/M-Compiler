# MathLang Compiler

> A small compiler for mathematical programs: parse → check → classify → IR → optimize → Python → execute. Use it from a browser UI or the command line.

![Python](https://img.shields.io/badge/python-3.11%2B-blue)
![Interface](https://img.shields.io/badge/interface-web%20%2B%20CLI-green)

MathLang parses source code, runs semantic checks, classifies the program with a local ML model, generates and optimizes an intermediate representation (IR), generates Python from the optimized IR, and executes it.

## Table of contents

- [Features](#features)
- [Architecture](#architecture)
- [Compiler pipeline](#compiler-pipeline)
- [Phase-by-phase walkthrough](#phase-by-phase-walkthrough)
- [Project structure](#project-structure)
- [Setup](#setup)
- [Run the web interface](#run-the-web-interface)
- [Run a `.math` file](#run-a-math-file)
- [Examples](#examples)
- [Tests](#tests)

---

## Features

- Numeric variables, arithmetic expressions, and standard operator precedence
- Scalar math functions: `sqrt`, `sin`, `cos`, `tan`, `log`, `abs`
- Vector literals and operations: `dot`, `norm`
- Matrix literals and operations: `transpose`, `det`/`determinant`, `inverse`
- `show` statements for displaying expression results
- Linear equation solving with `solve`
- Regression metrics: `mae`, `mse`, `rmse`, `r2`
- Local ML classifier that assigns a category and confidence to a program
- Browser views for the result, compiler-phase output, and the full workflow

Language syntax and supported operations: [docs/LANGUAGE_SPECIFICATION.md](docs/LANGUAGE_SPECIFICATION.md)

---

## Architecture

### System overview

```mermaid
flowchart LR
    subgraph Clients
        W["Browser UI<br/>web/index.html + app.js"]
        C["CLI<br/>python main.py file.math"]
    end

    M["main.py<br/>local server + CLI entry"]
    O["src/compiler<br/>orchestration"]

    subgraph Pipeline["Compiler pipeline (src/)"]
        direction LR
        L["lexer"] --> P["parser"] --> S["semantic"] --> ML["ml"] --> IR["ir"] --> OPT["optimizer"] --> CG["codegen"]
    end

    RT["runtime +<br/>math_engine"]
    OUT["Result / phase output"]

    W -->|"source code"| M
    C -->|"file path"| M
    M --> O --> Pipeline
    CG -->|"generated Python"| RT
    RT --> OUT
    OUT --> M
    M --> W
    M --> C
```

### Web request flow

```mermaid
sequenceDiagram
    actor U as User
    participant B as Browser (app.js)
    participant S as main.py server
    participant K as Compiler
    participant R as Runtime

    U->>B: Enter program, click Run
    B->>S: Send source code
    S->>K: compile(source)
    K->>K: Lex, parse, analyze, classify
    K->>K: Generate IR, optimize, generate Python
    K->>R: Execute generated Python
    R-->>K: Output / errors
    K-->>S: Phase results + execution output
    S-->>B: Results for all views
    B-->>U: Result, Phases, and Workflow views
```

---

## Compiler pipeline

The browser's **Workflow** view presents these ten stages:

```mermaid
flowchart TD
    A["1. Source code"] --> B["2. Lexical analysis"]
    B --> C["3. Syntax analysis"]
    C --> D["4. AST"]
    D --> E["5. Semantic analysis"]
    E --> F["6. Symbol table"]
    F --> G["7. ML classification"]
    G --> H["8. Intermediate representation"]
    H --> I["9. Optimization"]
    I --> J["10. Execution"]

    classDef front fill:#dbeafe,stroke:#2563eb,color:#1e3a8a;
    classDef mid fill:#fef3c7,stroke:#d97706,color:#78350f;
    classDef back fill:#dcfce7,stroke:#16a34a,color:#14532d;
    class A,B,C,D front;
    class E,F,G mid;
    class H,I,J back;
```

| Group | Stages | Purpose |
|-------|--------|---------|
| Front end | 1–4 | Turn text into a structured tree |
| Analysis | 5–7 | Validate meaning and classify the program |
| Back end | 8–10 | Generate, optimize, and run code |

Internally, the compiler generates Python from the optimized IR before execution. The web interface also provides phase tabs for tokens, AST, semantic analysis, symbols, ML classification, IR, optimized IR, and execution.

### Error handling

```mermaid
flowchart LR
    S["Source"] --> LX{"Lexer OK?"}
    LX -- no --> E1["Lexical error"]
    LX -- yes --> PS{"Parser OK?"}
    PS -- no --> E2["Syntax error"]
    PS -- yes --> SM{"Semantic OK?"}
    SM -- no --> E3["Semantic error"]
    SM -- yes --> CONT["Classify, IR, optimize, codegen"]
    CONT --> EX{"Runtime OK?"}
    EX -- no --> E4["Runtime error"]
    EX -- yes --> OK["Output"]
```

---

## Phase-by-phase walkthrough

All phases below follow this example program. The intermediate formats shown are **illustrative** of what each phase produces; exact token names, node names, and IR shapes depend on the implementation (see [docs/](docs/)).

```text
x = 10
y = 20
z = x + y
show z
```

### Phase 1 · Source code

Plain text from the browser editor, a preset, or a `.math` file.

```mermaid
flowchart LR
    A["Editor / preset"] --> S["Source string"]
    B[".math file"] --> S
```

### Phase 2 · Lexical analysis (`src/lexer/`)

Characters become tokens.

```mermaid
flowchart LR
    A["z = x + y"] --> B["IDENT z"] --> C["ASSIGN ="] --> D["IDENT x"] --> E["PLUS +"] --> F["IDENT y"]
```

### Phase 3 and 4 · Syntax analysis and AST (`src/parser/`)

The parser applies grammar and operator precedence and builds the AST.

```mermaid
flowchart TD
    PR["Program"] --> A1["Assign x"]
    PR --> A2["Assign y"]
    PR --> A3["Assign z"]
    PR --> SH["Show z"]
    A1 --> N1["Number 10"]
    A2 --> N2["Number 20"]
    A3 --> BIN["BinaryOp +"]
    BIN --> V1["Var x"]
    BIN --> V2["Var y"]
    SH --> V3["Var z"]
```

### Phase 5 and 6 · Semantic analysis and symbol table (`src/semantic/`)

Checks that names are defined before use and that operations are valid. Declared names go into the symbol table.

```mermaid
flowchart LR
    AST["AST"] --> CHK{"Names defined?<br/>Operations valid?"}
    CHK -- yes --> ST["Symbol table<br/>x, y, z"]
    CHK -- no --> ERR["Semantic error"]
```

| Symbol | Kind (illustrative) |
|--------|---------------------|
| `x` | number |
| `y` | number |
| `z` | number |

### Phase 7 · ML classification (`src/ml/`)

A local classifier assigns the program a category and a confidence score.

```mermaid
flowchart LR
    SRC["Program"] --> FE["Feature extraction"] --> MODEL["Local classifier"] --> OUT["Category + confidence"]
```

### Phase 8 · Intermediate representation (`src/ir/`)

The AST is lowered to a simple, linear form.

```mermaid
flowchart LR
    AST["AST"] --> LOW["Lowering"] --> IR["t1 = 10<br/>t2 = 20<br/>t3 = t1 + t2<br/>show t3"]
```

### Phase 9 · Optimization (`src/optimizer/`)

The optimizer rewrites the IR into a cheaper equivalent. For example, constant folding (illustrative):

```mermaid
flowchart LR
    IR["t1 = 10<br/>t2 = 20<br/>t3 = t1 + t2<br/>show t3"] --> OPT["Optimizer"] --> OIR["t3 = 30<br/>show t3"]
```

### Phase 10 · Code generation and execution (`src/codegen/`, `src/runtime/`, `src/math_engine/`)

Python is generated from the optimized IR and executed. Runtime helpers and the math engine handle vectors, matrices, `solve`, and regression metrics.

```mermaid
flowchart LR
    OIR["Optimized IR"] --> CG["Python code generator"] --> PY["Generated Python"]
    PY --> EXE["Execute"]
    RT["runtime + math_engine"] --> EXE
    EXE --> OUT["Output: 30"]
```

### All phases at a glance

```mermaid
flowchart LR
    P1["1 Source"] --> P2["2 Tokens"] --> P3["3-4 AST"] --> P4["5-6 Semantic +<br/>Symbols"] --> P5["7 ML class"] --> P6["8 IR"] --> P7["9 Optimized IR"] --> P8["10 Execution"]
```

---

## Project structure

```text
.
├── main.py                 # Local web server and file-based CLI entry point
├── requirements.txt        # Python dependencies
├── SETUP.md                # Installation and run instructions
├── docs/                   # Language and compiler phase documentation
├── examples/               # Sample .math programs
├── src/
│   ├── lexer/              # Tokenizer and token definitions
│   ├── parser/             # Parser and AST node definitions
│   ├── semantic/           # Semantic checks and symbol table
│   ├── ml/                 # Local program classifier
│   ├── ir/                 # Intermediate representation
│   ├── optimizer/          # IR optimization
│   ├── codegen/            # Python code generation
│   ├── runtime/            # Math runtime helpers
│   ├── math_engine/        # Equation, matrix, and metric operations
│   └── compiler/           # Compiler orchestration
└── web/
    ├── index.html          # Browser interface structure
    ├── style.css           # Interface styling
    └── app.js              # Browser interaction and rendering
```

---

## Setup

Requirements: Python 3.11 or newer and pip. From the project root:

```bash
python -m pip install -r requirements.txt
```

A virtual environment is recommended. Platform-specific steps are in [SETUP.md](SETUP.md).

---

## Run the web interface

```bash
python main.py
```

Open [http://localhost:5000](http://localhost:5000). Enter a program or pick a preset, then choose **Run**.

- **Workflow** shows the compiler stages.
- The output tabs let you inspect individual phase results.
- The terminal also logs the pipeline.

Stop the server with `Ctrl+C`. Port 5000 must be available.

---

## Run a `.math` file

```bash
python main.py examples/01_basic.math
python main.py examples/05_vectors.math
python main.py examples/06_matrices.math
```

File mode prints the compiler pipeline and final execution output. CLI phase flags are not implemented; use the browser interface to inspect individual phase outputs.

---

## Examples

The `examples/` directory includes programs for basic expressions, functions, precedence, errors, vectors, matrices, linear equations, and regression metrics.

```text
x = 10
y = 20
z = x + y
show z
```

```bash
python main.py examples/01_basic.math
```

---

## Tests

If a test suite is included in your checkout, run it from the project root:

```bash
python -m pytest
```