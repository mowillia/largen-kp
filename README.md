# Large N Limit of Knapsack Problem

<p align="center">
<img align = "center" src = "https://user-images.githubusercontent.com/8810308/111638380-a53b3400-87d0-11eb-9407-78a613cdd922.png"  onmouseover= "Motivation for statistical physics based algorithm" width = "75%">
</p>

The Knapsack Problem is a classic problem from combinatorial optimization. In the "0-1" version of the problem [1], we are given N objects each of which has a value and a weight, and our objective is to find the collection of objects that maximizes the total value of the collection while ensuring that the weight remain under a given maximum. 

This repository provides algorithms for solving various incarnations of the  Knapsack Problem in the limit of where the total number of elements is large. Currently the libary supports approximate solutions to the "0-1", "bounded", and "unbounded" versions of the problem. 

There are exact algorithms for the knapsack problem [(RossettaCode Knapsack)](https://rosettacode.org/wiki/Knapsack_problem), but these take longer as the number of items increases. The algorithms in this repository provide approximate solutions in much less time. 

Table of times?

Problem Setup 

## Knapsack Instance

In the following examples, we will use the item list, weights, values, and weight limits given as follows.
```
import numpy as np

items = (
    ("map", 9, 150), ("compass", 13, 35), ("water", 153, 200), ("sandwich", 50, 160),
    ("glucose", 15, 60), ("tin", 68, 45), ("banana", 27, 60), ("apple", 39, 40),
    ("cheese", 23, 30), ("beer", 52, 10), ("suntan cream", 11, 70), ("camera", 32, 30),
    ("t-shirt", 24, 15), ("trousers", 48, 10), ("umbrella", 73, 40),
    ("waterproof trousers", 42, 70), ("waterproof overclothes", 43, 75),
    ("note-case", 22, 80), ("sunglasses", 7, 20), ("towel", 18, 12),
    ("socks", 4, 50), ("book", 30, 10),
    )

# defining weight and value vectors and weight limit
weight_vec = np.array([item[1] for item in items])
value_vec = np.array([item[2] for item in items])
Wlimit = 400
```

These values are taken from the problem statement in [RossettaCode Knapsack: 0-1](https://rosettacode.org/wiki/Knapsack_problem/0-1)

## Running Large N algorithm

Given weights, values, and a limit, the large N algorithm outputs a list of 1s and 0 correspon algorithm corresponding to putting the respective item in the list in the knapsack or leaving it out. To quickly run the algorithm, execute the following code after defining the item list above.

```
from largeN import zero_one_algorithm

soln = zero_one_algorithm(weights = weight_vec, values = value_vec, limit = Wlimit)
for k in range(len(soln)):
    if soln[k] == 1:
        print(items[k][0])
```

```
>>>
map
compass
water
sandwich
glucose
banana
suntan cream
waterproof trousers
waterproof overclothes
note-case
sunglasses
socks
```

To apply the problem to other instances of items, values, and weights, just replace the values and weight lists in the quick start up with your chosen lists. 

## Plotting potential function

The potential function for the zero-one problem is 
```
FN_zero_one = lambda z, weights, values, limit, T: - limit*np.log(z)-np.log(1-z) + np.sum(np.log(1+z**(weights)*np.exp(values/T)))
```
This function gives a continuous representation of the standard discrete optimization objective. If the function has a local minimum, then the large N algorithm can solve the knapsack problem. This minimum depends on temperature, and as the temperature is lowered the minimum better defines an optimal solution for the knapsack problem. To plot the potential function for the above instance, execute the following code. 

```
from plots import plot_potential_zero_one
import numpy as np

plot_potential_zero_one(weights = weight_vec, values = value_vec, limit = Wlimit, T= 1.5)
>>>
```
<p align="center">
<img align = "middle" src = "https://user-images.githubusercontent.com/8810308/111629285-84221580-87c7-11eb-9486-6828c446040d.png" width = "40%">
</p>

## Plotting total value as a function of temperature

To plot the calculated total value as a function of temperature, execute the following code

```
from plots import plot_value_vs_temp

plot_value_vs_temp(weights = weight_vec, values = value_vec, limit = Wlimit, temp_low=1.0, temp_high = 60.0)
>>>
```
<p align="center">
<img align = "middle" src = "https://user-images.githubusercontent.com/8810308/111657616-fd2e6680-87e1-11eb-8077-cd7ed1f25c29.png" width = "40%">
</p>

## Algorithm comparison plots

In the original paper, we compare the performance of various classic knapsack problem algorithms to the proposed algorithm. The algorithms we compare are

- **Brute Force:**(`brute_force`) Involves listing all possible combinations of items, computing the total weights and total values of each combination and selecting the combination with the highest value with a weight below the stated limit. 

- **Dynamical Programming Solution:**(`knapsack_dpV`)  Standard iterative solution to the problem which involves storing sub-problem solutions in matrix elements

- **Fully Polynomial Time Approximate Solution (FPTAS):**(`fptas`)  Algorithm that is polynomial time in the number of elements and which has a tunable accuracy

- **Greedy Algorithm:**(`greedy`)  Involves computing the ratio of weights to volumes for each object and filling in the collection until the max weight is reached. 

- **Simulated Annealing:**(`simannl_knapsack`)   Involves representing the system computationally as a statistical physics one and then "annealing" the system to low temperatures. 

- **Large N Algorithm:**(`zero_one_algorithm`)  Algorithm based on statistical physics representation of the system

A quick comparison of these algorithms for the problem instance shown above is given by the following code. 

Assembling needed algorithms and modules
```
from brute import brute_force
from dp import knapsack01_dpV
from fptas import fptas
from greedy import greedy
from annealing import simann_knapsack
from largeN import zero_one_algorithm

from tabulate import tabulate

import time
```
Defining dictionary of algorithms and empty dictionary for results
```
# dictionary of algorithm names and functions
algo_name_dict = {'Brute': brute_force,
                  'DP': knapsack01_dpV,
                  'FPTAS': fptas,
                  'Greedy': greedy,
                  'Annealing': simann_knapsack,
                  'Large N': zero_one_algorithm}

# dictionary of algorithm names and results
results_name_dict = dict()
```
Running algorithm and creating table of results
```
for name, func in algo_name_dict.items():
    start_clock = time.time()
    soln  = func(weights = weight_vec, values = value_vec, limit = Wlimit)
    
    # calculating values
    tot_value = str(round(np.dot(value_vec, soln), 0))
    tot_weight = str(round(np.dot(weight_vec, soln), 0))
    time_calc = str(round(time.time()-start_clock, 5)) 
    
    # assembling results
    results_name_dict[name] = [name, tot_value, tot_weight, time_calc]
    
# creating table of results
tabular_results = []
for k, v in results_name_dict.items():
    tabular_results.append(v) 
```
Printing Table
```
print(tabulate(tabular_results, ["Algorithm", "Value", "Weight", "Time (sec)"], tablefmt="grid"))
>>>
+-------------+---------+-----------+--------------+
| Algorithm   |   Value |   Weight  |   Time (sec) |
+=============+=========+===========+==============+
| Brute       |    1030 |       396 |     33.0739  |
+-------------+---------+-----------+--------------+
| DP          |    1030 |       396 |      0.00488 |
+-------------+---------+-----------+--------------+
| FPTAS       |    1030 |       396 |      0.00396 |
+-------------+---------+-----------+--------------+
| Greedy      |    1030 |       396 |      8e-05   |
+-------------+---------+-----------+--------------+
| Annealing   |     920 |       387 |      0.13638 |
+-------------+---------+-----------+--------------+
| Large N     |    1030 |       396 |      0.00107 |
+-------------+---------+-----------+--------------+

```


## Knapsack Variations

Repository includes code for two variations of the knapsack problem: The unbounded and the bounded knapsack problems. Implementing them is identical to the implementation for the `zero_one_algorithm`. 

Example

Notes: Although these algorithms are analytically based on a large N approximation, Python has bounds on the size of integers it can process. Thus when the unbounded knapsack problem estimates C (the implicit bound on the number of items in the knapsack; see corresponding section in paper for a discussion) the value it results in could lead to an overflow error. 


## Reproducing figures from paper

The notebooks that reproduce the figures in the paper are as follows

- `potential_landscape.ipynb`: Reproduces Figure 2(a); Runs in < 1 minute
- `total_value_vs_temperature.ipynb`: Reproduces Figure 2(b); Runs in < 1 minute
- `algorithm_comparisons.ipynb`: Reproduces Figure 3; Runs in 15 minutes
- `limit_ratio_vs_temperature`: Reproduces Figure 4; Runs in < 1 minute



## Installation

To install latest stable version:
```
pip install torchdiffeq
```

To install latest on GitHub:
```
pip install git+https://github.com/rtqichen/torchdiffeq
```

## Examples
Examples are placed in the [`examples`](./examples) directory.

We encourage those who are interested in using this library to take a look at [`examples/ode_demo.py`](./examples/ode_demo.py) for understanding how to use `torchdiffeq` to fit a simple spiral ODE.

<p align="center">
<img align="middle" src="./assets/ode_demo.gif" alt="ODE Demo" width="500" height="250" />
</p>

## Basic usage
This library provides one main interface `odeint` which contains general-purpose algorithms for solving initial value problems (IVP), with gradients implemented for all main arguments. An initial value problem consists of an ODE and an initial value,
```
dy/dt = f(t, y)    y(t_0) = y_0.
```
The goal of an ODE solver is to find a continuous trajectory satisfying the ODE that passes through the initial condition.

To solve an IVP using the default solver:
```
from torchdiffeq import odeint

odeint(func, y0, t)
```
where `func` is any callable implementing the ordinary differential equation `f(t, x)`, `y0` is an _any_-D Tensor representing the initial values, and `t` is a 1-D Tensor containing the evaluation points. The initial time is taken to be `t[0]`.

Backpropagation through `odeint` goes through the internals of the solver. Note that this is not numerically stable for all solvers (but should probably be fine with the default `dopri5` method). Instead, we encourage the use of the adjoint method explained in [1], which will allow solving with as many steps as necessary due to O(1) memory usage.

To use the adjoint method:
```
from torchdiffeq import odeint_adjoint as odeint

odeint(func, y0, t)
```
`odeint_adjoint` simply wraps around `odeint`, but will use only O(1) memory in exchange for solving an adjoint ODE in the backward call.

The biggest **gotcha** is that `func` must be a `nn.Module` when using the adjoint method. This is used to collect parameters of the differential equation.

## Differentiable event handling

We allow terminating an ODE solution based on an event function. This can be invoked with `odeint_event`:
```
from torchdiffeq import odeint_event
odeint_event(func, y0, t0, *, event_fn, reverse_time=False, odeint_interface=odeint, **kwargs)
```
 - `func` and `y0` are the same as `odeint`.
 - `t0` is a scalar representing the initial time value.
 - `event_fn(t, y)` returns a tensor, and is a required keyword argument.
 - `reverse_time` is a boolean specifying whether we should solve in reverse time. Default is `False`.
 - `odeint_interface` is one of `odeint` or `odeint_adjoint`, specifying whether adjoint mode should be used for differentiating through the ODE solution. Default is `odeint`.
 - `**kwargs`: any remaining keyword arguments are passed to `odeint_interface`.

The solve is terminated at an event time `t` and state `y` when an element of `event_fn(t, y)` is equal to zero. Multiple outputs from `event_fn` can be used to specify multiple event functions, of which the first to trigger will terminate the solve.

Both the event time and final state are returned from `odeint_event`, and can be differentiated. Gradients will be backpropagated through the event function.

The numerical precision for the event time is determined by the `atol` argument.

See example of simulating and differentiating through a bouncing ball in [`examples/bouncing_ball.py`](./examples/bouncing_ball.py).

<p align="center">
<img align="middle" src="./assets/bouncing_ball.png" alt="Bouncing Ball" width="500" height="250" />
</p>

## Keyword arguments for odeint(_adjoint)

#### Keyword arguments:
 - `rtol` Relative tolerance.
 - `atol` Absolute tolerance.
 - `method` One of the solvers listed below.
 - `options` A dictionary of solver-specific options, see the [further documentation](FURTHER_DOCUMENTATION.md).

#### List of ODE Solvers:

Adaptive-step:
 - `dopri8` Runge-Kutta 7(8) of Dormand-Prince-Shampine
 - `dopri5` Runge-Kutta 4(5) of Dormand-Prince **[default]**.
 - `bosh3` Runge-Kutta 2(3) of Bogacki-Shampine
 - `adaptive_heun` Runge-Kutta 1(2)

Fixed-step:
 - `euler` Euler method.
 - `midpoint` Midpoint method.
 - `rk4` Fourth-order Runge-Kutta with 3/8 rule.
 - `explicit_adams` Explicit Adams-Bashforth.
 - `implicit_adams` Implicit Adams-Bashforth-Moulton.

Additionally, all solvers available through SciPy are wrapped for use with `scipy_solver`.

For most problems, good choices are the default `dopri5`, or to use `rk4` with `options=dict(step_size=...)` set appropriately small. Adjusting the tolerances (adaptive solvers) or step size (fixed solvers), will allow for trade-offs between speed and accuracy.

## Frequently Asked Questions
Take a look at our [FAQ](FAQ.md) for frequently asked questions.

## Further documentation
For details of the adjoint-specific and solver-specific options, check out the [further documentation](FURTHER_DOCUMENTATION.md).

## References
[1] Mobolaji Williams. "Large N Limit of the Knapsack Problem." *Journal Name.* 2021. [[arxiv]](https://arxiv.org/abs/XXXX)

---

If you found this library useful in your research, please consider citing
```
@article{williams2021knapsack,
  title={Large N Limit of the Knapsack Problem},
  author={Williams, Mobolaji},
  journal={arXiv preprint arXiv:CCC},
  year={2021}
}
```
