# Wordle MEM Report

This repository contains a technical report on **Modified Entropy Maximizing (MEM)**. The project analyzes Wordle-solving strategies from an entropy-based perspective and experimentally examines the trade-off between partition diversity and partition uniformity.

## Overview

Wordle can be interpreted as an information search problem.

Each guess partitions the current set of possible answers into multiple groups according to Wordle feedback patterns. A good guess can be understood as one that effectively reduces the uncertainty of the remaining candidate set after receiving feedback.

A standard entropy-based Wordle solver evaluates each guess based on the entropy of the feedback distribution it induces. However, entropy combines two different properties into a single value:

1. **Partition diversity** — how many non-empty feedback blocks are created
2. **Partition uniformity** — how evenly the candidates are distributed among those blocks

This report separates these two components and analyzes the trade-off between them through a scoring function called **Modified Entropy Maximizing (MEM)**.

## Key Results

`λ ∈ [0, 1]` is a parameter that controls the relative weight between partition diversity and partition uniformity.

* `λ = 0.5` has the same form as normalized standard entropy.
* `λ > 0.5` places more emphasis on partition diversity.
* `λ < 0.5` places more emphasis on partition uniformity.

The experiments were conducted under the following setting:

* 2,309 Wordle answer candidates
* 12,947 allowed guess words
* Comparison from `λ = 0.0` to `λ = 1.0` in increments of 0.1

| λ | First Guess | Average Number of Guesses | Maximum Number of Guesses |
| --- | ----------- | ------------------------: | ------------------------: |
| 0.0 | roate | 5.0645 | 9 |
| 0.5 | soare | 3.4638 | 6 |
| 0.7 | reast | 3.4353 | 5 |
| 1.0 | trace | 3.4305 | 6 |

In terms of the average number of guesses, `λ = 1.0` achieved the best performance with **3.4305 guesses**.

On the other hand, `λ = 0.7` solved every answer within **5 guesses**, and it solved the largest number of answers within 4 guesses. Therefore, while `λ = 1.0` performs best in terms of average performance, `λ = 0.7` can be considered a more balanced setting when taking the stability of the solving distribution into account.

These results suggest that emphasizing partition diversity can improve the average performance of entropy-based Wordle strategies, while an appropriate balance between diversity and uniformity can lead to a more stable solving distribution.

## Repository Structure

```text
.
├── FollowUp/
│   ├── Dynamic_MEM.md
│   └── Why_MEM_works.md
├── LatexSources/
│   ├── main_EN.tex
│   └── main_KOR.tex
├── notebooks/
│   ├── BnC MEM.ipynb
│   ├── ModifiedEntropyMaximizing.ipynb
│   └── bellman_dynamic_MEM.ipynb
├── MEM_Wordle_Solver.exe
├── README.md
├── report_EN.pdf
├── report_KOR.pdf
└── requirements.txt
```

## Reports

* [English Report](./report_EN.pdf)
* [Korean Report](./report_KOR.pdf)

## LaTeX Sources

The LaTeX source files are located in the following directory:

```text
LatexSources/
```

The files are organized as follows:

```text
LatexSources/main_EN.tex
LatexSources/main_KOR.tex
```

## Supplementary Notebook

The experimental notebook is located at:

```text
notebooks/ModifiedEntropyMaximizing.ipynb
```

This notebook contains the implementation and experimental analysis used to write the report.

## Supplementary Demo

A Windows executable file is included:

```text
MEM_Wordle_Solver.exe
```

This executable is a Wordle solver implemented based on the ideas presented in the report.

## Follow-up: Why MEM Works?

This follow-up note discusses why diversity-heavy MEM works well for Wordle.

The main idea is that the optimal value of $\lambda$ reflects the structure of the answer space. In a symmetric game such as Bulls and Cows, the entropy-equivalent setting $\lambda = 0.5$ performs best. In Wordle, however, the answer space is biased by English word structure, so a diversity-dominant value such as $\lambda = 0.7$ performs better.

* Essay: [Why_MEM_works.md](./FollowUp/Why_MEM_works.md)
* Bulls and Cows experiment: [BnC MEM.ipynb](./notebooks/BnC%20MEM.ipynb)

## Follow-up: Dynamic MEM

The original MEM strategy uses a fixed value of $\lambda$ throughout the game. However, the best balance between information gain and candidate reduction may vary depending on the current candidate set.

Dynamic MEM allows $\lambda$ to be selected independently at each state. For each candidate set, it generates MEM-optimal guesses from multiple values of $\lambda$, then uses a Bellman-style recursive evaluation to choose the guess with the lowest expected future cost.

Using the official Wordle answer set, Dynamic MEM achieved:

* Average guesses: 3.429190
* Maximum guesses: 5
* Solved within 4 guesses: 2258 / 2309

This result is slightly better than the best tested fixed-λ MEM strategy, suggesting that the preferred balance between entropy and expected candidate reduction changes over the course of the game.

For the full method, recurrence relation, and experimental results, see:

* [Dynamic MEM](./FollowUp/Dynamic_MEM.md)
