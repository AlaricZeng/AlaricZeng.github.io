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

#### First we check if the first row is part of the scorpion

We traverse the first row and count the zeros' number, store it in _CZero_. We also count the ones' number and store in _COne_. We also record the first appear zero as _FZero_ and first appear one as _FOne_.

##### If COne equals to one

Then the first row could be the tail(not necessary, just maybe) 
So we check M[_FOne_][j] j = 1, 2, ..., n; to see if the counting of one equals to two(One should be M[_FOne_][1] and other let it be M[_FOne_][_SOne_]). Check if zeros' number in M[_SOne_][j] j = 1, 2, ..., n is n - 2. If it is then we find the scorpion

#### If COne equals to two

Then the first row could be the body(not necessary, just maybe) 



[back](./)
