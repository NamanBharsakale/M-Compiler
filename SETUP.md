# MathLang Setup Guide

This guide covers installing dependencies and running MathLang as a local web application or command-line compiler.

## Requirements

- Python 3.11 or newer
- pip
- Git (only needed to clone the repository)

## Get the project

Clone the repository, then change into its directory:

```bash
git clone https://github.com/PratikRathod-tech/COMPILER_DESIGN_PROJECT.git
cd COMPILER_DESIGN_PROJECT
```

If you already have the project, open a terminal in its root directory—the directory containing `main.py` and `requirements.txt`.

## Create and activate a virtual environment

Using a virtual environment is recommended.

### Windows PowerShell

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

If PowerShell blocks activation, allow scripts for this terminal session and try again:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
.\venv\Scripts\Activate.ps1
```

### Windows Command Prompt

```bat
python -m venv venv
venv\Scripts\activate.bat
```

### macOS or Linux

```bash
python3 -m venv venv
source venv/bin/activate
```

## Install dependencies

From the project root, run:

```bash
python -m pip install -r requirements.txt
```

The requirements include pytest, NumPy, and scikit-learn.

## Run the web application

Start the local server:

```bash
python main.py
```

Open [http://localhost:5000](http://localhost:5000) in your browser. The page contains a MathLang editor, example presets, Run and Workflow actions, a result view, and tabs for inspecting compiler phases. The Workflow view displays the compilation pipeline.

Stop the server with `Ctrl+C` in the terminal where it is running. Port 5000 must be available.

## Run a source file from the command line

Pass a `.math` source file to `main.py`:

```bash
python main.py examples/01_basic.math
```

The compiler prints the pipeline and execution result in the terminal. Other included examples can be run the same way, for example:

```bash
python main.py examples/05_vectors.math
python main.py examples/06_matrices.math
python main.py examples/linear.math
python main.py examples/metrics.math
```

The command-line mode accepts a source-file path; phase-specific options such as `--tokens` or `--ast` are not implemented. Use the web app's phase tabs or Workflow view to inspect those outputs.

## Run tests

If a test suite is included in your checkout, run it from the project root:

```bash
python -m pytest
```

## Troubleshooting

### `python` is not recognized

Install Python and enable the option to add it to `PATH`, or use the Python launcher `py` on Windows (for example, `py -m venv venv`).

### `pip` or a package is not found

Activate the virtual environment and run `python -m pip install -r requirements.txt` from the project root.

### `ModuleNotFoundError` for `src`

Run commands from the project root, not from inside `src/` or `examples/`.

### Port 5000 is already in use

Stop the other process using the port, then start the application again. The current server uses port 5000.

### PowerShell does not allow virtual-environment activation

Use the process-scoped execution policy command in the Windows PowerShell instructions above, or activate the environment from Command Prompt instead.
