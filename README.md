This project implements a **deterministic, feasible-at-all-times solver** for the **multi-dimensional, multi-knapsack problem (MKP)**.  
It combines a strong **greedy initialization** with a **local improvement / repair phase** based on slack-aware and normalized scoring.

---
### Features

| Component                 | Description                                                        |
| ------------------------- | ------------------------------------------------------------------ |
| **Feasible at all times** | Every solution satisfies capacity and exclusivity constraints      |
| **Greedy initialization** | Fast seeding using normalized value-to-weight ratio                |
| **Slack-aware repair**    | Dynamically balances exploration across multiple dimensions        |
| **Local improvement**     | Single replacement and 2-item “kickset” operators to free capacity |
| **Metrics & bounds**      | evaluation of utilization, value, heuristic bound, and runtime     |

---
### Problem Definition

Given:
- $K$ knapsacks
- $N$ items
- $D$ resource dimensions
- Values $v_i$ and weights $w_{i,d}$
- Capacity constraints $C_d$​
    
Find:
$\text{maximize } \sum_i v_i x_i$

subject to:
$\sum_i w_{i,d} x_{k,i} \le C_d, \quad x_{k,i} \in \{0,1\}, \quad \sum_k x_{k,i} \le 1$

---
### Main Algorithm 
#### 1. **Global Scoring**

Each item receives a **normalized global score** based on all dimensions:

$s_i = \frac{v_i}{1 + \sum_d \frac{w_{i,d}}{C_d}}$​​

#### 2. **Slack-Aware Bin Selection**

For each item, the solver picks the bin where it achieves the highest **slack-aware score**:

$s_{i,k} = \frac{v_i}{1 + \sum_d \frac{w_{i,d}}{\text{remaining}_{k,d}}}$

This dynamically prioritizes items that fit well given current remaining capacities.
#### 3. **Local Improvement Phase**

Once the greedy fill is complete:

- Try to **insert unassigned items** via:
    - direct fit
    - **1-for-1 replacement** (drop the weakest item)
    - **2-item kickset** (drop two poor items to admit one strong item)
        
- Continue until no further improvement occurs.
    
This stage reclaims space from over-weighted, low-value items and increases total value while preserving feasibility.

---
### Metrics Reported

| Metric                     | Description                                           |
| -------------------------- | ----------------------------------------------------- |
| **Feasibility**            | True/False check for all constraints                  |
| **Total value**            | Objective function value                              |
| **Items used**             | Number of assigned items                              |
| **Utilization (max/mean)** | Resource usage balance across bins and dimensions     |
| **Heuristic bound ratio**  | Approximate quality ratio vs. pooled fractional bound |
| **Runtime**                | Execution time in milliseconds                        |

---
### Results
| Problem | Items | Knapsacks | Dimensions | Value    | Time   | Feasible |
| ------- | ----- | --------- | ---------- | -------- | ------ | -------- |
| 1       | 20    | 2         | 2          | 1065     | <1 ms  | Yes      |
| 2       | 100   | 10        | 10         | ~39.5 k  | ~20 ms | Yes      |
| 3       | 5000  | 100       | 100        | ~1.841 M | ~7 s   | Yes      |

---
### Intuition

- **Normalized scoring** gives global ranking (what to try first).
- **Slack-aware scoring** decides _where_ to place each item.
- **Local improvement** refines this by freeing space for missed high-value items.
- You can easily extend this framework via **hybridization** with Tabu or Simulated Annealing for exploration
---
### Citations

**Kellerer, H., Pferschy, U., & Pisinger, D. (2004). _Knapsack Problems_. Springer.** - Density and pseudo-utility scores; defines the basis for all greedy MKP solvers.

**Eiben, A. E., & Smith, J. E. (2015). _Introduction to Evolutionary Computing_. Springer. Ch:3**  - Describes greedy initialization and adaptive local search 
