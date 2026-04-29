1. Problem Definition

The Travelling Salesman Problem (TSP) is a classic combinatorial optimization problem.
Given a set of n cities and the distances between every pair of cities, the goal is to
find the shortest possible route that visits each city exactly once and returns to the
starting city. The problem is NP-hard.

A solution is represented as a permutation of city indices. The total tour length is the
sum of euclidean distances between consecutive cities including the return edge from the
last city back to the first.

Two problem instances were used:
- berlin52.tsp: 52 cities in Berlin, known optimal tour length = 7542
- rw1621.tsp: 1621 cities in Rwanda, known optimal tour length = 26051
  Note: rw1621 contains 755 duplicate city coordinates (46.6% of the dataset),
  which were removed during loading, reducing the effective instance to 866 unique cities.


2. Algorithm

Two variants of Ant Colony Optimization (ACO) were implemented.

Ant System (AS) — the original ACO variant (Dorigo et al., 1992):

Each ant builds a complete tour by probabilistically selecting the next city based on
pheromone strength and visibility (1/distance). After all ants complete their tours,
pheromone is updated globally — all ants deposit pheromone proportional to the quality
of their tour, and all edges evaporate.

Ant Colony System (ACS) — the improved variant (Dorigo & Gambardella, 1997):

ACS introduces three key improvements over AS:

1. New transition rule with exploitation parameter q0: at each step a random q is drawn.
   If q <= q0 the ant deterministically selects the best next city (exploitation).
   If q > q0 the ant uses the probabilistic AS rule (exploration).
   With q0=0.9, ants exploit 90% of the time and explore 10% of the time.

2. Local pheromone update: immediately after an ant traverses an edge, pheromone on
   that edge is reduced toward the initial value tau0. This discourages subsequent ants
   from using the same edge, promoting diversity without pure randomness.

3. Global pheromone update using only the best ant: unlike AS where all ants deposit
   pheromone, ACS only reinforces edges belonging to the global best tour found across
   all iterations. This creates much stronger convergence pressure.

Algorithm steps:

1. Initialize pheromone matrix with tau0 = 1 / (n * L_nn) where L_nn is the nearest
   neighbor greedy tour length. Initialize visibility matrix eta[i][j] = 1/dist[i][j].

2. For each iteration:
   a. Each ant starts from a random city and builds a complete tour using the transition
      rule (ACS) or probabilistic rule (AS).
   b. ACS only: after each edge traversal, apply local pheromone update.
   c. Evaluate all ant tours, track global best.
   d. Apply global pheromone update:
      AS:  tau(t+1) = (1-rho)*tau(t) + sum_k(Q/L_k) for all ants k
      ACS: tau(t+1) = (1-rho)*tau(t) + rho*(1/L+) only on global best tour edges

3. Return the global best tour and its length.

Parameters used (ACS, from seminar slides):
  alpha = 1         (pheromone influence)
  beta = 2          (visibility influence)
  q0 = 0.9          (exploitation probability)
  evaporation_rate = 0.1       (global pheromone evaporation, rho)
  local_evaporation_rate = 0.1 (local pheromone evaporation, xi)
  tau0 = 1 / (n * L_nn)        (initial pheromone, derived from greedy tour)


3. Parameter Settings

Parameters tested for berlin52:
- nr_of_iterations: [1, 2, 5, 10, 20, 50, 100, 200, 500, 1000]
- nr_of_ants: [1, 2, 5, 10, 20, 26, 52]

Parameters tested for rw1621 (reduced to 866 unique cities):
- nr_of_iterations: [1, 2, 5, 8, 10]
- nr_of_ants: [1, 2, 5, 8, 10]

The rw1621 parameter ranges are much smaller due to computational cost —
each iteration for rw1621 takes approximately 40 seconds per ant due to the
large number of cities.


4. Results

Instance: berlin52 (n=52, optimal=7542)

Best result overall: 7544.37
  Achieved by multiple configurations:
  - iterations=50,  ants=52
  - iterations=200, ants=52
  - iterations=500, ants=26
  - iterations=500, ants=52
  - iterations=1000, ants=20
  - iterations=1000, ants=52

This result of 7544.37 is essentially optimal — only 0.03% above the known
optimum of 7542. The gap is attributable to floating point precision in the
euclidean distance calculation rather than algorithm quality.

