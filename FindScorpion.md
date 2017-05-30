---
layout: default
---

## Find a Scorpion


### Problem Description

A scorpion on n vertices is a graph that has a vertex of degree 1 (the tail), connected to a vertex of degree
2 (the body), connected to a vertex of degree n-2 (the head). The other n-3 vertices (the feet) can be
arbitrarily interconnected.

Give an O(n) algorithm for deciding whether or not an arbitrary graph is a scorpion, assuming that the
graph is represented by an adjacency matrix.


### Algorithm

Let M[i][j] i = 1, 2, ..., n;  j = 1, 2, ..., n; represents the adjacency matrix.

If M[i][j] == 0 represents there is no direct connection between i and j and M[j][i] must be zero.

If M[i][j] == 1 represents there is a direct connection between i and j and M[j][i] must be one.

Pick the first row M[1][j] j = 1, 2, ..., n;

*	First condition: the first row is part of the scorpion, which means the first row is either head, body or tail.

We traverse the first row and count the zeros' number, store it in CZero. We also count the ones' number and store in COne.



[back](./)
