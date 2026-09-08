# Dynamic Branch Predictor Using Perceptron Learning

A C-based **dynamic branch prediction simulator** developed for **CSE 360: Computer Architecture**. The project implements a simplified **Perceptron Branch Predictor** with a Global History Register (GHR), PC hashing, confidence-based training, saturated weight updates, and both synthetic and file-based branch traces.

## Overview

Branch prediction helps reduce control-hazard penalties in pipelined processors by predicting the outcome of conditional branches. This project demonstrates a history-based predictor that learns branch behavior using a perceptron model.

The simulator is designed to study how well a perceptron predictor learns structured and complex branch-history patterns, including an XOR relationship between the Program Counter (PC) and global history. fileciteturn2file0L29-L42

## Objectives

The project aims to:

- Implement a perceptron-based dynamic branch predictor.
- Maintain bias and global-history weights.
- Generate predictions using a weighted dot product.
- Train on mispredictions and low-confidence predictions.
- Apply saturated weight updates.
- Maintain and update the Global History Register (GHR).
- Map branch PCs to perceptron entries using hashing.
- Support synthetic and file-based branch traces.
- Evaluate prediction accuracy across multiple workloads. fileciteturn2file0L35-L42

## Prediction Model

The predictor calculates:

```text
Y = w0 + Σ(wi × xi)
```

where `w0` is the bias, `wi` are history weights, and `xi` represents each GHR bit as `+1` or `-1`.

Prediction rule:

```text
Y >= 0  → Taken (+1)
Y < 0   → Not Taken (-1)
```

This dot-product calculation is the core prediction mechanism described in the report. fileciteturn2file0L48-L62

## Training

The predictor trains when either:

```text
prediction is wrong
OR
|Y| <= θ
```

The low-confidence condition allows the predictor to strengthen a prediction even when the current prediction is technically correct but not confident enough. The weight update is:

```text
wi ← Saturate(wi + t × xi)
```

Saturation keeps weights inside their configured range and models limited-width hardware behavior. fileciteturn2file0L64-L74 fileciteturn2file0L133-L150

## Core Components

### Perceptron Table

The predictor stores multiple perceptrons. The global predictor state contains the table, table size, history length, weight width, weight limits, threshold `θ`, GHR, and related configuration. Each perceptron stores the bias and history weights. fileciteturn2file0L75-L86

### PC Hashing

The branch PC is hashed and then mapped to a perceptron-table entry. The implementation uses a hash-based index, helping distribute branches across the table and reduce aliasing between different branch addresses. fileciteturn2file0L152-L160

### Global History Register

The GHR stores recent branch outcomes. After each branch, it is shifted and updated with the newest outcome while preserving the configured history length. fileciteturn2file0L88-L105

## Trace Generation

The simulator supports these synthetic patterns:

| Pattern | Description |
|---|---|
| `loop` | Mostly taken with an occasional not-taken outcome |
| `alternating` | Strict `T, N, T, N...` pattern |
| `xor` | Outcome depends on the XOR of PC and GHR |
| `correlated` | Outcome depends strongly on a history bit |
| `random` | Random taken/not-taken outcomes |
| `mixed` | Combination of different workload patterns |

These patterns are implemented in the source code and were used in the project testing. fileciteturn2file0L106-L118 fileciteturn2file0L161-L190

The project also supports an external trace file. At runtime, the user can choose whether to use a synthetic pattern or read PC/outcome pairs from a trace file. fileciteturn2file1L701-L727

## Interactive Configuration

The program lets the user configure:

- Perceptron table size
- History length
- Weight bits
- Simulation steps
- Summary-print interval
- Trace-file mode
- Synthetic pattern

Default configuration:

| Parameter | Default |
|---|---:|
| Table Size | `4096` |
| History Length | `32` |
| Weight Bits | `8` |
| Simulation Steps | `200,000` |
| Pattern | `mixed` |
| Summary Interval | `10,000` |

The report uses this as the standard simulation configuration and reports `θ = 75` for a history length of 32. fileciteturn2file0L197-L206

## Workflow

```text
Program Start
     │
     ▼
Read Configuration
     │
     ▼
Create Perceptron Table
     │
     ▼
Generate / Read Branch Trace
     │
     ▼
Hash PC → Perceptron Entry
     │
     ▼
Read GHR + Calculate Y
     │
     ▼
Predict Taken / Not Taken
     │
     ▼
Compare With Actual Outcome
     │
     ▼
Train if Wrong OR Low Confidence
     │
     ▼
Update GHR
     │
     ▼
Repeat
     │
     ▼
Final Statistics
```

