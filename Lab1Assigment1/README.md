1. Problem Definition

The knapsack problem is a classic combinatorial optimization problem. Given n objects, each with a value v and a weight w, and a knapsack with a maximum weight capacity W, the goal is to select a subset of objects such that the total value is maximized without exceeding the weight limit W. Each object is either selected (xi = 1) or not selected (xi = 0). The problem is NP-hard, meaning exact solutions become computationally infeasible as n grows, which motivates the use of heuristic and metaheuristic approaches.

Two problem instances were used in the experiments:
- knapsack-20.txt: 20 objects, maximum weight capacity of 524
- knapsack-200.txt: 200 objects, maximum weight capacity of 112648

2. Algorithm

The method implemented is Random Search. The algorithm generates k random candidate solutions and returns the one with the highest value that satisfies the weight constraint. If a solution exceeds the maximum weight, it is assigned a value of -1 and is effectively discarded from consideration.

Steps of the algorithm:

1. Load the problem instance from the input file (number of objects, value-weight pairs, and max weight capacity).
2. Generate k random binary strings of length n, where each position is independently set to 0 or 1. Each string represents a candidate solution.
3. For each candidate solution, compute the total value and total weight of the selected items.
4. If the total weight exceeds the capacity W, assign the solution a value of -1 (infeasible).
5. Return the maximum value found among all k candidate solutions.

The random solutions are stored in a set to avoid duplicates. The value of k is varied across runs to observe how the number of random samples affects the quality of the best solution found.

3. Results

All results were saved in the output files report_20.txt and report_200.txt.

Notable observations for n=20:
- At k=2, 3 out of 5 runs produced infeasible solutions (-1), meaning all 2 randomly generated solutions exceeded the weight limit. This is expected behavior for very small k with a relatively tight weight constraint.
- As k increases, the best found value tends to increase or stabilize, since more solutions are sampled and there is a higher probability of finding a good feasible one.
- The highest value found across all runs was 632 (Run 4, k=12).
- Results show high variance between runs at the same k value, which is characteristic of pure random search.

Notable observations for n=200:
- For n=200, the weight capacity is much more generous relative to item weights, so almost no solutions are infeasible (-1). This explains why even at k=2 relatively high values are achieved.
- The best value found in Run 1 was 132598 (k=126) and in Run 2 was 132910 (k=156).
- Unlike the n=20 case, the values for n=200 are relatively stable across different k values, typically in the range 130000 to 133000. This suggests that the search space for n=200 is dense with decent solutions, but finding the optimal is still not guaranteed by random search.
- There is no strong monotonic improvement trend with increasing k, which is also characteristic of random search: adding more random samples improves the chances of finding a better solution only gradually.


4. Discussion

Random search is a simple baseline metaheuristic that explores the solution space without any guided mechanism. Its main advantage is simplicity and ease of implementation. However, it has significant limitations:

- For small n (n=20), the weight constraint eliminates many random solutions, especially at low k, leading to frequent infeasible results. Increasing k helps but does not guarantee optimality.
- For large n (n=200), the constraint is easier to satisfy, but the search space is 2^200, making it practically impossible to find the optimal solution by random sampling.
- The quality of the best solution improves with k, but only marginally beyond a certain point. This demonstrates the fundamental limitation of random search: it cannot exploit structure in the problem.
- The high variance in results across runs at the same k value confirms that random search is unreliable and inconsistent, especially for small k.

5. Parameter Settings Summary

Instance n=20:
- k values tested: 2, 4, 6, 8, 10, 12, 14, 16, 18
- Number of independent runs: 5
- Capacity W: 524

Instance n=200:
- k values tested: 2, 4, 6, ..., 198 (step 2, 99 values total)
- Number of independent runs: 2 (shown in output file)
- Capacity W: 112648
