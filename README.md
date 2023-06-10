# DelauneyCaseStudy

A case study into the Delauney Triangulation, its use cases, and different implementations across languages.
Some understanding of geometry may be required. Particularly the concept of convex hulls and simplices.

Our goal is a parallelizeable, largely asynchronous and distributable method to Delauney Triangulation.

## Proofs about Triangulation

Here are a set of known facts and assumptions that help in formulating an efficient, functional approach to Delauney Triangulation.

### Assumptions

1. We are working in two dimensions
1. We are provided a collection of points that are pre sorted across both axes. (In our examples, x then y)

### Facts

Here are some facts (or 'facts') that we can use to help us in creating our algorithm(s).
Some of these are formally proven or inferable elsewhere but the actual proving is out of scope.
If you can find a demonstrably provable counter-example PR it or open an Issue and I'll adjust.

1. When densely connected, any collection of three or fewer points are in valid Delauney Triangulation (two and one points are considered degenerate tris but are still "valid")
1. If a hull A does not overlap a hull B, then hull A does not overlap the hull of any subset of points within B.
1. If a sorted set of points S is split into two adjacent sorted subsets A and B, then the hull of these sets A & B do not overlap
1. Merging overlapping hulls is possible but costly.
1. Two Triangulations whose hulls' do not overlap can be merged such that:

- We only need to evaluate the points on or directly adjacent to their hulls
- We only add edges that would bridge the two triangulations
- We only remove edges that would exist on the prior hulls

### Algorithm 1: Basically synchronous

Lets say we have a Triangulation of zero points and call it T. (Silly I know but roll with it)
Given a sorted set of points $P$,

Emit the first point and pop it from the set. That one singular point is inherently in Delauney triangulation. Lets call it t (little t, distinct from T).
T and t come from disjoint adjacent subsets of a sorted superset thus they don't overlap.
T and t don't overlap so we can merge them and call the new triangulation T.

Emit and pop a second point from our set. That point is also inherently in Delauney triangulation. Let's call it t.
T and t come from disjoint adjacent subsets of a sorted superset thus they don't overlap.
T and t don't overlap so we can merge them and call the new triangulation T.

Emit and pop a third point from our set. That point is also inherently in Delauney triangulation. Let's call it t.
T and t come from disjoint adjacent subsets of a sorted superset thus they don't overlap.
T and t don't overlap so we can merge them and call the new triangulation T.

Emit and pop a-
Wait this is just O(n) time with respect to O(merge).

Lets make an improvement.

### Algorithm 1.1: Still synchronous

The smallest collection of points that are inherently in Delauney Triangulation is a collection of three points.

Lets say we have a Triangulation of zero points and call it T.

Given a sorted set of points $P$,

Emit the first **THREE** points and pop them from the set. Tose three points are inherently in Delauney Triangulations. Lets call them t.
T and t come from disjoint adjacent subsets of a sorted superset thus they don't overlap.
T and t don't overlap so we can merge them and call the new triangulation T.

Emit the second group of **THREE** points....

It's a small improvement and we can asynchronously form these triplets. Rather than worrying about individual points, we can
create _futures/promises_ that promise $n = \lceil \frac{P}{3} \rceil$ adjacent sets of tris.

However we are constantly dependent and waiting on the merge between T and t. What if after emitting and beginning work on the first pair of tris, we emit another pair and start merging those.
While **THAT** pair of tris are merging we emit _another_ pair and...

### Algorithm 1.2: Divide and conquer

Given a sorted set of points $P$,
We asynchronously create $n = \lceil \frac{P}{3} \rceil$ adjacent sets of tris. (or futures/promises for these)

While the first pair of tris are merging, lets queue up the second pair. While the second pair are merging lets queue up the third pair...

Now we have $\lceil \frac{n}{2} \rceil$ tris. If we repeat this process we will eventually reach a final triangulation in O(n log n) merges.

## Summary ?

This is essentially how the typical divide and conquer approach works for Delauney Triangulation albeit in a future/promise dependency graph rather than
the more common recursive method.

Given subsets A,B,C,D, merge A&B and C&D into AB and CD, then merge AB&CD into ABCD. Are we done?
Not yet, more to come.
