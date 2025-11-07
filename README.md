# CI2025_LAB2 – Genetic Algorithm for the Traveling Salesman Problem  
## Cooperating with Riccardo Vaccari (348856)

## Description  
This project faces the **Traveling Salesman Problem (TSP)** — finding the shortest possible tour that visits each city exactly once and returns to the starting point.  

Problem instances can be extracted from the files `problems/problem_<type>_<size>.npy`, where:
- type: problem family 
  - `g` -> symmetric matrices with positive values (Standard TSP)
  - `r1`-> asymmetric matrices with positive values 
  - `r2`-> asymmetric matrices with both positive and negative values 
- size: number of cities (10, 20, 50, 100, 200, 500, 1000)

The optimization process is based on a **Genetic Algorithm (GA)** with adaptive mutation rate and optional soft restarts to escape local minima.


## Algorithm Overview  

### Initialization  
The initial population is built as a mix of **random** and **greedy** tours:  
- 80% random permutations of cities  
- 20% greedy solutions starting from random cities, extending by nearest neighbor  

This ensures both **diversity** and **early good-quality individuals**.

### Fitness Function  
The tour cost is computed as the sum of distances between consecutive cities as `problem[tour, np.roll(tour, -1)].sum()`

### Parent Selection  
Parents are chosen via **tournament selection**, which samples a small subset of the population and selects the fittest individual.

### Crossover Operators  
Different crossovers are used depending on whether the problem is **symmetric** or **asymmetric**:

- **Symmetric TSP** → *Inver-Over Crossover*  
  Combines edges by partially reversing segments following the order of the second parent.

- **Asymmetric TSP** → *Order Crossover (OX)*  
  Preserves a sub-sequence from one parent and fills the remaining positions according to the order of the second.

### Mutation Operators
Also in this case, different mutation are used depending on whether the problem is **symmetric** or **asymmetric**:
- **Inversion Mutation** (symmetric problems): reverses a random segment of the tour  
- **Insert Mutation** (asymmetric problems): extracts a city and reinserts it at a nearby position


### Evolution and Adaptation  
At each generation:
- Offspring are generated via crossover or mutation, according to the current mutation rate 
- The population is sorted by fitness, keeping only the top `POPULATION_SIZE` individuals  
- The best individual **(elitism)** is always preserved

If the population stops improving:
- The mutation rate **increases** every `patience` generations  
- A **soft restart** occurs every `threshold` generations without improvement, where half the population is replaced by new random individuals

This process slightly increases execution time but effectively prevents stagnation and often leads to better overall solutions.

### Adaptive Parameters

The algorithm automatically scales its main parameters according to the size of the problem instance (i.e., the number of cities).

Specifically, the configuration is defined in the `Config` class
- `self.POPULATION_SIZE = max(30, int(num_cities * 0.5))`
- `self.OFFSPRING_SIZE  = max(10, int(self.POPULATION_SIZE * 0.4))`
- `self.MAX_GENERATIONS = int(5000 + num_cities * 25)`
- `self.NO_IMPROVEMENT_LIMIT = int(100 + 100 * math.log2(num_cities))`
  
The chosen values were determined empirically through a trial-and-error process, ensuring stable convergence across different TSP sizes.

## Results

Solution of test problem: 2823.79


<div style="display: flex; justify-content: space-between; gap: 20px;">

<table>
<thead>
<tr><th colspan="2">Summary of best solutions (type = <code>g</code>)</th></tr>
<tr><th>Problem Size</th><th style="text-align:right;">Best Distance</th></tr>
</thead>
<tbody>
<tr><td>10</td><td style="text-align:right;">1497.66</td></tr>
<tr><td>20</td><td style="text-align:right;">1755.51</td></tr>
<tr><td>50</td><td style="text-align:right;">2668.05</td></tr>
<tr><td>100</td><td style="text-align:right;">4085.19</td></tr>
<tr><td>200</td><td style="text-align:right;">5625.20</td></tr>
<tr><td>500</td><td style="text-align:right;">8691.47</td></tr>
<tr><td>1000</td><td style="text-align:right;">12365.70</td></tr>
</tbody>
</table>

<table>
<thead>
<tr><th colspan="2">Summary of best solutions (type = <code>r1</code>)</th></tr>
<tr><th>Problem Size</th><th style="text-align:right;">Best Distance</th></tr>
</thead>
<tbody>
<tr><td>10</td><td style="text-align:right;">184.27</td></tr>
<tr><td>20</td><td style="text-align:right;">343.62</td></tr>
<tr><td>50</td><td style="text-align:right;">556.49</td></tr>
<tr><td>100</td><td style="text-align:right;">749.89</td></tr>
<tr><td>200</td><td style="text-align:right;">1084.31</td></tr>
<tr><td>500</td><td style="text-align:right;">1648.72</td></tr>
<tr><td>1000</td><td style="text-align:right;">2524.99</td></tr>
</tbody>
</table>

<table>
<thead>
<tr><th colspan="2">Summary of best solutions (type = <code>r2</code>)</th></tr>
<tr><th>Problem Size</th><th style="text-align:right;">Best Distance</th></tr>
</thead>
<tbody>
<tr><td>10</td><td style="text-align:right;">-411.70</td></tr>
<tr><td>20</td><td style="text-align:right;">-845.12</td></tr>
<tr><td>50</td><td style="text-align:right;">-2256.98</td></tr>
<tr><td>100</td><td style="text-align:right;">-4699.89</td></tr>
<tr><td>200</td><td style="text-align:right;">-9603.29</td></tr>
<tr><td>500</td><td style="text-align:right;">-24603.41</td></tr>
<tr><td>1000</td><td style="text-align:right;">-49477.87</td></tr>
</tbody>
</table>

</div>

## Visualization  

Has been added a sample plot showing **fitness progression** across some runs for each problem size (different number of cities).  
