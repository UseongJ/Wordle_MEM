# Dynamic MEM

기존 MEM은 하나의 $\lambda$를 선택한 뒤, 그 값을 전체 풀이 과정에서 고정하여 사용한다.

이번 follow-up에서는 이를 확장하여, $\lambda$를 전역적으로 고정하지 않고 **각 후보 상태마다 다시 선택하는 Dynamic MEM**을 시도한다.

핵심 아이디어는 간단하다.

> 각 상태에서 여러 $\lambda$가 제안하는 MEM 추측어를 만든 뒤,
> 그중 미래의 기대 시도 횟수가 가장 작은 선택을 사용한다.

---

## Statewise MEM Candidates

현재 후보 집합을 $C$, 허용 추측어 집합을 $G$라고 하자.

$\lambda$ 후보 집합은 다음과 같이 설정한다.

```math
\Lambda
=
\{0.40, 0.45, 0.50, \ldots, 0.95, 1.00\}
```

각 $\lambda \in \Lambda$에 대해 현재 상태에서 가장 높은 MEM score를 갖는 추측어를 다음과 같이 정의한다.

```math
g_\lambda(C)
=
\underset{g \in G}{\arg\max}\;
M_\lambda(g \mid C)
```

그러면 현재 상태에서 실제로 평가할 행동 집합은 다음과 같다.

```math
\mathcal{A}_{\mathrm{MEM}}(C)
=
\left\{
g_\lambda(C)
:
\lambda \in \Lambda
\right\}
```

서로 다른 $\lambda$가 같은 추측어를 선택할 수 있으므로, 중복된 추측어는 한 번만 평가한다.

---

## Bellman Evaluation

현재 상태 $C$에서 추측어 $g_\lambda(C)$를 선택했을 때, 피드백 $p$에 따라 만들어지는 다음 후보 집합을 다음과 같이 나타내자.

```math
C_p
=
C_p\!\left(g_\lambda(C)\right)
```

현재 추측을 포함하여 앞으로 필요한 평균 추가 시도 횟수를 $Q(C,\lambda)$로 정의하면 다음과 같다.

```math
Q(C,\lambda)
=
1
+
\sum_{p \neq p_{\mathrm{green}}}
\frac{|C_p|}{|C|}
V(C_p)
```

여기서 $p_{\mathrm{green}}$은 정답 피드백이다. 이 분기에서는 현재 추측으로 이미 문제가 해결되므로 미래 비용을 추가하지 않는다.

현재 상태의 value는 다음과 같이 정의한다.

```math
V(C)
=
\min_{\lambda \in \Lambda}
Q(C,\lambda)
```

중요한 점은 한 상태에서 선택된 $\lambda$를 하위 상태에 그대로 전달하지 않는다는 것이다.

각 하위 상태 $C_p$에서는 다시 다음 값을 계산한다.

```math
V(C_p)
=
\min_{\lambda' \in \Lambda}
Q(C_p,\lambda')
```

따라서 $\lambda$는 전체 게임에 고정된 hyperparameter가 아니라, 각 후보 상태에서 다시 선택되는 policy parameter가 된다.

---

## Terminal States

후보가 하나만 남으면 해당 단어를 직접 추측하면 되므로 다음과 같다.

```math
V(\{a\}) = 1
```

후보가 두 개 남은 경우에는 남은 후보 중 하나를 추측하면 평균적으로 다음 결과를 얻는다.

```math
V(C) = 1.5,
\qquad
V_{\max}(C) = 2,
\qquad
|C| = 2
```

---

## Selection Rule

각 상태에서는 평균 추가 시도 횟수가 가장 작은 선택을 우선한다.

평균값이 수치 오차 범위 안에서 같으면 다음 순서로 tie-breaking한다.

1. 최대 추가 시도 횟수가 더 작은 선택
2. 그래도 같다면 더 큰 $\lambda$

각 $\lambda$에서 MEM 추측어 자체가 동점인 경우에는 다음 순서를 사용한다.

1. 기대 잔여 후보 수가 더 작은 추측어
2. 가장 큰 피드백 블록의 크기가 더 작은 추측어
3. 현재 후보 집합에 포함된 추측어
4. 사전순으로 앞선 추측어

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

동일한 후보 상태가 여러 경로에서 반복해서 등장하므로, 상태별 value와 선택 결과를 memoization하여 재사용한다.

---

## Experimental Result

Dynamic MEM의 초기 상태 선택과 전체 결과는 다음과 같다.

| 항목       |           결과 |
| -------- | -----------: |
| 초기 λ     |         0.70 |
| 첫 추측어    |      `reast` |
| 평균 시도 횟수 | **3.429190** |
| 최대 시도 횟수 |        **5** |

시도 횟수별 분포는 다음과 같다.

| 시도 횟수 |  정답 수 |     비율 |   누적 비율 |
| ----: | ----: | -----: | ------: |
|     2 |    68 |  2.94% |   2.94% |
|     3 | 1,233 | 53.40% |  56.34% |
|     4 |   957 | 41.45% |  97.79% |
|     5 |    51 |  2.21% | 100.00% |

모든 2,309개 정답을 5회 이내에 해결했으며, 97.79%의 정답을 4회 이내에 해결하였다.

---

## Comparison

기존 고정 $\lambda$ 결과와 비교하면 다음과 같다.

| 방법              |  첫 추측어  |     평균 시도 횟수 | 최대 시도 횟수 |
| --------------- | :-----: | -----------: | -------: |
| 고정 λ = 0.7      | `reast` |       3.4353 |        5 |
| 고정 λ = 1.0      | `trace` |       3.4305 |        6 |
| **Dynamic MEM** | `reast` | **3.429190** |    **5** |

Dynamic MEM은 기존 고정 $\lambda$ 결과보다 평균 시도 횟수를 소폭 줄이면서, 최대 시도 횟수 5회를 유지했다.

고정 $\lambda = 1.0$과의 평균 차이는 약 다음과 같다.

```math
3.4305 - 3.429190
\approx
0.0013
```

성능 향상 자체는 크지 않지만, 하나의 $\lambda$를 전체 게임에 고정하는 대신 상태별로 다시 선택하는 방식이 조금 더 나은 policy를 만들 수 있음을 보여준다.

---

## Limitation

이 방법에는 다음과 같은 제한이 있다.

* 연속적인 $\lambda$ 전체가 아니라 정해진 grid만 탐색한다.
* 모든 허용 추측어가 아니라 MEM이 제안한 추측어만 평가한다.
* 결과는 사용한 word list와 tie-breaking rule에 의존한다.
* 고정 $\lambda$ MEM보다 계산 비용이 크다.

---

## Conclusion

이번 follow-up에서는 기존 MEM의 $\lambda$를 게임 전체에 고정하지 않고, 각 후보 상태에서 다시 선택하도록 확장했다.

각 $\lambda$가 제안하는 MEM 추측어를 만들고, 그 추측어들의 미래 기대 시도 횟수를 Bellman recurrence로 비교한다.

결과적으로 다음 성능을 얻었다.

* 평균 시도 횟수: **3.429190**
* 최대 시도 횟수: **5**
