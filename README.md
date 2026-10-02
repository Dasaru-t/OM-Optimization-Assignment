# Capacitated Vehicle Routing Problem (CVRP) - Zomato Delivery Optimization

This repository contains the optimization pipeline for solving a real-world Capacitated Vehicle Routing Problem (CVRP) applied to food delivery logistics. This project is submitted as the programming assignment for the Optimization Methods module. 

## Problem Context
Efficient delivery routing is a critical operational challenge in urban logistics. This project models a delivery network where a central depot (restaurant) must dispatch drivers to fulfill multiple customer orders without exceeding vehicle carrying capacities. The objective is to minimize the total travel distance, thereby reducing operational costs and delivery times[cite: 21].

## Dataset
Data is utilized exclusively to instantiate and validate the optimization model, rather than for predictive model training[cite: 21]. 
* **Source:** Zomato Food Delivery Insight Data (Kaggle)[cite: 23].
* **Usage:** Latitude and longitude coordinates of restaurants and delivery locations are extracted to compute a real-world geographic distance matrix[cite: 23]. 
* **Scope:** The dataset is filtered to specific city zones to test small-scale exact solutions and large-scale heuristic scalability[cite: 23].

## Optimization Methods
To evaluate performance trade-offs, this project implements two distinct mathematical approaches[cite: 21]:

1. **Exact Method (Integer Linear Programming):** 
   * Formulated using `PuLP` / `OR-Tools`[cite: 21].
   * Solves small subsets (e.g., 10-15 nodes) to find the guaranteed mathematical optimum[cite: 23]. 
   * Strict constraints include vehicle capacity limits, visiting each node exactly once, and sub-tour elimination[cite: 21].
2. **Heuristic Method (Genetic Algorithm):** 
   * Implemented using `DEAP` / `PyGAD`[cite: 21].
   * Evaluated on larger networks (50+ nodes) where exact methods scale exponentially, providing a near-optimal solution in a fraction of the runtime[cite: 21, 23].

## Comparative Evaluation
The two implementations are benchmarked against each other based on:
* **Solution Quality:** The heuristic's percentage gap from the exact method's known optimum on small subsets[cite: 21, 23].
* **Runtime & Scalability:** Computational performance as the number of delivery nodes increases[cite: 21, 23].
* **Feasibility:** Constraint satisfaction in real-world staging constraints[cite: 21].

## Repository Structure
```text
OM-Optimization-Assignment/
│
├── data/                  # Raw and processed Zomato datasets
├── notebooks/             # Jupyter notebooks for data staging, ILP, and GA implementation
├── src/                   # Python scripts for distance matrix calculations and algorithms
├── outputs/               # Generated route maps, performance plots, and logs
├── members.txt            # Student IDs and contribution breakdown
├── submission.txt         # Links to Kaggle dataset, GitHub repo, and YouTube presentation
├── report.pdf             # Final mathematical formulation and evaluation discussion
├── code_appendix.txt      # Plain text export of core algorithm code
├── requirements.txt       # Python dependencies (pulp, deap, folium, etc.)
└── README.md              # Project overview and instructions
