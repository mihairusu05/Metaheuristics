1. Problem Definition

The knapsack problem is a classic combinatorial optimization problem. Given n objects, each
with a value v and a weight w, and a knapsack with a maximum weight capacity W, the goal
is to select a subset of objects such that the total value is maximized without exceeding the
weight limit W. Each object is either selected (xi = 1) or not selected (xi = 0).

Two problem instances were used:
- knapsack-20.txt: 20 objects, maximum weight capacity of 524, optimal value = 726
- knapsack-200.txt: 200 objects, maximum weight capacity of 112648, optimal value aprox 134038(best found at least)


2. Algorithm

The method implemented is Simulated Annealing (SA). SA is a stochastic metaheuristic
inspired by the annealing process in metallurgy. It starts from a random solution and
iteratively explores the neighborhood by flipping a single random bit. Unlike hill-climbing,
SA can accept worse solutions with a probability that decreases over time, allowing it to
escape local optima.

Steps of the algorithm (following the course pseudocode):

1. t = 0, initialize T = T_max
2. Select a current point c at random (resample until feasible)
3. Repeat (inner loop, iterations_per_temp times):
   a. Select a new point x from the neighborhood of c (flip one random bit)
   b. If x is infeasible (exceeds weight capacity), skip and try again
   c. If eval(c) < eval(x): c <- x  (always accept improvement)
   d. Else if random[0,1) < e^((eval(x)-eval(c))/T): c <- x  (accept worse with probability)
   e. Update best if c is better than current best
4. T <- g(T, t), t <- t + 1
5. Repeat steps 3-4 until T < T_min (halting condition)
6. Return best solution found

The neighborhood is defined as all solutions reachable by flipping exactly one bit. For a
solution of size n this gives n possible neighbors per evaluation step.

Infeasible solutions (weight exceeding W) are handled by keeping the current solution c
always feasible. The initial solution is resampled until a feasible starting point is found.
Infeasible neighbors are skipped and the current solution remains unchanged, ensuring the
search always operates from a valid state.

The overflow guard MAX_EXP = log(sys.float_info.max) aprox 709.78 is used to prevent
math.exp from overflowing when the exponent is very large.

Three cooling schedules were tested:

Geometric:    T(k+1) = alpha * T(k)
Logarithmic:  T(k+1) = T(k) / log(k+1)
Linear:       T(k+1) = T(k) / k

where T(k) is the temperature at step k. The geometric schedule cools slowly and uniformly,
while logarithmic and linear collapse much faster due to their cumulative division structure.


3. Parameter Settings

Parameters tested for knapsack-20:

- iterations_per_temp: [5, 10, 20, 40]
- T_max: [100, 1000, 10000, 100000]
- T_min: [0.1, 0.01, 0.001]
- alpha: [0.9, 0.95, 0.99, 0.999]
- cooling: geometric, logarithmic, linear

Parameters tested for knapsack-200:

- iterations_per_temp: [50, 100, 200, 300]
- T_max: [100, 1000, 10000]
- T_min: [1, 0.1, 0.01]
- alpha: [0.9, 0.95, 0.99]
- cooling: geometric, logarithmic, linear

The number of neighbors checked per run varies significantly by cooling schedule.
4. Results

Instance: knapsack-20 (n=20, optimal=726)

Geometric cooling:
The geometric schedule is clearly the most effective. With alpha=0.999 and sufficient
iterations_per_temp, the algorithm consistently reaches the optimal value of 726 across
a wide range of T_max and T_min values.

-alpha, T_max, T_min represent the nr of temperatures checked where alpha holds most importance
-nr of iterations also important for nr of neighbours from n+ proves very good results

Best result: 726 (optimal), achieved by multiple parameter combinations, most reliably
with T_max=1000-10000, T_min=0.01-0.001, alpha=0.999, iterations_per_temp=20-40.

Logarithmic cooling:
The logarithmic schedule runs for very few steps (50-920 neighbours checked) due to the
cumulative division collapsing T rapidly. Results are consistently in the 500-726 range
with high variance. The best result of 726 was achieved occasionally with
iterations_per_temp=20, T_max=100, T_min=0.001. However results are unreliable and
highly dependent on the random starting solution. The schedule provides insufficient
exploration time for consistent convergence.


Linear cooling:
The linear schedule is the most aggressive, collapsing in 35-480 total neighbour
evaluations across all tested parameters. This is far too few for meaningful search on
knapsack-20. The best result of 726 was achieved twice:
  - iterations=40, T_max=1000, T_min=0.001, alpha=0.999: result=726 (400 neighbours)
  - iterations=40, T_max=100000, T_min=0.1, alpha=0.999: result=726 (400 neighbours)
Both cases required iterations_per_temp=40 (2x problem size) to compensate for the very
few temperature steps. Results are generally in the 500-690 range with high variance.

Instance: knapsack-200 (n=200, optimal ~134038)

Geometric cooling:
The geometric schedule again outperforms the other two schedules. Results consistently
reach 133000-134343 with good parameter settings. Key observations:

- T_max=10000 with alpha=0.99 consistently produces the best results (133500-134836).
- iterations_per_temp=300 with T_max=10000, alpha=0.99 produced the best observed
  result of 134836, exceeding the previously assumed optimal of 132910 found by random
  search in Lab 1.
- Higher T_max allows the algorithm to accept worse moves early and explore widely
  before converging.

Best result: 134836 (iterations=300, T_max=10000, T_min=0.1, alpha=0.99).

Logarithmic cooling:
Runs for 500-5700 total neighbour evaluations. Results are in the range 130419-133616,
generally lower quality than geometric. The best result was 133616 with
iterations=200, T_max=100, T_min=0.1. High variance between runs at the same parameters
confirms that the short search duration limits reliability.

Best result: 133616.

Linear cooling:
Runs for only 250-3000 total neighbour evaluations. Despite the very short runs, results
are reasonable (130940-133667) because each of the few temperature levels has many inner
iterations. The best result was 133667 with iterations=300, T_max=1000, T_min=0.01.

Best result: 133667.


5. Discussion

Geometric vs Logarithmic vs Linear:

The geometric schedule is clearly superior for both instances. Its controlled, uniform
cooling gives the algorithm sufficient time at each temperature level to explore the
neighborhood and find improving moves. The number of neighbours checked is orders of
magnitude higher than logarithmic or linear, which directly translates to better solution
quality.

The logarithmic and linear schedules collapse temperature so fast due to their cumulative
division structure that meaningful search is barely possible. For knapsack-20 with only
20 bits, occasional lucky random starts can still find the optimal, but this is not
reliable. For knapsack-200 the results are consistently worse than geometric.

Key parameter insights:

- alpha (geometric only): the most impactful parameter. Values close to 1 (0.999) allow
  slow cooling and thorough exploration. Values of 0.9 cool 10x faster and miss many
  good solutions.
- iterations_per_temp: should be at least equal to problem size n. For knapsack-20,
  iterations=20; for knapsack-200, iterations=200 is a natural baseline.

Comparison with previous labs:

SA with geometric cooling outperforms both random search, tabu search and SAHC from Labs 1 and 2.
For knapsack-200, the best SA result (134836) exceeds the previously best known result
of 132910 from random search, demonstrating SA's ability to find superior solutions
through guided probabilistic exploration.
