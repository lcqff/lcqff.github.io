---
layout: post
title: "[BFS] 백준 1697- 숨바꼭질"
author: pam
categories: Algorithm
sidebar: []
---

[백준 숨바꼭질](https://www.acmicpc.net/problem/1697)

### 문제
---
수빈이는 동생과 숨바꼭질을 하고 있다. 수빈이는 현재 점 N(0 ≤ N ≤ 100,000)에 있고, 동생은 점 K(0 ≤ K ≤ 100,000)에 있다. 수빈이는 걷거나 순간이동을 할 수 있다. 만약, 수빈이의 위치가 X일 때 걷는다면 1초 후에 X-1 또는 X+1로 이동하게 된다. 순간이동을 하는 경우에는 1초 후에 2*X의 위치로 이동하게 된다.

수빈이와 동생의 위치가 주어졌을 때, 수빈이가 동생을 찾을 수 있는 가장 빠른 시간이 몇 초 후인지 구하는 프로그램을 작성하시오.

### 입력
---
첫 번째 줄에 수빈이가 있는 위치 N과 동생이 있는 위치 K가 주어진다. N과 K는 정수이다.

### 출력
---
수빈이가 동생을 찾는 가장 빠른 시간을 출력한다.



<br><br>
<img width="1426" alt="N" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/ea99283c-e77d-4850-a937-d571266d9149">

문제를 읽고 위와 같은 Tree를 떠올렸다. 하나의 부모 노드가 2배, -1, +1의 값을 지니는 세개의 자식 노드를 가지는 형태의 트리이다. (이미 존재하는 노드와 동일한 값을 가지는 자식노드는 가지치기 했다.)  
위 트리를 탐색하며 K값을 가지는 노드를 찾으면 된다고 생각했다. 트리는 무한으로 이어지기에 DFS가 아닌 BFS를 사용해 Tree를 탐색했다.

총 탐색할 노드의 개수는 최대 100,001개로 O(100,001)의 시간복잡도가 소요된다.  
  
## 첫번째 코드

```java
class Node {
    int x;
    int y;

    public Node(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() {
        return x;
    }

    public int getY() {
        return y;
    }
}

public class Main {
    static int k;

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt();
        k = sc.nextInt();

        System.out.println(bfs(n));
    }

    private static int bfs(int n) {
        Queue<Node> q = new LinkedList<>();
        List<Integer> visitedNode = new ArrayList<>();
        q.offer(new Node(n, 0));
        int result = 0;

        while (!q.isEmpty()) {
            Node node = q.poll();
            int x = node.getX();
            int y = node.getY();

            if (x > 100_000 || x<0) {
                continue;
            } //범위를 벗어나는 경우
            if (visitedNode.contains(x)) {
                continue;
            } //이미 방문한 노드인 경우
            if (x== k) {
                result = y;
                break;
            } //K를 찾은 경우
            visitedNode.add(x);
            q.offer(new Node(x * 2, y + 1));
            q.offer(new Node(x + 1, y + 1));
            q.offer(new Node(x - 1, y + 1));
        }
        return result;
    }
}
```

위 코드의 제출 결과 <red>시간 초과</red>가 발생했다.  
본 문제의 시간 제한은 2초로, 대략 20,000,000번 내의 탐색이 진행될 경우 통과 가능할 터였다. 방문 가능한 노드의 범위를 `0<= x <= 100,000`으로 제한했기에 탐색하는 노드는 모두 100,000 내의 노드였을 터이기 때문에 중복 노드가 발생했다고 짐작할 수 있다.  

코드를 다시 보면 일반적인 BFS 탐색 순서와 틀린 부분이 있음을 확인할 수 있다.
<div class="captionedImg">
    <img width="868" alt="image" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/4b746856-9dd0-4663-929a-13193beb0f40">
    <p >BFS 전개 과정</p>
</div>

일반적인 BFS는 해당 노드를 방문한 이후 step에서 노드를 poll하고, poll한 자식노드를 방문 노드로 저장한다.   
그러나 위 코드는 노드를 poll함과 동시에 방문 노드로 저장하며, 그 자식 노드들은 방문 노드에 저장하지 않는다.

<img width="1640" alt="N1" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/2de36156-d2bf-4214-b950-4e25b2a8792d">
따라서 위 코드의 방문 Tree 상태가 위 이미지와 같다 판단했다.  

물론 이런 트리 형태라고 해도 최대 N 개수가 100,000개니 중복이 된다 해도 20,000,000개를 넘기 힘들지 않나? 생각했으나...ㅎㅎ 잘 모르겠다~ 

## 정답 코드
``` java
class Node {
    int x;
    int y;

    public Node(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() {
        return x;
    }

    public int getY() {
        return y;
    }
}

public class Main {
    static int k;
    static int[] visitedNode = new int[100_001];
    static Queue<Node> q = new LinkedList<>();

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt();
        k = sc.nextInt();

        System.out.println(bfs(n));
    }

    private static int bfs(int n) {
        q.offer(new Node(n, 0));
        visitedNode[n] = 1;
        int result = 0;
        if (checkNode(n,0)) return 0;
        while (!q.isEmpty()) {
            Node node = q.poll();
            int x = node.getX();
            int y = node.getY();

            if (checkNode(x*2,y+1) || checkNode(x-1,y+1) || checkNode(x+1,y+1)) return y+1;
        }
        return result;
    }

    private static boolean checkNode(int x, int y) {
        if (x > 100_000 || x<0) {
            return false;
        }
        if (x== k) {
            return true;
        }
        if (visitedNode[x] != 1) {
            q.offer(new Node(x, y));
            visitedNode[x] = 1;
        }
        return false;
    }
}
```

일반적인 BFS 탐색과 동일하게 노드를 방문한 이후 Stack에서 poll해주며 자식 노드들을 방문노드에 저장하는 방식으로 바꾸었다.  통과~