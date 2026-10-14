# Individual Contributions & Task Breakdown

**Project Title:** Capacitated Vehicle Routing Problem (CVRP) for Urban Food Delivery Optimization  
**Implementation File:** `CVRP_Food_Delivery_Optimization_executed.ipynb`  

This project was developed collaboratively in two structured phases, dividing responsibilities across problem modeling, exact mathematical programming, population-based evolutionary search, trajectory-based local search, and empirical scalability benchmarking.

---

## Summary of Rubric Alignment

| Rubric Criterion | Primary Owner | Notebook Section / Deliverable |
| :--- | :--- | :--- |
| **1. Problem Selection** | Member 1 (Dasaru) & Member 2 (Kosala) | Capacitated Vehicle Routing Problem (CVRP) on real-world Zomato food delivery data. |
| **2. Problem & Data Description** | Member 1 (Dasaru) | Section 1 & 2: Restaurant depot selection, order filtering, demand scaling, deterministic `hashlib.md5` geocoding, and vectorized Haversine distance matrix. |
| **3. Mathematical Formulation** | Member 1 (Dasaru) | Section 3: Two-index Miller-Tucker-Zemlin (MTZ) Integer Linear Program (ILP) with fleet-size lower-bound valid inequality ($\sum x_{0j} \ge \lceil \sum d_i / Q \rceil$). |
| **4. Optimization Methods (Exact + Population)** | Member 1 (Dasaru) | Section 3 & 4: Exact MILP solver via PuLP/CBC (with log-parsed MIP lower bounds) and Population Metaheuristic via DEAP Genetic Algorithm (`eaSimple`). |
| **4. Optimization Methods (Constructive + Trajectory)** | Member 2 (Kosala) | Section 5: Clarke-Wright Savings constructive heuristic ($O(n^2 \log n)$) and Trajectory Metaheuristic via Simulated Annealing with 10-nearest-neighbor granular moves. |
| **5. Implementation & Verification** | Member 1 & Member 2 | Section 2 & 6: Modular instance generator (`make_instance`), independent solution feasibility verifier (`check_solution`), and automated multi-seed runner. |
| **6. Comparative Results** | Member 2 (Kosala) | Section 6 & 7: 175-run scalability study across $n \in \{8, 10, 12, 15, 20, 30, 50, 100\}$ over 10 random seeds, certified MIP gap tables, and Mann-Whitney U statistical tests. |
| **7. Discussion & Visual Analysis** | Member 1 & Member 2 | Section 4, 5 & 7: Analysis of GA generational replacement vs. elitism, tight-capacity neighborhood granularity, runtime/quality scaling plots, boxplots, and convergence curves. |

---

## Detailed Task Breakdown by Member

### Member 1: Dasaru Thamaranga
**Focus: Problem Modeling, Data Engineering, Exact Optimization (ILP) & Genetic Algorithm**

1. **Problem Formulation & Dataset Engineering (Sections 1–2):**
   * Formulated the urban food delivery dispatch problem as a symmetric Capacitated Vehicle Routing Problem (CVRP) with uniform rider box capacity ($Q = 120$ units).
   * Built the data staging pipeline over `restaurants.csv`, `customers.csv`, and `orders.csv`: identified the highest-volume active restaurant as the central depot (`Classic Cafe (Desserts)`, ID 106), filtered cancelled orders, aggregated unique customer nodes (337 available customers), and derived integer order demands (`round(total_amount / 100)`).
   * Implemented deterministic `hashlib.md5` pseudo-geocoding around central Chennai (`stable_coords`) to guarantee cross-platform coordinate reproducibility, alongside the vectorized great-circle Haversine distance matrix (`haversine_matrix`).

