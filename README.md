# MathLang Compiler

MathLang is a small compiler for mathematical programs. It can be used through a browser-based local interface or from the command line. The compiler parses MathLang source, checks it semantically, classifies the program, generates and optimizes an intermediate representation, generates Python code, and executes it.

## Features

- Numeric variables, arithmetic expressions, and standard operator precedence
- Scalar math functions: `sqrt`, `sin`, `cos`, `tan`, `log`, and `abs`
- Vector literals and operations: `dot` and `norm`
- Matrix literals and operations, including `transpose`, `det`/`determinant`, and `inverse`
- `show` statements for displaying expression results
- Linear equation solving with `solve`
- Regression metrics: `mae`, `mse`, `rmse`, and `r2`
- A local ML classifier that assigns a category and confidence to a program
- Browser views for the result, compiler-phase output, and full workflow

See [docs/LANGUAGE_SPECIFICATION.md](docs/LANGUAGE_SPECIFICATION.md) for language syntax and supported operations.

## Compiler pipeline

The browser's Workflow view presents these ten stages:

1. Source code
2. Lexical analysis
3. Syntax analysis
4. AST
5. Semantic analysis
6. Symbol table
7. ML classification
8. Intermediate representation (IR)
9. Optimization
10. Execution

Internally, the compiler generates Python from the optimized IR before execution. The web interface also provides phase tabs for tokens, AST, semantic analysis, symbols, ML classification, IR, optimized IR, and execution.

## Project structure

```text
.
├── main.py                 # Local web server and file-based CLI entry point
├── requirements.txt        # Python dependencies
├── SETUP.md                # Installation and run instructions
├── docs/                   # Language and compiler phase documentation
├── examples/               # Sample .math programs
├── src/
│   ├── lexer/               # Tokenizer and token definitions
│   ├── parser/              # Parser and AST node definitions
│   ├── semantic/            # Semantic checks and symbol table
│   ├── ml/                  # Local program classifier
│   ├── ir/                  # Intermediate representation
│   ├── optimizer/           # IR optimization
│   ├── codegen/             # Python code generation
│   ├── runtime/             # Math runtime helpers
│   ├── math_engine/         # Equation, matrix, and metric operations
│   └── compiler/            # Compiler orchestration
└── web/
    ├── index.html           # Browser interface structure
    ├── style.css            # Interface styling
    └── app.js               # Browser interaction and rendering
```

## Setup

Requirements are Python 3.11 or newer and pip. From the project root, install dependencies:

```bash
python -m pip install -r requirements.txt
```

Using a virtual environment is recommended. Platform-specific setup steps are in [SETUP.md](SETUP.md).

## Run the web interface

Start the server from the project root:

```bash
python main.py
```

Then open [http://localhost:5000](http://localhost:5000) in a browser. Enter a program in the editor or select a preset, then choose **Run** to compile and execute it. **Workflow** displays the compiler stages; the output tabs let you inspect individual results. The server logs the pipeline in the terminal as well.

Stop the server with `Ctrl+C`. Port 5000 must be available.

## Run a `.math` file

Pass the path of a source file to `main.py` to compile and run it in the terminal:

```bash
python main.py examples/01_basic.math
python main.py examples/05_vectors.math
python main.py examples/06_matrices.math
```

This file mode prints the compiler pipeline and final execution output. CLI phase flags are not implemented; use the browser interface to inspect individual phase outputs.

## Examples

The `examples/` directory includes programs demonstrating basic expressions, functions, precedence, errors, vectors, matrices, linear equations, and regression metrics. For example:

```text
x = 10
y = 20
z = x + y
show z
```

Run it with:

```bash
python main.py examples/01_basic.math
```

## Tests

If a test suite is included in your checkout, run it from the project root:

```bash
python -m pytest
```
