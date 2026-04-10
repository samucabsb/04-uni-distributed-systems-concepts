# Performance Evaluation Report - MPI

**Course:** Distributed Systems / Concurrent Programming
**Student:** Samuel de Souza Rodrigues
**Term:** 5th Semester - Systems Analysis and Development
**Professor:** Rafael
**Date:** April 10, 2026

---

# 1. Problem Description

The computational problem solved consists of analyzing text similarity between pairs of questions from a real dataset ("Quora Question Pairs"). The goal is to identify the degree of similarity between texts to detect potential duplicates.

* **Implemented Problem:** Identification of textual similarity through the comparison of normalized and tokenized strings.
* **Algorithm Used:** Jaccard Similarity, which calculates the ratio between the intersection and the union of the sets of words (tokens) of two sentences.
* **Input Size:** 5,000 unique questions, generating a total of 12,492,501 pair comparisons ($n \times (n-1) / 2$).
* **Parallelization Goal:** Reduce the processing time of a quadratic complexity $O(n^2)$ algorithm by distributing the workload of the comparison loop across multiple processing cores.

---

# 2. Experimental Environment

The experiments were performed on local hardware with the following specifications:

| Item | Description |
| :--- | :--- |
| **Processor** | AMD Ryzen 5 7600X (4.70 GHz) |
| **Number of Cores** | 6 Physical Cores / 12 Threads (SMT) |
| **RAM** | 16.0 GB (15.2 GB usable) |
| **Operating System** | Windows 11 Pro (64-bit) |
| **Language Used** | Python 3.14 |
| **Parallelization Library** | mpi4py, pandas |
| **Compiler / Version** | Microsoft MPI (MS-MPI) |

---

# 3. Testing Methodology

* **Time Measurement:** Time was measured using Python's `time.time()` function, capturing the actual elapsed interval (*wall-clock time*) from the start of data loading to the completion of result collection.
* **Number of Executions:** One complete execution was performed for each process configuration.
* **Input Size:** Fixed at 5,000 lines from the original dataset to ensure metric consistency.

### Tested Configurations

* 1 process (original serial version `avaliador.py`)
* 2 processes (MPI)
* 4 processes (MPI)
* 8 processes (MPI)
* 12 processes (MPI)

---

# 4. Experimental Results

Execution times obtained during testing:

| No. Threads/Processes | Execution Time (s) |
| :--- | :--- |
| 1 (Serial) | 19.02 |
| 2 | 22.16 |
| 4 | 14.46 |
| 8 | 10.21 |
| 12 | 9.07 |

---

# 5. Speedup and Efficiency Calculation

### Speedup
$Speedup(p) = T(1) / T(p)$

Measures how much faster the parallel execution is compared to the serial version.

### Efficiency
$Efficiency(p) = Speedup(p) / p$

Measures the individual utilization of each allocated processor.

---

# 6. Results Table

| Threads/Processes | Time (s) | Speedup | Efficiency |
| :--- | :--- | :--- | :--- |
| 1 | 19.02 | 1.00 | 1.00 |
| 2 | 22.16 | 0.86 | 0.43 |
| 4 | 14.46 | 1.32 | 0.33 |
| 8 | 10.21 | 1.86 | 0.23 |
| 12 | 9.07 | 2.10 | 0.17 |

---

# 7. Execution Time Graph

![Execution Time Graph](graficos/tempo_execucao.png)

*(Trend: Sharp drop from 1 to 12 processes, except for the anomaly at 2 processes).*

---

# 8. Speedup Graph

![Speedup Graph](graficos/speedup.png)

*(Trend: Growth below the ideal 45-degree line).*

---

# 9. Efficiency Graph

![Efficiency Graph](graficos/eficiencia.png)

*(Trend: Downward curve as the number of processes increases).*

---

# 10. Results Analysis

* **Was the speedup close to ideal?** No. For 12 processes, the ideal speedup would be 12x, but the obtained value was 2.10x.
* **Did the application show scalability?** Yes, the time continued to decrease up to 12 processes, though sublinearly.
* **Parallelization Overhead:** The 2-process case (22.16s) was slower than the serial version (19.02s). This is due to communication costs: the time required for MPI to manage processes and for `comm.gather()` to collect millions of results in memory was greater than the benefit of distributed processing.
* **Bottlenecks:** The main bottleneck identified is data centralization at the master process (Rank 0). Sending 12.4 million dictionaries over the network/memory generates severe memory contention and communication latency.
* **Hardware Capacity:** Since the machine has 6 physical cores, running 12 processes utilizes *Hyperthreading* (logical kernels). This increases competition for cache and memory resources, degrading efficiency.

---

# 11. Conclusion

The experiment demonstrates that MPI parallelism is effective in reducing the total processing time for large volumes of data (~52% reduction between serial and 12 processes).

However, for this specific algorithm, the synchronization and data transfer costs are very high. The best scenario in terms of absolute time was with 12 processes, but the best cost-benefit ratio (efficiency) occurred with 4 processes.

As an improvement, it is suggested that each process performs local filtering (e.g., returning only the top 20 similarities) before sending the data to the master process, drastically reducing the volume of data transferred and increasing speedup.
