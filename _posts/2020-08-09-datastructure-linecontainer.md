---
layout: post
title: "Convex Hull Trick"
date: 2020-08-09 21:11
category: algorithm
author: zzz
tags: [Data Structure, Line Container, DP Optimization, Convex Hull Trick]
summary: A data structure that can supports querying maximum of multiple linear functions dynamically.
---

## Meta Problem

Given a set of linear functions `y = ai * x + bi`, get the maximum of y for any given x.

## Observations

![Example1](https://codeforces.com/predownloaded/06/b1/06b12912502c14c161d0ec6c21f7d3118652fe53.png)

From[1].

In the example, it can be found that,
1. the maximum is contributed by a collection of segments forming a convex hull.
2. from left to right, the slope in the critical segments collection is increasing, and the intercept is decreasing.

## Data Structure

### Code Library
Here is a data structure that can easily solve the problem.
```c++
struct Line {
	mutable ll k, m, p;
	bool operator<(const Line& o) const { return k < o.k; }
	bool operator<(ll x) const { return p < x; }
};

struct LineContainer : multiset<Line, less<>> {
	// (for doubles, use inf = 1/.0, div(a,b) = a/b)
	const ll inf = LLONG_MAX;
	ll div(ll a, ll b) { // floored division
		return a / b - ((a ^ b) < 0 && a % b); }
	bool isect(iterator x, iterator y) {
		if (y == end()) { x->p = inf; return false; }
		if (x->k == y->k) x->p = x->m > y->m ? inf : -inf;
		else x->p = div(y->m - x->m, x->k - y->k);
		return x->p >= y->p;
	}
	void add(ll k, ll m) {
		auto z = insert({k, m, 0}), y = z++, x = y;
		while (isect(y, z)) z = erase(z);
		if (x != begin() && isect(--x, y)) isect(x, y = erase(y));
		while ((y = x) != begin() && (--x)->p >= y->p)
			isect(x, erase(y));
	}
	ll query(ll x) {
		assert(!empty());
		auto l = *lower_bound(x);
		return l.k * x + l.m;
	}
};
```

From [2].

### Detailed Analysis

#### Line

```c++
struct Line {
	mutable ll k, m, p;
	bool operator<(const Line& o) const { return k < o.k; }
	bool operator<(ll x) const { return p < x; }
};
```

The defination of `Line`, with `k` denoting the slope, `m` denoting the intercept, and `p` is the x-axis of right end intersection in the convex hull segment collection. In the convex hull collection, it's easy to know that `p` is increasing, as well as `k`.
So, `Line` also has two overloaded `<` operators, which is to support multiindexing set.

#### LinerContainer::query

```c++
	ll query(ll x) {
		assert(!empty());
		auto l = *lower_bound(x);
		return l.k * x + l.m;
	}
```

With `p` explained, and the multiset having multi-indexes, one being the slope and one being the intersection, it's not very hard to see that for a given x, we need to find out the line with smallest p that is >= x.

#### LineContainer::isect

```c++
	bool isect(iterator x, iterator y) {
		if (y == end()) { x->p = inf; return false; }
		if (x->k == y->k) x->p = x->m > y->m ? inf : -inf;
		else x->p = div(y->m - x->m, x->k - y->k);
		return x->p >= y->p;
	}
```

Given two lines, `y = k1 * x + m1` and `y = k2 * x + m2`, the x-axis of the intersect is `(m2 - m1) / (k1 - k2)`.
The return value here is true if the current intersection is no less than the next intersection.

Consider the following case, the first line is the red line, the second is the blue one, and the last is the green line.
![](https://zizhong.github.io/assets/img/insert_line_1.png)
`p` of the red line is less than `p` of the blue line. Therefore, we need to keep all these lines.

However, in the second case,
![](https://zizhong.github.io/assets/img/insert_line_2.png)
`p` of the red line is larger than `p` of the blue line. Therefore, we need to remove the blue line.

So that, if `isect` returns true, we need to erase `y`.

#### Min Hull

If we take the symmetry of the lines by the x-axis, we can get the minimum of the line collections. 

### Application 1: [Codeforces][1083 E][The Fair Nut and Rectangles]

[code](https://codeforces.com/contest/1083/submission/89193199)


### Application 2: [Codeforces][1388 E][Uncle Bogdan and Projections]

[code](https://codeforces.com/contest/1388/submission/89304569)

## Looking Back - Combinatorics based Learning

Learning new stuff pieces by pieces and steps by steps. A new area is a combination of old knowledges. Learning can also be solved by combinatorics.

## References
[1] [[Tutorial] Convex Hull Trick — Geometry being useful](https://codeforces.com/blog/entry/63823) by **meooow**.

[2] [Line Container Library](https://github.com/kth-competitive-programming/kactl/blob/master/content/data-structures/LineContainer.h) by **kth**.