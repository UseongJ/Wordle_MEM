# Dynamic MEM

The original MEM selects a single value of $\lambda$ and keeps it fixed throughout the entire solving process.

In this follow-up, we extend the method by introducing **Dynamic MEM**, where $\lambda$ is not fixed globally but is selected again at each candidate state.

The main idea is simple:

> At each state, generate the MEM guesses proposed by multiple values of $\lambda$,
> then choose the one with the smallest expected number of future guesses.

---

## Statewise MEM Candidates

Let $C$ be the current candidate set and $G$ be the set of allowed guesses.

We define the candidate set of $\lambda$ values as follows:

```math
\Lambda
=
\{0.40, 0.45, 0.50, \ldots, 0.95, 1.00\}
```

For each $\lambda \in \Lambda$, let the guess with the highest MEM score at the current state be

```math
g_\lambda(C)
=
\underset{g \in G}{\arg\max}\;
M_\lambda(g \mid C)
```

The set of actions that are actually evaluated at the current state is therefore

```math
\mathcal{A}_{\mathrm{MEM}}(C)
=
\left\{
g_\lambda(C)
:
\lambda \in \Lambda
\right\}
```

Different values of $\lambda$ may select the same guess, so duplicate guesses are evaluated only once.

---

## Bellman Evaluation

Suppose the guess $g_\lambda(C)$ is selected at the current state $C$.

Let the next candidate set produced by feedback pattern $p$ be

```math
C_p
=
C_p\!\left(g_\lambda(C)\right)
```

Define $Q(C,\lambda)$ as the expected number of additional guesses required from the current state, including the current guess:

```math
Q(C,\lambda)
=
1
+
\sum_{p \neq p_{\mathrm{green}}}
\frac{|C_p|}{|C|}
V(C_p)
```

Here, $p_{\mathrm{green}}$ denotes the correct-answer feedback pattern.

No future cost is added for this branch because the answer has already been found by the current guess.

The value of the current state is defined as

```math
V(C)
=
\min_{\lambda \in \Lambda}
Q(C,\lambda)
```

An important point is that the value of $\lambda$ selected at one state is not passed down to its child states.

At each child state $C_p$, the optimization is performed again:

```math
V(C_p)
=
\min_{\lambda' \in \Lambda}
Q(C_p,\lambda')
```

Thus, $\lambda$ is no longer a hyperparameter fixed for the entire game. Instead, it becomes a policy parameter that is selected separately at each candidate state.

---

## Terminal States

If only one candidate remains, the solver can guess that word directly:

```math
V(\{a\}) = 1
```

If two candidates remain, guessing one of them gives the following average and maximum costs:

```math
V(C) = 1.5,
\qquad
V_{\max}(C) = 2,
\qquad
|C| = 2
```

---

## Selection Rule

At each state, the action with the smallest expected number of additional guesses is preferred.

If the expected values are equal within numerical tolerance, ties are broken in the following order:

1. Choose the action with the smaller maximum number of additional guesses.
2. If they are still tied, choose the larger value of $\lambda$.

If multiple guesses have the same MEM score for a given $\lambda$, the following tie-breaking rules are used:

1. Choose the guess with the smaller expected number of remaining candidates.
2. Choose the guess with the smaller largest feedback block.
3. Prefer a guess that belongs to the current candidate set.
4. Choose the lexicographically earlier guess.

---

## Algorithm

```text
Solve(C):
    if |C| = 1:
        return 1

    if |C| = 2:
        return mean 1.5, maximum 2

    for lambda in Lambda:
        g_lambda(C) =
            best MEM guess under lambda

    remove duplicated guesses

    for each candidate guess:
        partition C by feedback
        recursively evaluate every child state
        compute mean cost and maximum cost

    choose the candidate with minimum mean cost
    break ties by smaller maximum cost
    break remaining ties by larger lambda

    cache and return the result
```

Because the same candidate state can appear through multiple paths, the value and selected action for each state are stored and reused through memoization.

---

## Implementation

The implementation used for the experiment is available in the following notebook:

* [Dynamic MEM Notebook](../notebooks/bellman_dynamic_MEM.ipynb)

---

## Experimental Results

The initial-state choice and overall performance of Dynamic MEM are shown below.

| Item                      |       Result |
| ------------------------- | -----------: |
| Initial λ                 |         0.70 |
| First guess               |      `reast` |
| Average number of guesses | **3.429190** |
| Maximum number of guesses |        **5** |

The distribution by number of guesses is as follows:

| Number of guesses | Number of answers | Percentage | Cumulative percentage |
| ----------------: | ----------------: | ---------: | --------------------: |
|                 2 |                68 |      2.94% |                 2.94% |
|                 3 |             1,233 |     53.40% |                56.34% |
|                 4 |               957 |     41.45% |                97.79% |
|                 5 |                51 |      2.21% |               100.00% |

All 2,309 answers were solved within five guesses, and 97.79% of the answers were solved within four guesses.

---

## Comparison

The results are compared with the previous fixed-$\lambda$ methods below.

| Method          | First guess | Average number of guesses | Maximum number of guesses |
| --------------- | :---------: | ------------------------: | ------------------------: |
| Fixed λ = 0.7   |   `reast`   |                    3.4353 |                         5 |
| Fixed λ = 1.0   |   `trace`   |                    3.4305 |                         6 |
| **Dynamic MEM** |   `reast`   |              **3.429190** |                     **5** |

Dynamic MEM slightly reduces the average number of guesses compared with the fixed-$\lambda$ methods while preserving a maximum of five guesses.

The difference in average performance compared with fixed $\lambda = 1.0$ is approximately

```math
3.4305 - 3.429190
\approx
0.0013
```

The improvement itself is small. However, the result shows that selecting $\lambda$ separately at each state can produce a slightly better policy than using a single fixed value throughout the entire game.

---

## Limitations

This method has several limitations:

* It searches only over a predefined grid of $\lambda$ values rather than over the full continuous range.
* It evaluates only the guesses proposed by MEM, rather than all allowed guesses.
* The results depend on the word list and tie-breaking rules used.
* It requires more computation than fixed-$\lambda$ MEM.

---

## Conclusion

In this follow-up, we extended the original MEM by allowing $\lambda$ to be selected separately at each candidate state instead of fixing it for the entire game.

At each state, the method generates the guesses proposed by different values of $\lambda$ and compares their expected future costs using a Bellman recurrence.

The resulting performance is:

* Average number of guesses: **3.429190**
* Maximum number of guesses: **5**
