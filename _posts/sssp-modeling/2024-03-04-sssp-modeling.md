---
layout: post
title:  Single Source Shortest Path Modeling
date:   2024-03-04
tags: cp graph-theory
usemathjax: true
---

A few weeks ago, I stumbled upon [CF 1915G](https://codeforces.com/contest/1915/problem/G) while doing 
some practice. The problem statement is as follows:

> You are given an undirected, weighted, graph, with \\(n\\) vertices and \\(m\\) edges (\\(2 \leq n \leq 1000, n - 1 \leq m \leq 1000\\)).
At vertex \\(i\\), you are given a bicycle with slowness \\(s_i\\) (\\(1 \leq s_i \leq 1000\\)).
Suppose you are traversing over edge \\((u, v)\\), and let \\(w_{u, v}\\) be the weight of that edge.
The total time taken to cross this edge is \\(s \cdot w_{u, v}\\) ,
where \\(s\\) can be any bicycle you have after the path traversal to $ u $
in this graph.
Find the shortest path from vertex \\(1\\) to vertex \\(n\\). It is guaranteed that this graph is connected.


The first obvious observation here is that it is always optimal to use the fastest
bike that has been collected.
So in other words, it is sufficient to know some information about the path, 
to a given vertex.
If our graph was a DAG, this would be trivial with DP.
Although we don't have an easy constraint like that, an idea similar to that of a DP state can be used here. 

#### Graph Modeling

Suppose we had a graph consisting of every vertex-bike pair, where,
each bike represents the fastest bike in the shortest path to a given vertex.
Assuming that the edge-set of this graph can be derived from the original
graph's edge set, finding the shortest path on the original graph 
from $ 1 \rightarrow n $
is the same as finding the shortest path from $ (1, s_1) \rightarrow (n, s_i) $,
for any possible $ s_i $.
Now if we can construct this graph, we can easily run Dijkstra's and compute
the answer.

If we are at vertex $ (u, s) $, and there exists an edge $ (u, v) $
in the first graph, it is obvious there exists some edge between $ u $ and $ v $
in the new graph.
The weight of this edge must be $ = w_{u, v} \cdot s $, as $ s $ 
is the fastest bike on a path to $ u $.
Then, the fastest bike on a path to $ v $ will either be the bike at $ v $,
or the fastest bike on a path to $ u $; in other words $ \min(s, s_v) $.
Thus every edge will be in the form $ ((u, s), (v, \min(s, s_v))) $.
Implementation-wise, it is a bit tedious to create an entirely new adjacency
list for this new graph.
Instead, consider every vertex in the new graph as a "state", with
the path length appended to the state in a similar fashion to Dijkstra's.
Given state $ (u, s, d) $, we only need to consider states that can be made
from vertices adjacent to $ u $ in the original graph.
Then, for any $ v $ adjacent to $ u $, the new state is
$ (v, \min(s, s_v), d + w_{u, v} \cdot s) $.
This simple modification makes the code implementation nearly identical
to standard Dijkstra's.

Here is my implementation:

```cpp
#include <bits/stdc++.h>
using namespace std;
 
#define int long long
#define pi pair<int, int>
#define vt vector
#define vi vt<int>
#define vs vt<string>
#define vb vt<bool>
#define rep(i, n) for(int i = 0; i < n; i++)
#define all(x) x.begin(), x.end()
#define f first
#define s second
#define pb push_back
 
template<class T> using pq = priority_queue<T>;
template<class T> using pqg = priority_queue<T, vector<T>, greater<T>>;
 
template<class T> bool ckmin(T& a, const T& b) { return b < a ? a = b, 1 : 0; }
template<class T> bool ckmax(T& a, const T& b) { return a < b ? a = b, 1 : 0; }
 
void setIO(string name = "") {
  cin.tie(0)->sync_with_stdio(0);
  if (name.size()) {
    freopen((name + ".in").c_str(), "r", stdin);
    freopen((name + ".out").c_str(), "w", stdout);
  }
}
 
struct State {
  int u, b, d;
 
  State(int u_, int b_, int d_) : u(u_), b(b_), d(d_) {}
 
  bool operator>(const State& s) const {
    return d > s.d;
  }
};
 
const int maxs = 1005;
 
void solve() {
  int n, m; cin >> n >> m;
  vt<vt<pi>> adj(n);
  rep(i, m) {
    int u, v, w; cin >> u >> v >> w;
    u--; v--;
    adj[u].pb({v, w});
    adj[v].pb({u, w});
  }
  vi b(n);
  rep(i, n) cin >> b[i];
  vt<vi> dist(n, vi(maxs, 1e18));
  pqg<State> pq;
  pq.push(State(0, b[0], 0));
  dist[0][b[0]] = 0;
  while(pq.size()) {
    State s = pq.top();
    pq.pop();
    if(dist[s.u][s.b] != s.d) continue;
    for(auto [v, w] : adj[s.u]) {
      if(ckmin(dist[v][min(b[v], s.b)], w * s.b + s.d)) {
        pq.push(State(v, min(b[v], s.b), w * s.b + s.d));
      }
    }
  }
  int ans = 1e18;
  for(int i = 0; i < maxs; i++) ckmin(ans, dist[n - 1][i]);
  cout << ans << endl;
}
 
signed main() {
  setIO("");
  int t; cin >> t;
  while(t--) {
    solve();
  }
}
```

#### CF 1473E

Although the previous problem was just a
relatively straight forward implementation,
I found the whole "Graph Modeling" idea pretty interesting.
This resulted in me looking for similar problems. 
I then found [CF 1473E](https://codeforces.com/contest/1473/problem/E), which
was definitely a step up in difficulty (1800 to 2400) compared to the previous problem.
The problem statement is as follows

> You are given a weighted, undirected, connected graph, with $ n $ vertices and
$ m $ edges $ (2 \leq n \leq 2 \cdot 10^5; 1 \leq m \leq 2 \cdot 10^5) $.
The weight of the $ i $-th edge is defined as $ w_i $. Let the weight of a 
k-length walk $ e_1, e_2, \dots , e_k $ be 
$ \sum\limits_{i=1}^{k}{w_{e_i}} - \max\limits_{i=1}^{k}{w_{e_i}} + \min\limits_{i=1}^{k}{w_{e_i}} $
Find the minimum weight walk from $ 1 $ to the $ i $-th vertex, for each $ i $
$ ( 2 \leq i \leq n ) $.

*Note that I use "walk" rather than "path" here, contrary 
to the original problem statement.
The intended solution allows vertices to be repeated, so the use of "path"
would (generally) be considered incorrect wording.*

Let us consider a more generalized form of this problem: suppose that you
must add and subtract an edge weight exactly once, in a walk. 
How would you find the minimum weight walk? Well, if we are forced to subtract 
an edge weight, it is obvious that we should choose the edge with the 
maximum weight.
Likewise, we should add the minimum weight edge. 
Thus, this generalized problem is analogous to the first problem, and
finding the solution to the second problem gives the solution to the first.
Looking at the problem with this new perspective gives way to constructing
a solution using graph modeling. Let our state be $ (u, p, m, d) $, 
where $ u $ is the current vertex, $ p $ denotes whether we have added an edge,
$ m $ denotes whether we have subtracted an edge, and $ d $ is the standard
Dijkstra's path distance. Now there are four implicit edges that are induced
from every edge in the original graph. 

<ol>
    <li> 
        Traversing $ (u, v) $ normally
        <ul>
             $ (u, p, m, d) \rightarrow  (v, p, m, d + w_{u, v}) $
        </ul>
    </li>
    <li>
        Adding $ (u, v) $. This is only possible if $ p = 0 $ in the first state.
        <ul>
            $ (u, 0, m, d) \rightarrow (v, 1, m, d + 2 \cdot w_{u, v}) $
        </ul>
    </li>
    <li>
        Subtracting $ (u, v) $. Only possible if $ m = 0 $ in the first state.
        <ul>
            $ (u, p, 0, d) \rightarrow (v, p, 1, d) $
        </ul>
    </li>
    <li>
        Adding and subtracting $ (u, v) $. This is a bit weird, but accounts
        for walks where the walk length is of size $ 1 $.
        <ul>
            $ (u, 0, 0, d) \rightarrow (v, 1, 1, d + w_{u, v}) $
        </ul>
    </li>
</ol>

Now we have found all the potential edges required when implicitly traversing
the new graph. 
Just like in the first problem, the implementation is nearly identical to 
standard Dijkstra's.
Thus, the shortest walk from $ 1 $ to $ i $ is simply just $ dist[i][1][1] $.

My code implementation:

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define pi pair<int, int>
#define vt vector
#define vi vt<int>
#define vs vt<string>
#define vb vt<bool>
#define rep(i, n) for(int i = 0; i < n; i++)
#define all(x) x.begin(), x.end()
#define f first
#define s second
#define pb push_back

template<class T> bool ckmin(T& a, const T& b) { return b < a ? a = b, 1 : 0; }
template<class T> bool ckmax(T& a, const T& b) { return a < b ? a = b, 1 : 0; }

template<class T> using pq = priority_queue<T>;
template<class T> using pqg = priority_queue<T, vector<T>, greater<T>>;

template<class T, class... Args>
auto create(size_t n, Args&&... args) {
	if constexpr(sizeof...(args) == 1)
		return vector<T>(n, args...);
	else
		return vector(n, create<T>(args...));
}

void setIO(string name = "") {
  cin.tie(0)->sync_with_stdio(0);
  if (name.size()) {
    freopen((name + ".in").c_str(), "r", stdin);
    freopen((name + ".out").c_str(), "w", stdout);
  }
}

struct State {
  int u, pl, mi, d;

  State(int _u, int _p, int _m, int _d) : u(_u), pl(_p), mi(_m), d(_d) {}

  bool operator>(const State& s) const {
    return d > s.d;
  }
};

void solve() {
  int n, m; cin >> n >> m;
  vt<vt<pi>> adj(n);
  rep(i, m) {
    int u, v, w; cin >> u >> v >> w;
    u--, v--;
    adj[u].pb({v, w});
    adj[v].pb({u, w});
  }
  auto dist = create<int>(n, 2, 2, 1e18);
  dist[0][0][0] = 0;
  pqg<State> pq;
  pq.push(State(0, 0, 0, 0));
  while(pq.size()) {
    State s = pq.top();
    pq.pop();
    if(dist[s.u][s.pl][s.mi] < s.d) continue;
    dist[s.u][s.pl][s.mi] = s.d;
    for(auto [v, w] : adj[s.u]) {
      if(ckmin(dist[v][s.pl][s.mi], s.d + w)) {
        pq.push(State(v, s.pl, s.mi, s.d + w));
      }
      if(!s.pl && ckmin(dist[v][1][s.mi], s.d + 2 * w)) {
        pq.push(State(v, 1, s.mi, s.d + 2 * w));
      }
      if(!s.mi && ckmin(dist[v][s.pl][1], s.d)) {
        pq.push(State(v, s.pl, 1, s.d));
      }
      if(!s.pl && !s.mi && ckmin(dist[v][1][1], s.d + w)) {
        pq.push(State(v, 1, 1, s.d + w));
      }
    }
  }
  for(int i = 1; i < n; i++) {
    cout << dist[i][1][1] << " ";
  }
  cout << endl;
}

signed main() {
  setIO("");
  solve();
}
```

#### References
1. [https://codeforces.com/blog/entry/45897](https://codeforces.com/blog/entry/45897)
2. [https://codeforces.com/blog/entry/70589](https://codeforces.com/blog/entry/70589)