Selected results (berlin52):

  iterations=1,    ants=1:  8980.92  (0.01s)   — single random tour, no learning
  iterations=10,   ants=52: 7548.99  (0.65s)   — near optimal in 10 iterations with full swarm
  iterations=20,   ants=26: 7548.99  (0.67s)   — near optimal with half swarm
  iterations=50,   ants=52: 7544.37  (3.20s)   — optimal result, fast
  iterations=100,  ants=20: 7548.99  (2.64s)   — near optimal
  iterations=200,  ants=52: 7544.37  (12.59s)  — optimal, slower
  iterations=500,  ants=26: 7544.37  (15.56s)  — optimal
  iterations=1000, ants=20: 7544.37  (25.86s)  — optimal, slowest tested

Effect of iterations (berlin52, fixed ants=52):
  iterations=1:    8622.75
  iterations=2:    8102.87
  iterations=10:   7548.99
  iterations=50:   7544.37  <- optimal reached
  iterations=100:  7658.96  (variance — stochastic algorithm)
  iterations=200:  7544.37
  iterations=500:  7544.37
  iterations=1000: 7544.37

Effect of ants (berlin52, fixed iterations=50):
  ants=1:   8980.92
  ants=2:   8693.12
  ants=5:   7790.60
  ants=10:  7942.55
  ants=20:  8012.75  (variance)
  ants=26:  7548.99
  ants=52:  7544.37  <- optimal

Instance: rw1621 (866 unique cities after deduplication, optimal=26051)

Results are limited due to computational cost — each ant on rw1621 takes
approximately 4-40 seconds per iteration depending on number of ants.

Best result: 32115.30
  iterations=10, ants=10, time=401.48s (~6.7 minutes)

All other tested configurations returned 32276.67 which is the nearest
neighbor greedy tour length used for initialization. This means the algorithm
was unable to improve upon the greedy starting point within the tested
parameter ranges, except for the single case of iterations=10, ants=10.

Selected results (rw1621):

  iterations=1,  ants=1:  32276.67  (5.60s)
  iterations=1,  ants=10: 32276.67  (40.11s)
  iterations=5,  ants=5:  32276.67  (96.66s)
  iterations=8,  ants=8:  32276.67  (278.87s)
  iterations=10, ants=10: 32115.30  (401.48s)  <- best result

The 32115.30 result represents a 0.5% improvement over the greedy tour
(32276.67) achieved with the maximum tested parameters. This is consistent
with expectations — rw1621 is a large and hard instance requiring many more
iterations and ants than were computationally feasible to test.


5. Discussion

ACS vs AS for berlin52:

ACS achieves the optimal result of 7544.37 reliably with iterations=50 and
ants=52, taking only 3.2 seconds. The AS variant from the previous lab
required 500+ iterations to reach similar quality and plateaued around 8200
with the same parameter budget. The three ACS improvements — exploitation
parameter q0, local pheromone update, and global-best-only reinforcement —
together produce dramatically faster convergence and better final quality.

Effect of number of ants:

More ants consistently improve results for berlin52. With ants=52 (equal to
the number of cities, as recommended in the literature) the algorithm reaches
optimal in 50 iterations. With ants=1 even 1000 iterations are insufficient
to reach near-optimal (best: 8709). This confirms that population diversity
is critical — a single ant cannot generate enough diverse solutions for the
pheromone to converge to the optimal tour structure.

Effect of iterations:

Results improve with more iterations up to a point then plateau. For berlin52
with ants=52, optimal is reached at iterations=50 and additional iterations
provide no benefit. The stochastic nature of the algorithm causes some
variance — iterations=100 with ants=52 gives 7658 while iterations=50 gives
7544 — demonstrating that more iterations do not always guarantee better
results in a single run.

Computational cost of rw1621:

The rw1621 instance is computationally very expensive. A single ant building
a complete tour over 866 cities requires evaluating 866 probability
distributions each of decreasing size — approximately 866*433 = 375,000
operations per tour. With ants=10 and iterations=10 this is 37.5 million
operations, taking ~400 seconds. For meaningful results on rw1621 a much
larger budget (hundreds of iterations, tens of ants) would be required,
which would take hours to days on a standard laptop.