2. **Exact Method — Strengthened MTZ Integer Linear Program (Section 3):**
   * Formulated the two-index Miller-Tucker-Zemlin (MTZ) binary integer linear program in `PuLP` using the COIN-OR CBC solver (`solve_ilp`) with flow conservation, degree constraints, and continuous subtour elimination variables ($u_i$).
   * Strengthened the LP relaxation by introducing the valid fleet-size lower-bound cut $\sum_{j \in V'} x_{0j} \ge \lceil \sum_{i \in V'} d_i / Q \rceil$, which reduced the $n=15$ solve time from timing out at 300 s (28.1% gap) to proving global optimality (`104.225 km`, 0.0% gap) in **175.97 s**.
   * Implemented exact solver status verification (`m.sol_status == pulp.LpSolutionOptimal`) and regex log parsing (`cbc_n*.log`) to extract certified dual lower bounds when CBC hits the 300 s time limit on larger instances ($n = 20, 30$).

3. **Metaheuristic 1 — Population-Based Genetic Algorithm in DEAP (Section 4):**
   * Designed and implemented the Genetic Algorithm (`run_ga` and `ga_decode`) using a giant-tour permutation chromosome ($0 \dots n-1$) decoded into capacity-feasible vehicle routes via greedy sequential splitting.
   * Configured evolutionary operators in `DEAP`: Tournament Selection (`tournsize=3`), Ordered Crossover (`cxOrdered`, `cxpb=0.8`), Shuffle-Index Mutation (`mutShuffleIndexes`, `indpb=0.05`, `mutpb=0.2`), Population Size = 200, and Generations = 500 with `HallOfFame(1)` tracking.
   * Analyzed the population convergence behavior of `eaSimple`, documenting how full generational replacement without elite re-insertion causes population fitness fluctuations and why greedy permutation splitting struggles as the search space scales to $n = 50$ and $100$.

---

### Member 2: Kosala
**Focus: Constructive Heuristic, Granular Simulated Annealing, Scalability Benchmarking & Statistical Evaluation**

1. **Constructive Heuristic — Clarke-Wright Savings Algorithm (Section 5.1):**
   * Implemented the classical deterministic Clarke-Wright parallel savings heuristic (`clarke_wright` and `run_cw`), computing pairwise merge savings $s_{ij} = c_{0i} + c_{0j} - c_{ij}$ and greedily merging route endpoints subject to vehicle capacity $Q$.
   * Established a sub-millisecond industrial baseline ($< 0.01\text{ s}$ even at $n = 100$) that achieves optimal routing at $n = 8$ (`73.74 km`) and within 3.53% of the best-known solution at $n = 100$ (`518.80 km`).

2. **Metaheuristic 2 — Trajectory-Based Simulated Annealing with Granular Neighborhoods (Section 5.2):**
   * Designed and implemented the Simulated Annealing trajectory metaheuristic (`simulated_annealing` and `run_cw_sa`) warm-started from the Clarke-Wright solution.
   * Engineered four capacity-aware local search move operators with $O(\text{route length})$ incremental delta-cost evaluation: intra/inter-route **Relocate** (35% probability), inter-route **Swap** (20%), inter-route **2-opt\*** tail exchange (25%), and intra-route **2-opt** segment reversal (20%).
   * Implemented a **10-nearest-neighbor granular candidate list** (`k_near=10`) to overcome tight vehicle packing (113–120 / 120 units), paired with automatic initial temperature calibration ($T_0$ set for 30% initial uphill acceptance) and geometric cooling over $\max(100,000,\; 5,000n)$ iterations.

3. **Independent Verification, Scalability Study & Statistical Analysis (Sections 2.1, 6 & 7):**
   * Implemented the independent route cost evaluator and feasibility assertion checker (`check_solution`) verifying customer coverage, depot return, and capacity limits across all 175 experimental runs.
   * Executed the full scalability and robustness benchmark across 8 problem sizes ($n \in \{8, 10, 12, 15, 20, 30, 50, 100\}$) with 10 random seeds per stochastic method, exporting raw and aggregated metrics to `results/all_runs.csv` and `results/summary.csv`.
   * Performed two-sided non-parametric **Mann-Whitney U tests** (`results/ga_vs_sa_test.csv`), proving statistically significant superiority ($p < 0.05$) of CW+SA over GA for all $n \ge 12$ (improving tour distance by 21.9% at $n=50$ and 36.6% at $n=100$ while running 5× faster).
   * Generated the comparative visualization suite (`scalability.png`, `robustness_boxplots.png`, `convergence.png`, and `routes_n50.png`).
