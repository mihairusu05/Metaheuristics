
1. Problem Definition

The Travelling Salesman Problem (TSP) is a classic combinatorial optimization problem. Given a set of n cities and the distances between every pair of cities, the goal is to find the shortest possible route that visits each city exactly once and returns to the starting city. The problem is NP-hard, meaning exact solutions become computationally infeasible as n grows.

A solution is represented as a permutation of city indices, where each consecutive pair represents a directed edge in the tour. The total tour length is the sum of euclidean distances between consecutive cities plus the return edge from the last city back to the first.

Two problem instances were used:
- berlin52: 52 cities in Berlin, known optimal tour length = 7542
- rw1621: 1621 cities in Rwanda, known optimal tour length = 26051


2. Algorithm

The method implemented is Tabu Search (TS) with a 2-opt neighborhood. Tabu Search is a metaheuristic local search algorithm designed to escape local optima by maintaining a memory structure (the tabu list) that forbids recently made moves, forcing the search to explore new areas of the solution space.

Steps of the algorithm:

1. Generate an initial solution using the greedy nearest-neighbor heuristic.
2. Initialize an empty tabu list M with fixed maximum size (tabu tenure).
3. Repeat for a set number of iterations:
   a. Identify a set T of candidate 2-opt moves (either all pairs or a random sample).
   b. Select the best admissible move from T: a move is admissible if it is not on the
      tabu list, OR if it satisfies the aspiration criterion (produces a tour shorter
      than the global best).
   c. Apply the selected 2-opt move to obtain the new current solution.
   d. Update the tabu list by adding the applied move.
   e. Update the local best if the new solution is better.
4. After all iterations, update the global best if the local best improves it.
5. Repeat steps 2-4 for max_tries independent restarts.
6. Return the global best tour length found.

The 2-opt move consists of selecting two indices i and j in the tour and reversing the
segment between them. This effectively removes two crossing edges and reconnects them
without crossing, which is the primary source of improvement in TSP tours.

Aspiration criterion: if a move is tabu but leads to a solution better than the global
best found so far, the tabu status is overridden and the move is accepted. This prevents
the tabu list from blocking genuinely superior solutions.

3. Parameter Settings

Tabu tenure: set to 3*n as recommended in the course slides, where n is the number of
cities. For berlin52 this gives tenure=156, for rw1621 this gives tenure=4863.

Sample size: for berlin52 a sample of 200 pairs was used out of 1326 possible pairs. 
For rw1621 a sample of 5000 pairs was used out of 1,313,010 possible pairs .

Max tries: 4 independent restarts for all experiments, each starting from a fresh greedy or random one
solution with a different random starting city.


4. Results

For belin52:

The best result achieved was 7544.37, within 0.03% of the known optimal of 7542.
This result was found by both initialization strategies at sufficient iteration counts.

Instance: rw1621 (n=1621, known optimal = 26051)

Best result achieved: approximately 32106 (iterations=10, sample_size=5000, max_tries=4).
This is approximately 23% above the known optimal of 26051.


5. Discussion

For berlin52 the Tabu Search implementation achieves near-optimal results (7544 vs
optimal 7542) at higher iteration counts, demonstrating that the algorithm is working
correctly. The 2-opt neighborhood is well-suited to TSP as it directly removes crossing
paths, which are the primary source of suboptimality in TSP tours.

The behavior observed in the berlin52 runs also confirms that Tabu Search correctly
escapes local optima. In several runs the current solution was seen to worsen temporarily
(moving to a longer tour) before finding a shorter one in subsequent iterations -- this
is the key mechanism that distinguishes Tabu Search from pure hill-climbing.

For rw1621 the result of 32106 is significantly above the optimal of 26051 due to two
constraints. First, the neighborhood sample of 5000 out of 1.3 million pairs means that
only 0.4% of possible improvements are considered per iteration. Second, with only 10
iterations per try the search has very limited time to improve from the greedy starting
point.

The fundamental bottleneck for rw1621 is computational. A full neighborhood scan would
require evaluating 1,313,010 pairs per iteration, each involving a tour length computation
over 1621 cities, resulting in approximately 2.1 billion operations per iteration in pure
Python.
