1. Problem Definition

The Travelling Salesman Problem (TSP) is a classic combinatorial optimization problem.
Given a set of n cities and the distances between every pair of cities, the goal is to
find the shortest possible route that visits each city exactly once and returns to the
starting city. The problem is NP-hard.

A solution is represented as a permutation of city indices. The total tour length is the
sum of euclidean distances between consecutive cities including the return edge from the
last city back to the first.

Two problem instances were used:
- berlin52: 52 cities in Berlin, known optimal tour length = 7542
- rw1621: 1621 cities in Rwanda, known optimal tour length = 26051


2. Algorithm

The method implemented is an Evolutionary Algorithm (EA) / Genetic Algorithm (GA) for
TSP. Unlike single-solution metaheuristics (SA, Tabu Search), EA maintains a population
of candidate solutions and evolves them over generations through selection, crossover,
and mutation operators.

Steps of the algorithm:

1. Population initialization: half the population is initialized using greedy nearest-
   neighbor tours starting from different cities (i % n), the other half is random
   permutations. This hybrid initialization ensures a good starting region.

2. For each generation:
   a. Evaluate fitness (tour length) for all individuals.
   b. Sort population by fitness (ascending, shorter = better).
   c. Apply elitism: keep the best elite_size individuals unchanged.
   d. Check stagnation: if no improvement for no_improve_limit generations, inject
      20 random individuals to restore diversity.
   e. Fill the rest of the new population by:
      - Selecting two parents via tournament selection (size=3)
      - Applying crossover (order or PMX) with rate 0.7
      - Applying inversion mutation to each child
   f. Replace population with new population.

3. Return the best tour length found.

Crossover operators:

Order Crossover (OX): copies a random subtour from parent1 into the child, then fills
remaining positions with cities from parent2 in the order they appear starting after the
subtour end point. Preserves relative ordering of cities.

Partially Mapped Crossover (PMX): copies a random segment from parent1, then uses a
mapping defined by the segment to place remaining cities from parent2. Preserves absolute
positions of cities within the copied segment.

Mutation operator: inversion mutation — selects two random indices i and j and reverses
the segment between them. This is equivalent to a single 2-opt move and is well-suited
for TSP as it can remove crossing paths.

Selection: tournament selection with size 3 — randomly picks 3 individuals, returns the
one with shortest tour. Applied separately to select each parent.

Elitism: the best elite_size individuals are carried unchanged to the next generation,
guaranteeing the best solution never gets lost.

Stagnation detection: if no_improve_count reaches 200, 20 random tours are injected into
the population to restore genetic diversity and allow the search to escape local optima.


3. Parameter Settings

Parameters tested for berlin52:

- iterations: [100, 500, 1000, 2500]
- initial_population: [20, 50, 100]
- elite_size: [3, 5, 10]
- crossover_type: order, pmx
- crossover_rate: 0.7 (fixed)
- tournament_size: 3 (fixed)
- no_improve_limit: 200 (fixed)

Parameters tested for rw1621:

- iterations: [50, 100, 200]
- initial_population: [10, 20, 30]
- elite_size: [3, 5, 10]
- crossover_type: order, pmx
- crossover_rate: 0.7 (fixed)
- tournament_size: 3 (fixed)
- no_improve_limit: 200 (fixed)


4. Results

Instance: berlin52 (n=52, known optimal=7542)

Order crossover results summary:

The best result of 7544.37 (essentially optimal, within 0.03% of 7542) was achieved.

Key observations for order crossover:
- Population size is the most impactful parameter. population=100 consistently
  outperforms population=20 and population=50 across all iterations and elite sizes.
- With population=100, results improve steadily from 7650 at 100 iterations to 7544
  at 1000+ iterations.

PMX crossover results summary:

The best result of 7544.37 was also achieved by PMX .

Notable: PMX reaches 7544 faster than OX — already at iterations=500 with population=100,
while OX needs iterations=1000. However at iterations=2500 both converge to the same
result, suggesting both operators are equally capable given enough generations.

Overall best for berlin52: 7544.37 (both crossover types, population=100, iterations>=500)

Instance: rw1621 (n=1621, known optimal=26051)

Order crossover results summary:

Results are tightly clustered around 32276.67 (the greedy tour length from starting
city 0) with small improvements for higher iterations and larger populations:

Best results:
  iterations=200, population=20, elite=10: 32129.34
  iterations=100, population=30, elite=3:  32175.76
  iterations=200, population=30, elite=3:  32207.02

The improvement over the greedy baseline (32276.67) is modest at around 0.4-0.5%,
reflecting the difficulty of the instance.

PMX crossover results summary:

Best result:
  iterations=100, population=30, elite=3:  32072.43

PMX achieves a slightly better best result than OX for rw1621, though the difference
is small and results are generally similar across both operators.

Overall best for rw1621: 32072.43 (PMX, population=30, elite=3, iterations=100)


5. Discussion

Comparison of crossover operators:

For berlin52, both OX and PMX achieve the same near-optimal result of 7544 given
sufficient population (100) and iterations (500+). PMX converges slightly faster,
reaching 7544 at iterations=500 vs iterations=1000 for OX. For rw1621, PMX gives a
marginally better best result (32072 vs 32129 for OX). Overall the two operators are
comparable in quality.

Effect of population size:

Population size is the dominant parameter for solution quality. Small populations
(20-50) converge prematurely to local optima due to insufficient genetic diversity,
regardless of how many iterations are run. For berlin52, population=100 is necessary
to reliably reach near-optimal results. For rw1621, even population=30 shows clear
improvement over population=10.

Effect of elitism:

Higher elitism (elite=10) helps at lower iteration counts by preserving good solutions.
At higher iteration counts the difference diminishes as the population naturally converges
to good solutions.

Importance of greedy initialization:

The greedy initialization was the critical design decision. Without it, the population
starts with random tours of length 20000-30000 for berlin52 and 500000+ for rw1621,
requiring far more generations to reach competitive results. With greedy initialization,
berlin52 reaches 7544 by generation 250, demonstrating that starting in a good
neighborhood dramatically reduces the search burden.


The EA matches the best results of Tabu Search and SA for berlin52 and achieves a
slightly better result than Tabu Search for rw1621 (32072 vs 32106). The key advantage
of EA over single-solution methods is the ability to combine good partial solutions
from multiple individuals through crossover, which helps escape local optima that trap
single-solution methods.

Computational note for rw1621:

Each generation for rw1621 requires evaluating tour_length for all individuals, which
loops over 1621 cities per individual. With population=30 and 200 iterations this is
manageable. However the initialization cost (computing greedy tours) is approximately
7 seconds per greedy tour, limiting how many greedy individuals can be included. A full
greedy initialization for all 1621 cities would take approximately 3 hours.
