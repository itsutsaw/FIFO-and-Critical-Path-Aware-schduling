# 🚀 FIFO and Critical-Path Aware Issue Scheduling in sim-outorder

### 🧑‍💻 Authors
- **Utsaw Kumar (25CS06023)**  
- **Abhishek Kumar (25CS06012)**  

---


###**🚀 How to Run**
# Build the Simulator
cd simplesim-3.0
make clean
make sim-outorder
# Run a Benchmark
./sim-outorder tests/bin.little/test-math > output.txt 2>&1
# Extract Key Metrics
grep -E "sim_cycle|sim_IPC|ruu_latency" output.txt




## 📘 Project Overview
This project modifies the **instruction issue scheduling** logic of the `sim-outorder` simulator in the **SimpleScalar toolset**.  
Modern out-of-order (OoO) processors exploit *instruction-level parallelism (ILP)* by dynamically reordering instructions.  
We implemented and analyzed two scheduling policies:

1. **FIFO Ready Queue Scheduling** — oldest ready instruction issues first.  
2. **Critical-Path (Slip-Based) Scheduling** — prioritizes the most delayed instruction.

These modifications aim to study how scheduler design affects key processor metrics such as **IPC**, **latency**, and **total cycles**.

---

## 🎯 Objective
The objective of this project is to:
- Understand the internal behavior of out-of-order pipelines in SimpleScalar.  
- Modify the scheduler to implement FIFO and slip-based (critical-path) scheduling.  
- Evaluate their performance impact on various benchmarks.  
- Compare results with the default age-based scheduler.

---

## ⚙️ Methodology

### 🔹 Source Code Analysis
- Located scheduling logic in `sim-outorder.c`.  
- Identified key structures:
  - `RUU_station` — tracks in-flight instructions.
  - `ready_queue` — holds ready-to-issue instructions.
  - `ruu_issue()` — core issue logic.

### 🔹 FIFO Ready Queue Implementation
- Modified `readyq_enqueue()` to insert instructions in the order they become ready.  
- Added `ready_cycle` timestamp to each instruction for tracking.  
- Ensured fairness: the instruction waiting longest in the ready queue issues first.

### 🔹 Critical-Path (Slip-Based) Scheduling
- Introduced **slip = sim_cycle - rs->seq**, representing instruction delay since decoding.  
- Sorted ready queue each cycle before issuing:  
  1. Highest slip first (most critical).  
  2. If tie → instruction that became ready first (`ready_cycle`).  
  3. If tie → older program order (`seq`).  
- This approach prioritizes instructions along the critical dependency chain.

### 🔹 Compilation and Verification
- Recompiled simulator using:
  ```bash
  make clean
  make sim-outorder

**🔹 Benchmarking**

Benchmarks from tests/bin.little/:
test-math, test-fmath, anagram, test-printf, test-lswlr, test-llong

**Compilation and Verification**

./sim-outorder tests/bin.little/test-math > baseline_math.txt 2>&1
grep -E "sim_cycle|sim_IPC|ruu_latency" baseline_math.txt


**Table Metrics**

| Benchmark       | Policy   | sim_cycle | sim_IPC | ruu_latency |
| --------------- | -------- | --------- | ------- | ----------- |
| **test-math**   | Baseline | 202185    | 0.9367  | 6.3514      |
|                 | Modified | 202313    | 0.9361  | 6.3219      |
| **test-fmath**  | Baseline | 59162     | 0.7342  | 6.4712      |
|                 | Modified | 59187     | 0.7339  | 6.4436      |
| **anagram**     | Baseline | 12422     | 0.5692  | 4.4792      |
|                 | Modified | 12419     | 0.5693  | 4.4643      |
| **test-printf** | Baseline | 1157407   | 1.0885  | 7.7979      |
|                 | Modified | 1152115   | 1.0935  | 7.7191      |
| **test-llong**  | Baseline | 25242     | 0.8115  | 8.5585      |
|                 | Modified | 25220     | 0.8122  | 8.5134      |
