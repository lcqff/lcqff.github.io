---
layout: post
title: "[BFS] 백준 1697- 숨바꼭질"
author: 팜
categories: Algorithm
sidebar: []
---

[백준 숨바꼭질](https://www.acmicpc.net/problem/1697)



```
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
            }
            if (visitedNode.contains(x)) {
                continue;
            }
            if (x== k) {
                result = y;
                break;
            }
            visitedNode.add(x);
            q.offer(new Node(x * 2, y + 1));
            q.offer(new Node(x + 1, y + 1));
            q.offer(new Node(x - 1, y + 1));
        }
        return result;
    }
}
```