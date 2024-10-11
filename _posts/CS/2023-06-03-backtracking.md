---
layout: post
title: "[Algorithm] 백트래킹(Bactracking)"
author: 파녀미
categories: Algorithm
banner:
  image: https://github.com/lcqff/lcqff.github.io/assets/71930280/183f2d64-6716-4a61-bc8f-5cd525bea29f
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold;"
  subheading_style: "color: gold"
tags: Algorithm backtracking
sidebar: []
---

<style>
  .imageRow {
    display:flex;
    margin: 20px 0;
  }
  .captionedImg {
    display: grid;
    align-content: flex-end;
    margin: 0 20px;
    text-align:center;
    font-size: 12px;
    color:gray;
  }
</style>

<div class="captionedImg">
<img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/b3dc2c26-4e13-41b0-8d0c-856625a8a1db">
<p>https://www.programiz.com/dsa/backtracking-algorithm</p>
</div>

<div class="callout">
  <div>💡</div>
  <div>
    <strong>Backtracking</strong><br/>
    주어진 문제에 대한 가능한 모든 경우의 수를 탐색하면서 해를 찾는 알고리즘
  </div>
</div>

- 주어진 자식 노드 중 하나를 골라 탐색한다.
  - 만일 선택한 노드가 조건에 맞지(Promising하지) 않다면 부모 노드로 돌아간다.
  - 만일 선택한 노드가 조건에 맞다면 다음 자식 노드로 이동한다.
- 트리의 끝에 도착했다면 경로를 해에 추가한다.

---

#### 경로 도출하기

어떠한 최적의 답을 구하기 위해서, 우리는 후보 솔루션들을 트리 형태로 구축하라 수 있습니다. 이렇게 구축된 트리를 **상태 공간 트리(State Space Tree)**라고 부릅니다.

이렇게 주어진 상태 공간 트리에서 도출할 수 있는 경로를 구하기 위해 사용되는 알고리즘에는 **DFS(깊이 우선 탐색)**와 **BFS(너비 우선 탐색)**가 있습니다. 그러나 DFS와 BFS는 트리에서 찾을 수 있는 모든 경로를 탐색하므로, 해답에 대한 조건이 주어진 경우 불필요한 경로까지 탐색하여 비효율적일 수 있습니다.

따라서 어떤 조건에 대한 최적의 해를 구하기 위해 **백트래킹**이 사용됩니다.
<br/>

<div class="imageRow">
<img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/1053ef11-82d9-4394-bdfd-b3aac4fb8c83">
<img src="https://github.com/lcqff/lcqff.github.io/assets/71930280/b999fe60-8d83-4d49-ac87-cc2c3ce4af17">
</div>

#### Promising한 노드

> **백트래킹은 결정 문제(예, 아니오로 답할 수 있는 문제)를 다루는 데 사용된다.**

백트래킹은 일반적으로 DFS과 결합되어 사용됩니다. 단, DFS가 해답에 대한 조건과 상관없이 모든 경로를 탐색하는 반면, 백트래킹은 노드를 지날 때마다 해당 노드가 경로에 부합하는가 여부를 판단하는 과정을 거칩니다.

노드가 조건에 부합함을 **'유망하다(Promising 하다)'**고 하며, 노드가 유망한 경우 다음 자식 노드 중 하나를 골라 탐색합니다.
만일 노드가 유망하지 않을 경우, 우리는 바로 직전의 노드로 돌아가 다른 경로를 선택하게 됩니다. 이를 **Backtracking**이라 부릅니다.
<br/>이때, 직전의 노드로 돌아가기위해 DFS의 **재귀적인 특성**이 사용됩니다.

이렇게 노드의 유망성을 따지는 것으로 백트래킹은 조건에 맞지 않는 경로를 사전에 차단해 살펴봐야하는 노드의 수와 해답에 대한 경우의 수를 줄여 더 효율적인 탐색을 가능하게 합니다.

{: .box-note}
**이렇게 Backtracking으로 유망성을 따져 만들어진 경로만을 모아 만들어진 트리를 pruned state space tree라고 합니다.**

#### Backtracking이 사용되는 예시

1. 조합 가능한 모든 경우의 수 탐색
2. n-Queens Problem
3. Sum of subset Problem
4. 그래프 경로 찾기

#### 예시 코드

일반적으로 백트래킹은 아래와 같은 형태로 사용됩니다.

```python
def getRoute(k): #백트래킹을 사용해 트리를 탐색.
    global route
    if (len(route)==N):
        getDis()
        return

    for i in range(1,N):
        if i not in route:
            route.append(i)
            if promising(i): #노드의 유망성 판단
                getRoute(i+1)
            route.pop()
getRoute(0)
```

```python
def getcombination(k):  # 백트래킹을 사용해 n개로 가능한 모든 조합을 구함.
    for i in range(k, n):
        if i not in combination:
            combination.append(i)
            getcombination(i+1)
            combination.pop()

getCombination(0)
```
