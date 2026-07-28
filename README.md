# c-to-LLVM-ir
# C to LLVM IR Conversion & CFG Visualization Tool

A Python-based command-line tool that compiles C source files to LLVM Intermediate
Representation (IR), extracts and renders their Control Flow Graphs (CFGs), and runs
lightweight static checks (e.g. division-by-zero detection) directly on the IR.

## What it does

1. **Compiles C → LLVM IR** using `clang`, producing a human-readable `.ll` file for
   any input `.c` program.
2. **Generates the Control Flow Graph** for each function using LLVM's `opt
   -dot-cfg` pass and renders it as a PNG with Graphviz.
3. **Runs structural/static analysis on the IR**, including:
   - Basic block counting
   - Instruction and function counting
   - Division-by-zero detection (flags `sdiv`/`udiv` instructions with a
     constant zero divisor)
4. **Reports complexity metrics** (instructions + basic blocks) that can be
   aggregated across many files to compare program complexity.

## Why

Reading raw LLVM IR and manually tracing control flow is tedious. This tool
automates the compile -> analyze -> visualize loop so you can quickly inspect how a
C program's control flow and structural complexity look at the IR level, and
catch simple correctness issues (like a literal division by zero) purely from
the IR - without executing the program.

## Tech stack

- **Clang / LLVM (`opt`)** - compiling C to IR and generating CFG `.dot` files
- **Graphviz** - rendering `.dot` files to PNG images
- **Python** - orchestration, IR parsing, and metric computation
- **pandas / matplotlib** - aggregating and plotting complexity statistics across
  a dataset of C programs

## Requirements

- `clang`, `opt`, and `graphviz` installed and on your `PATH`
- Python 3 with `pandas` and `matplotlib`

On Debian/Ubuntu (or in a Colab notebook):

```bash
apt update
apt install llvm clang graphviz -y
pip install pandas matplotlib
```

## Usage

Run the tool on a single C file:

```bash
python ir_tool.py path/to/program.c
```

This produces, alongside the input file:

- `program.ll` - the (cleaned) LLVM IR
- `program_cfg.png` - the rendered control flow graph
- Console output flagging any detected division-by-zero instructions

Example:

```bash
python ir_tool.py sample.c
```

```c
// sample.c
int add(int a, int b) {
    return a + b;
}

int main() {
    int result = add(3, 4);
    printf("%d\n", result);
    return 0;
}
```

To flag a division-by-zero:

```c
int main() {
    int x = 10;
    int y = 0;
    int z = x / 0;   // detected by the static check
    return z;
}
```

## Batch analysis

The included notebook (`Untitled7.ipynb`) also shows how to run the tool over an
entire corpus of C programs (sourced from
[TheAlgorithms/C](https://github.com/TheAlgorithms/C) and a public Kaggle C
programs dataset) to compute a `complexity_score` (instructions + 2 x basic
blocks) per program and plot its distribution across the dataset.

## Project structure

```
.
├── ir_tool.py          # Core CLI tool: compile, analyze, visualize
├── Untitled7.ipynb     # Exploration notebook / batch analysis over a C dataset
└── README.md
```

## Notes

- Built and tested in a Google Colab environment; works the same locally as long
  as `clang`, `opt`, and `graphviz` are installed.
- This is an exploratory/analysis tool, not a full static analyzer - the
  division-by-zero check is a simple pattern match on IR instructions with a
  constant zero divisor, not a full data-flow analysis.
