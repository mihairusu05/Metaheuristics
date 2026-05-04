1. Problem Definition

The knapsack problem is a classic combinatorial optimization problem. Given n objects, each with a value v and a weight w, and a knapsack with a maximum weight capacity W, the goal is to select a subset of objects such that the total value is maximized without exceeding the weight limit W. Each object is either selected (xi = 1) or not selected (xi = 0). The problem is NP-hard, meaning exact solutions become computationally infeasible as n grows, which motivates the use of heuristic and metaheuristic approaches such as hill-climbing.

Two problem instances were used in the experiments:
- knapsack-20.txt: 20 objects, maximum weight capacity of 524
- knapsack-200.txt: 200 objects, maximum weight capacity of 112648


2. Algorithm

The method implemented is Steepest Ascent Hill-Climbing (SAHC). Unlike simple random search, SAHC uses a guided local search mechanism: starting from a random solution, it systematically examines all possible single-bit changes and moves to the best improvement found. This process repeats until no improvement is possible, at which point a new random solution is generated and the search restarts. The overall best solution found across all restarts is tracked and returned at the end.

Steps of the algorithm :

1. Choose a binary string at random. Call this string current-hilltop.
2. Going from left to right, systematically flip each bit in the string, one at a time, recording the fitness of each resulting one-bit mutant.
3. If any of the resulting one-bit mutants give a fitness increase, set current-hilltop to the one-bit mutant giving the highest fitness increase. In case of ties among mutants with equal best fitness, one is chosen at random.
4. If there is no fitness increase among any mutant, save current-hilltop as a candidate for the global best and go to step 1 with a new random solution. Otherwise, go to step 2 with the new current-hilltop.
5. When a set number of function evaluations has been performed (each bit flip in step 2 counts as one function evaluation), return the highest hilltop found across all restarts.

The fitness function returns the total value of selected items if the total weight is within the capacity W, and returns -1 for infeasible solutions that exceed the weight limit.


3. Implementation Details

- load_data(file_name): reads the input file and returns the number of objects, list of (value, weight) pairs, and maximum weight.
- compute_value_for_solution(solution, max_weight, weights_values): evaluates a candidate solution, returning its total value or -1 if infeasible.
- steepest_ascent_hill_climbing(filename, n_function_evaluations): implements the SAHC algorithm. Generates an initial random solution, performs full scans of all bit-flip mutants, updates the current hilltop to the best strictly improving mutant found in each scan (with random tie-breaking among equally best mutants), triggers a random restart when no improvement is found, and returns the globally best solution and its fitness once the evaluation budget is exhausted.
- generate_report(input_file, output_file, n_function_evaluations): runs the algorithm for a range of n values, saves results to a text file, and makes plots.

4. Parameter Settings

The parameter tested is n, the total number of fitness function evaluations, meaning the total budget of bit flips allowed across all restarts. Since one full scan of a solution of length l costs l function evaluations, for n=20 each full scan costs 20 evaluations and for n=200 each full scan costs 200 evaluations.

Results were logged every 10 fitness evaluations, recording the best solution found so far as both the binary vector and its fitness value.

Instance n=20:
- n values tested: 50, 100, 1000
- Each scan costs 20 function evaluations
- Capacity W: 524

Instance n=200:
- n values tested: 10 to 10000 in steps of 10
- Each scan costs 200 function evaluations
- Capacity W: 112648


5. Results

All results were saved in report_20.txt and report_200.txt. Charts were generated showing best fitness found vs number of fitness evaluations (see images).

Instance: knapsack-20.txt (n=20, W=524)

Run with n=50 (at most 2 full scans):
Results show high variance and several infeasible solutions (-1) appearing in the log, which is expected at very low budgets. The best values seen ranged from -1 up to around 689. With only 50 evaluations and a problem size of 20, SAHC can complete at most 2 full scans before the budget runs out, severely limiting exploration.

Run with n=100 (at most 5 full scans):
Results improve noticeably. Infeasible solutions still appear early due to random restarts landing on infeasible starting points, but by around 60-70 evaluations the algorithm consistently finds values in the 700 range, with the best value reaching 726 at around evaluation 310 in one of the restarts.

Run with n=1000 (at most 50 full scans):
This is the most informative setting. The best value found across the full run was 726, first reached around evaluation 370 and then rediscovered multiple times, confirming it is a strong and likely near-optimal local hilltop. Values across different restarts fluctuate between roughly 620 and 726. The chart shows a noisy but upward-trending curve that flattens after roughly 200-400 evaluations.

Instance: knapsack-200.txt (n=200, W=112648)

With n varying from 10 to 10000 in steps of 10:
The chart shows a distinctive pattern. At very low n values (below roughly 400-600 evaluations), the best fitness can drop to -1 , reflecting that with fewer than 2-3 full scans (each costing 200 evaluations) the algorithm has almost no opportunity to hill-climb from its initial random solution. Once n exceeds roughly 600-800 evaluations, the best fitness rapidly stabilizes in the range of 130000 to 133000. Beyond that point the curve flattens almost completely, indicating that SAHC quickly reaches local optima for this instance and additional evaluations do not help significantly. The best value found was approximately 133000.


6. Discussion

SAHC is a significant improvement over random search for the knapsack problem. Whereas random search simply samples random solutions with no feedback, SAHC actively moves toward better solutions by exploiting the neighborhood structure of the search space through single-bit mutations. This guided mechanism allows it to find high-quality solutions much faster per evaluation.

For n=20, SAHC consistently finds solutions with values in the 680-726 range once given a sufficient evaluation budget of n=100 or more. The best value of 726 appears repeatedly across different restarts, suggesting it is the maximum.

For n=200, the algorithm behavior is dominated by the cost of a full scan. At 200 evaluations per scan, even a budget of n=1000 only allows 5 complete scans. Despite this, SAHC quickly converges to high-fitness solutions in the 130000-133000 range. The flat plateau visible in the chart confirms that local optima are reached early and random restarts do not escape them reliably within the tested budgets.

The main limitation of SAHC is susceptibility to local optima. Once no single-bit flip improves the current solution, the algorithm restarts from scratch with no memory or guidance from previous searches.

