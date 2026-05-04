
1. Problem Definition

This assignment applies an Evolutionary Algorithm to minimize two continuous benchmark
functions. Unlike previous labs which dealt with discrete optimization (TSP, knapsack),
here solutions are real-valued vectors and the search space is continuous.

The goal is to find x = [x1, x2, ..., xn] that minimizes the fitness function f(x).
The global optimum for both functions is f(x) = 0 at x = [0, 0, ..., 0].

All experiments were run with dimension n = 3.

Function 1: Sphere Function (De Jong's function 1)

  f(x) = sum(xi^2), i=1:n
  Bounds: -5.12 <= xi <= 5.12
  Global optimum: f(x) = 0 at x(i) = 0

The sphere function is continuous, convex, and unimodal. It has no local minima other
than the global one, making it the simplest benchmark — a well-configured EA should
reliably find the optimum.

Function 2: Schwefel 1 (Rotated hyper-ellipsoid function)

  f(x) = sum_{i=1}^{n} sum_{j=1}^{i} xj^2, i=1:n
  Bounds: -65.536 <= xi <= 65.536
  Global optimum: f(x) = 0 at x(i) = 0

Schwefel 1 is also unimodal and convex, but the variables are coupled — each term in
the outer sum includes all previous variables. The effective weight of each variable
increases with its index, creating an asymmetric landscape. x0 contributes to all n
terms while xn-1 contributes only to the last term, making the function harder than
sphere despite sharing the same optimal point.


2. Algorithm

The method implemented is a standard Evolutionary Algorithm (EA) with real-valued
encoding. The algorithm follows scheme where elitism
preserves the best 10 individuals from the current generation alongside newly generated
children.

Steps of the algorithm:

1. Initialize population of pop_size individuals, each a list of n real numbers
   sampled uniformly from [lower, upper].

2. For each generation:
   a. Sort population by fitness (ascending, lower = better).
   b. Keep top 10 individuals unchanged (elitism).
   c. While new population is not full:
      - Select two parents using the chosen selection strategy.
      - Apply crossover operator with probability 0.8 to produce children.
      - Apply mutation to each child.
      - Add children to new population.
   d. Replace population with new population.

3. Return the individual with the lowest fitness from the final population.

Crossover operators (2 implemented):

Complete average crossover: produces one child as the element-wise mean of both parents.
  child[i] = (parent1[i] + parent2[i]) / 2
This always moves the child toward the midpoint between parents, providing strong
exploitation of the region between two good solutions.

Arithmetic crossover: produces two children using a random blending coefficient alpha.
  child1[i] = alpha * parent1[i] + (1 - alpha) * parent2[i]
  child2[i] = (1 - alpha) * parent1[i] + alpha * parent2[i]
Applied with crossover_rate=0.8. If crossover does not occur, children are copies of
the parents. More flexible than complete average as alpha varies randomly each time.

Mutation operators (2 implemented):

Uniform mutation: replaces each gene with a random value from [lower, upper] with
probability mutation_rate=0.01. Provides large jumps across the search space.

Gaussian mutation: perturbs each gene by adding a sample from N(0, sigma). All genes
are mutated every application. Provides normally distributed local perturbations.
sigma = (upper - lower) / 10:
  sphere:   sigma = 10.24 / 10 = 1.024
  schwefel: sigma = 131.072 / 10 = 13.107

Selection strategies (2 implemented):

Tournament selection: randomly samples tournament_size=3 individuals and returns the
one with lowest fitness. Simple, fast, and robust.

Roulette wheel selection: proportional selection where each individual's probability is
proportional to its inverted fitness (since we minimize). Fitnesses are inverted as
max_fit - f + epsilon so better solutions get higher probability. Uses cumulative
probability with a random spin to select.

3. Parameter Settings

Parameters tested:

- iterations: [100, 500, 1000]
- population_size: [50, 100]
- crossover_type: [complete_average, arithmetic]
- mutation_type: [uniform, gaussian]
- selection_type: [tournament, roullete]
- dimension: 3 (fixed)

4. Results

Note: results in the output files were stored as the best individual vector [x1,x2,x3].
Fitness values below were computed by applying the fitness function to these vectors.

Instance: Sphere Function

Best result overall: 1.53e-19
  iterations=1000, population=50, crossover=arithmetic, mutation=uniform, selection=roullete

Top 5 results (sphere):

  1.53e-19  iter=1000, pop=50,  crossover=arithmetic,      mutation=uniform,  selection=roullete
  1.10e-18  iter=500,  pop=50,  crossover=arithmetic,      mutation=uniform,  selection=tournament
  1.71e-17  iter=500,  pop=50,  crossover=arithmetic,      mutation=uniform,  selection=roullete
  3.57e-13  iter=1000, pop=100, crossover=arithmetic,      mutation=uniform,  selection=roullete
  1.26e-12  iter=500,  pop=100, crossover=arithmetic,      mutation=uniform,  selection=roullete

Average fitness by operator type (sphere):

  Mutation:
    uniform:  avg=3.04e-03, best=1.53e-19
    gaussian: avg=6.80e-03, best=6.20e-05

  Crossover:
    arithmetic:      avg=6.98e-03, best=1.53e-19
    complete_average: avg=2.85e-03, best=1.15e-11

  Selection:
    tournament: avg=5.93e-03, best=1.10e-18
    roullete:   avg=3.90e-03, best=1.53e-19

Instance: Schwefel 1 Function

Best result overall: 6.31e-17
  iterations=1000, population=50, crossover=arithmetic, mutation=uniform, selection=roullete

Top 5 results (schwefel):

  6.31e-17  iter=1000, pop=50,  crossover=arithmetic,       mutation=uniform,  selection=roullete
  6.91e-12  iter=1000, pop=50,  crossover=complete_average,  mutation=uniform,  selection=roullete
  6.33e-11  iter=500,  pop=100, crossover=complete_average,  mutation=uniform,  selection=tournament
  3.15e-09  iter=1000, pop=100, crossover=arithmetic,        mutation=uniform,  selection=roullete
  4.96e-08  iter=500,  pop=100, crossover=arithmetic,        mutation=uniform,  selection=roullete

Average fitness by operator type (schwefel):

  Mutation:
    uniform:  avg=1.53e-01, best=6.31e-17
    gaussian: avg=1.43e+00, best=1.84e-01

  Crossover:
    arithmetic:       avg=7.15e-01, best=6.31e-17
    complete_average: avg=8.72e-01, best=6.91e-12

  Selection:
    tournament: avg=7.14e-01, best=6.33e-11
    roullete:   avg=8.73e-01, best=6.31e-17


5. Discussion

Uniform vs Gaussian mutation:

Uniform mutation dramatically outperforms gaussian mutation on both functions. The best
sphere result with uniform mutation is 1.53e-19 while the best with gaussian is only
6.20e-05 — a difference of 14 orders of magnitude. For schwefel the gap is even larger:
uniform achieves 6.31e-17 while gaussian average is 1.43e+00, meaning gaussian often
fails to converge at all within 1000 iterations.

Arithmetic vs complete average crossover:

Arithmetic crossover achieves the best results on both functions (1.53e-19 vs 1.15e-11
for sphere, 6.31e-17 vs 6.91e-12 for schwefel). However complete average has a better
average fitness on sphere (2.85e-03 vs 6.98e-03), suggesting complete average is more
consistent while arithmetic occasionally achieves much better results. The random alpha
in arithmetic crossover allows more diverse children than the fixed midpoint of complete
average, which helps escape local convergence but introduces more variance.

Tournament vs roulette wheel selection:

Both selection strategies perform similarly overall. Roulette wheel achieves the best
individual results on both functions (1.53e-19 vs 1.10e-18 for sphere). Tournament
has a slightly better average on schwefel (7.14e-01 vs 8.73e-01). The difference is
small and both strategies are appropriate for these unimodal functions. For multimodal
functions the difference would likely be larger, with tournament selection more robust
against premature convergence.

Effect of iterations and population:

More iterations consistently improve results. The worst results almost all come from
iterations=100 with gaussian mutation. With uniform mutation, even iterations=500
with population=50 achieves excellent results (1.10e-18 for sphere). Population size
has less impact than mutation type — population=50 achieves better best results than
population=100 in many cases, suggesting that with good operators a smaller population
converges more effectively in the same number of iterations.