The actual simulation loop follows this process and reports accuracy, training statistics, timing, and throughput. fileciteturn2file1L801-L838

## Results

The report evaluates the predictor using 200,000 simulated branch outcomes per pattern:

| Trace Pattern | Correct Predictions | Accuracy |
|---|---:|---:|
| ALTERNATING | ≈199,900 | ≈99.9% |
| XOR | ≈192,000 | ≈96.0% |
| CORRELATED | ≈199,200 | ≈99.6% |
| MIXED | ≈183,000 | ≈91.5% |
| RANDOM | ≈100,200 | ≈50.1% |

These are the results reported for the project's standard simulation runs. fileciteturn2file0L192-L216

For the mixed workload:

```text
Total Predictions      200,000
Correct Predictions    183,000
Mispredictions          17,000
Overall Accuracy         91.50%
```

The report highlights the approximately **96.0% XOR accuracy** as evidence of the predictor's ability to learn the more complex PC/history relationship targeted by the project. fileciteturn2file0L218-L235

## Output Metrics

At the end of the simulation, the program reports:

- Simulated steps
- Total predictions
- Correct predictions
- Mispredictions
- Accuracy
- Trained updates
- Low-confidence updates
- Weight range
- Training threshold `θ`
- Elapsed CPU time
- Approximate predictions per second

These metrics are printed directly by the implementation. fileciteturn2file1L826-L838

## Project Structure

```text
Dynamic-Branch-Predictor/
│
├── Implement_a_dynamic_branch_predictorF.c
└── README.md
```

The current project uses a single C source file containing the predictor, trace generation/reading, interactive configuration, simulation loop, and result reporting. fileciteturn2file1L384-L425

## Requirements

- GCC, Clang, or MinGW
- A terminal / command prompt
- Standard C compiler environment

The report describes the implementation as portable C and notes compatibility with MinGW, GCC, and Clang. fileciteturn2file0L106-L115

## Build and Run

### Compile

```bash
gcc Implement_a_dynamic_branch_predictorF.c -o branch_predictor
```

### Run

Linux / macOS:

```bash
./branch_predictor
```

Windows (MinGW):

```bash
branch_predictor.exe
```

The program will then ask for the predictor configuration. Press **Enter** to accept a displayed default value.

Example:

```text
Perceptron table size (power of two) [4096]:
History length m [default 32, max 63]:
Weight bits (signed) [3..16, default 8]:
Simulation steps [200000]:
Interval for summary prints [10000]:
Read from trace file instead of synthetic pattern? (y/n) [n]:
Pattern options: loop | alternating | xor | correlated | random | mixed
Choose pattern [mixed]:
```

## Key Concepts Demonstrated

- Dynamic branch prediction
- Perceptron learning
- Global History Register (GHR)
- Bias and history weights
- Dot-product prediction
- Confidence-based training
- Saturating arithmetic
- PC hashing
- Predictor-table indexing
- Synthetic branch-trace generation
- Accuracy and performance measurement

## Limitations

This is an educational simulation of a simplified perceptron branch predictor rather than a complete hardware implementation. The main focus is the prediction and learning mechanism, along with configurable synthetic and file-based workloads.

## Future Work

The report proposes several extensions:

1. **Gated Perceptrons** — combine the perceptron with a meta-predictor and a simpler predictor such as GShare.
2. **Adaptive History Length** — dynamically adjust the effective history length.
3. **Detailed Trace Analysis** — study which classes of real branches benefit most from perceptron prediction.
4. **Hardware Emulation** — explore fixed-point and other hardware-oriented optimizations. fileciteturn2file0L247-L255

## Project Information

**Course:** CSE 360 — Computer Architecture  
**Section:** 06  
**Institution:** East West University, Department of Computer Science and Engineering  
**Project:** Dynamic Branch Predictor using Perceptron Learning  
**Language:** C

**Project Members**

- Shawna Akter
- Mahfuz Uddin Ahmed

The report lists the submission date as **16 December 2025**. fileciteturn2file0L11-L27

## Conclusion

This project demonstrates a practical implementation of a simplified **Perceptron-based Dynamic Branch Predictor**. By combining global branch history, weighted prediction, PC hashing, confidence-based training, and saturated updates, the simulator can learn a variety of branch patterns.

The reported results show very high accuracy on structured workloads and approximately **96% accuracy on the XOR workload**, which is the key complex-pattern result emphasized in the project. The implementation provides a useful foundation for exploring more advanced branch-prediction techniques and hardware-oriented designs.
