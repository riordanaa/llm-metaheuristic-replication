# Can an LLM replicate a metaheuristic from the paper alone?

Code for **Large Language Models for Metaheuristic Implementation: A Case Study with Variable Neighborhood Search**,
A. Riordan, D. Aloise, R. Martín-Santamaría. ICVNS 2025, Springer LNCS 16256, pp. 150–165, 2026.
[doi:10.1007/978-3-032-19582-1_11](https://doi.org/10.1007/978-3-032-19582-1_11)

The target is the "less is more" VNS (LIMA) for balanced minimum sum-of-squares clustering by Costa, Aloise and
Mladenović (2017). Its authors never released the original code, and the paper's swap-move cost formula is
missing an interaction term.

`LocalSearch.cpp` and `VNS.cpp` in this repository were written by an LLM (Gemini 2.5 Pro). It worked from the
paper, the essential framework files and the C++ headers, with minimal human guidance. We used the
**Context, Communication, Iteration (CCI)** prompting workflow described in the paper.

## Results

- Gemini's implementation matched the original within 0.01%: mean gap 0.032%, median 0.009%, over 160 paired
  runs (16 UCI datasets × 10 runs).
- A one-sided Wilcoxon non-inferiority test, with a 0.01% margin, gives p ≈ 2×10⁻⁶.
- Gemini correctly included the interaction term that is missing from the paper.
- Claude Sonnet 4 copied the formula as printed and worked around the resulting errors with periodic full
  recomputation. That is not a correct local search, so its version was not benchmarked.

| | |
|---|---|
| LLM-generated | `src/LocalSearch.cpp`, `src/VNS.cpp` |
| Given to the LLM | the original paper, essential framework files (solution representation, objective function, etc.), C++ headers for LocalSearch and VNS |
| Original implementation | not public; provided by its authors for this comparison |

Chat logs: [Gemini 2.5 Pro session (final implementation)](https://gemini.google.com/share/c1d3ab4a6b54) ·
[Claude Sonnet 4 session (comparison)](https://claude.ai/share/50ac7f73-df99-4c96-a52a-50fa6796053f)

## Fair Comparison Setup

To ensure accurate performance comparison with the original LIMA VNS:

- **Identical Initial Solutions:** The `initial_solutions/` folder contains the same starting solutions used to benchmark the original implementation
- **Modified Execution:** `run_with_init` script runs experiments using these pre-generated initial solutions
- **Same Test Framework:** Uses identical datasets, parameters, and evaluation metrics

## Compiling

You only have to use the makefile in the *"src"* folder:

```bash
make
```

To clean the compiled files use:

```bash
make clean
```

## Executing

### Standard Execution

32-bits version:
```bash
./lima_vns_32 <path/instance.csv> <k=number of clusters> <cpu time limit> <number of runs> <seed> <path/output file> <path/cluster assignment file>
```

64-bits version:
```bash
./lima_vns_64 <path/instance.csv> <k=number of clusters> <cpu time limit> <number of runs> <seed> <path/output file> <path/cluster assignment file>
```

The cluster assignments file has the instance used as the first column. The remaining columns are the cluster assignments, for example, "iris.csv, 0, 0, 1, ...", which means:
- point 1 is assigned to cluster 0;
- point 2 is assigned to cluster 0;  
- point 3 is assigned to cluster 1;

### Controlled Comparison (Recommended)
Run with identical initial solutions for fair comparison:
```bash
./run_with_init
```

This script:
- Uses pre-generated initial solutions from `initial_solutions/` folder
- Runs 10 experiments per dataset
- Saves results to `results/` and assignments to `assignments/`
- Matches the experimental setup used for the original implementation

### Analyze Results
```bash
./analyze_results
```

Generates summary statistics (best, worst, mean, variance) for all experimental runs.

## Repository Structure

```
├── src/                    # Source code
│   ├── LocalSearch.cpp     # 🤖 LLM-Generated
│   ├── VNS.cpp            # 🤖 LLM-Generated  
│   ├── [framework files]   # Original supporting code
│   └── Makefile
├── datasets/              # Test instances
├── initial_solutions/     # Pre-generated starting solutions
├── run_with_init         # Modified execution script
├── analyze_results       # Results analysis script
├── results/              # Output directory (created on run)
└── assignments/          # Cluster assignments (created on run)
```

## The CCI framework

- **Context:** the full paper plus a project skeleton, with no algorithm code. We were careful not to include
  data structures that would steer the model toward a particular design.
- **Communication:** the task is broken into steps (summary, component plan, pseudocode, then C++), each with
  a clear goal.
- **Iteration:** compile and run errors and performance metrics go back to the model. An independent solution
  validator, which the model never writes, guards against hallucinated results.

## Citation

If you use this code, please cite our paper and the original algorithm:

```bibtex
@inproceedings{riordan2026llm,
  title={Large Language Models for Metaheuristic Implementation: A Case Study with Variable Neighborhood Search},
  author={Riordan, Aidan and Aloise, Daniel and Mart{'\i}n-Santamar{'\i}a, Ra{'u}l},
  booktitle={Variable Neighborhood Search (ICVNS 2025)},
  series={Lecture Notes in Computer Science},
  volume={16256},
  pages={150--165},
  year={2026},
  publisher={Springer},
  doi={10.1007/978-3-032-19582-1_11}
}
```

```bibtex
@article{costa2017lima,
  title={Less is more: basic variable neighborhood search heuristic for balanced minimum sum-of-squares clustering},
  author={Costa, Leandro R and Aloise, Daniel and Mladenovi{\'c}, Nenad},
  journal={Information Sciences},
  volume={415},
  pages={104--118},
  year={2017},
  publisher={Elsevier}
}
```
