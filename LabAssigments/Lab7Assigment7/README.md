1. Problem Definition

This assignment applies two variants of Particle Swarm Optimization (PSO) to minimize
two continuous benchmark functions. PSO is a population-based metaheuristic inspired by
the social behavior of bird flocking and fish schooling.

Each particle in the swarm represents a candidate solution — a real-valued vector in the
search space. Particles move through the search space guided by their own best known
position and the swarm's global best known position.

Two problem instances were used:

Sphere Function (De Jong's function 1):
  f(x) = sum(xi^2), i=1:n
  Bounds: -5.12 <= xi <= 5.12
  Global optimum: f(x) = 0 at x(i) = 0
  Unimodal, convex, no local minima except the global one.

Schwefel 1 (Rotated hyper-ellipsoid function):
  f(x) = sum_{i=1}^{n} sum_{j=1}^{i} xj^2
  Bounds: -65.536 <= xi <= 65.536
  Global optimum: f(x) = 0 at x(i) = 0
  Unimodal, convex, variables are coupled — xi affects all terms where index >= i.

All experiments used dimension n = 3.


2. Algorithm

Two PSO variants were implemented and compared.

PSO Variant 1 — Inertia Weight PSO (PSO1):

The standard PSO formulation with an inertia weight w that controls how much of the
previous velocity is retained at each step. The velocity update rule is:

  v(t+1) = w * v(t) + c1 * r1 * (pbest - x(t)) + c2 * r2 * (gbest - x(t))
  x(t+1) = x(t) + v(t+1)

where r1 and r2 are random numbers in [0,1] sampled independently each iteration,
pbest is the particle's personal best position, and gbest is the swarm's global best.

Parameters :
  w  = 0.7    (inertia weight — balances exploration and exploitation)
  c1 = 1.6    (cognitive coefficient — attraction toward personal best)
  c2 = 1.6    (social coefficient — attraction toward global best)

PSO Variant 2 — Constriction Coefficient PSO (PSO2):

Instead of a manually tuned inertia weight, a constriction coefficient chi is derived
mathematically from c1 and c2. The velocity update rule is:

  v(t+1) = chi * (v(t) + c1*r1*(pbest-x(t)) + c2*r2*(gbest-x(t)))
  x(t+1) = x(t) + v(t+1)

The constriction coefficient chi is computed as:
  phi = c1 + c2
  chi = 2 / |2 - phi - sqrt(phi^2 - 4*phi)|

Parameters :
  c1  = 2.05   (cognitive coefficient)
  c2  = 2.05   (social coefficient)
  phi = 4.10   (c1 + c2, must be > 4)
  chi = 0.7298 (derived automatically)

The key theoretical advantage of the constriction variant is that convergence is
mathematically guaranteed as long as phi > 4, eliminating the risk of particle
divergence that exists with inertia weight PSO if w is set too high.

Both variants share the same structure:

1. Initialize swarm_size particles with random positions in [lower, upper] and random
   velocities.
2. Evaluate fitness of each particle, initialize personal bests and global best.
3. For each iteration:
   a. Update velocity of each particle using the respective formula.
   b. Update position: x = x + v.
   c. Clip position to bounds.
   d. Evaluate new fitness.
   e. Update personal best if new position is better.
   f. Update global best if personal best improved it.
4. Return global best position and fitness.


3. Parameter Settings

Fixed parameters:
  dimension: 3
  w (PSO1): 0.7
  c1, c2 (PSO1): 1.6, 1.6
  c1, c2 (PSO2): 2.05, 2.05

Parameters varied:
  swarm_size:  [5, 10, 30, 50, 100, 500]
  iterations:  [5, 50, 100, 500, 1000, 2000]

Total combinations: 6 * 6 = 36 per variant per function, 144 total.


4. Results

Instance: Sphere Function

PSO1 (Inertia Weight) — selected results:

  swarm=5,   iter=5:    0.867
  swarm=5,   iter=500:  3.07e-12
  swarm=5,   iter=2000: 3.76e-19
  swarm=10,  iter=500:  1.52e-46
  swarm=10,  iter=1000: 1.29e-81
  swarm=10,  iter=2000: 3.80e-167
  swarm=30,  iter=500:  1.15e-50
  swarm=30,  iter=2000: 6.59e-189
  swarm=100, iter=500:  2.61e-55
  swarm=100, iter=2000: 1.96e-200
  swarm=500, iter=2000: 4.32e-206   

PSO2 (Constriction) — selected results:

  swarm=5,   iter=5:    1.418
  swarm=5,   iter=500:  5.73e-15
  swarm=5,   iter=2000: 1.32e-06
  swarm=10,  iter=500:  2.03e-38
  swarm=10,  iter=2000: 8.02e-150
  swarm=30,  iter=500:  5.30e-44
  swarm=30,  iter=2000: 1.19e-167
  swarm=100, iter=500:  5.04e-50
  swarm=100, iter=2000: 8.46e-181
  swarm=500, iter=2000: 8.29e-188   

Instance: Schwefel 1 Function

PSO1 (Inertia Weight) — selected results:

  swarm=5,   iter=5:    1.303
  swarm=5,   iter=500:  2.82e-22
  swarm=5,   iter=2000: 9.10e-18
  swarm=10,  iter=500:  3.06e-40
  swarm=10,  iter=2000: 5.32e-158
  swarm=30,  iter=2000: 1.07e-181
  swarm=100, iter=2000: 6.35e-200
  swarm=500, iter=2000: 1.24e-208   

PSO2 (Constriction) — selected results:

  swarm=5,   iter=5:    2.949
  swarm=5,   iter=500:  7.21e-19
  swarm=5,   iter=2000: 5.67e-11
  swarm=10,  iter=500:  1.03e-39
  swarm=10,  iter=2000: 2.25e-152
  swarm=30,  iter=2000: 1.69e-167
  swarm=100, iter=2000: 2.98e-180
  swarm=500, iter=2000: 3.17e-189   


5. Discussion

PSO1 vs PSO2:

PSO1 consistently outperforms PSO2 across both functions and almost all parameter
combinations. The best results comparison:

  Sphere:   PSO1 = 4.32e-206,  PSO2 = 8.29e-188  (PSO1 wins)
  Schwefel: PSO1 = 1.24e-208,  PSO2 = 3.17e-189  (PSO1 wins)

This result is counterintuitive since constriction PSO has stronger theoretical
convergence guarantees. The likely explanation is that for n=3 unimodal functions,
divergence is not a risk so the theoretical advantage of constriction does not matter.
The inertia weight w=0.7 with c1=c2=1.6 happens to be a well-tuned combination for
these specific functions, while the constriction parameters c1=c2=2.05 and chi=0.7298
produce slightly more conservative velocity updates that converge more slowly.

Effect of swarm size:

Larger swarms improve results but with diminishing returns. The improvement from
swarm=5 to swarm=10 is dramatic (e.g. sphere PSO1: 3.76e-19 vs 3.80e-167 at 2000
iterations). Beyond swarm=30 the improvement is smaller. This makes sense — a swarm
of 10+ particles covers the search space well enough for a 3-dimensional unimodal
problem that additional particles provide minimal benefit.

Effect of iterations:

Iterations have the most dramatic impact on result quality. At iter=5 both variants
give poor results (0.07 to 1.4 fitness) while at iter=2000 results reach below 1e-180
for most swarm sizes. The results scale roughly as e^(-iterations) suggesting
exponential convergence toward the optimum, which is characteristic of PSO on
unimodal functions.


Sphere vs Schwefel:

Both functions converge to essentially the same quality results under the same
parameters, confirming that both are unimodal and accessible to PSO. Schwefel 1 is
slightly harder at low iteration counts (the coupling between variables slows initial
convergence) but the final results are comparable.

Both PSO variants vastly outperform the EA results from the previous section of this
lab. The best EA result for sphere was 1.53e-19 while PSO1 reaches 4.32e-206 — a
difference of 187 orders of magnitude. This demonstrates the fundamental advantage of
PSO for continuous unimodal optimization: the velocity mechanism guides particles
directly toward the optimum rather than relying on random crossover and mutation.
