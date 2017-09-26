---

title: "Revisiting the Mohr-Mascheroni Theorem"
date: 2017-09-25
categories: math geometry
excerpt: "A compass is not as powerful as a compass & straightedge after all"
published: true

---

{::options parse_block_html="true" /}

# Contents
{:.no_toc}

   * I will become the TOC
{:toc}

# Introduction

The Mohr-Mascheroni theorem (independently proved in 1672 and 1797) makes an
astounding statement: that any point constructible with a compass and
straightedge can be constructed with the compass alone. It treats the tools for
classical geometric constructions as a system for computation: a *geometric*
system of computation capable of placing points at desired locations.

There's a long history of analyzing the power of *numerical* systems of
computation. You might think that the fundamental determiner of the power of a
numerical computer is the kinds of atomic numeric operations it can perform.
It's tempting to say that a computer with multiplication should be fundamentally
more powerful than one with only addition.

However, differences in atomic numeric operations end up being far less
important than differences in the kinds of control-flow afforded by the
computational system. For example, a [primitive recursive][prim-rec] system that
can increment integers is capable of computing sums, products, powers, and
factorials, but it is incapable of computing the [Ackermann function][ackermann]
and this limitation cannot be overcome by adding any new atomic numeric
operations to the system. Ultimately it is not differences in the availability
of numeric operations like addition and multiplication that give rise to
differences in numerical computational power: it is differences in control-flow
constructs like if-statements and loops. Primitive recursion is not limited
because it has no atomic operation for multiplication, but because it does not
allow unbounded loops.

Returning to geometric systems of computation, the Mohr-Mascheroni theorem seems
to be stacking two geometric systems against each other, and comparing their
computational power.

| System A     | System B      |
|--------------|---------------|
| compass      | compass       |
| straightedge |               |

However, this comparison is done without any regard for the kinds of
control-flow constructs available in the two systems. The history of differences
in control-flow constructs determining the computational power of numerical
computers hints that this may be an oversight, and this post serves to correct
that oversight.

# The Primitives of Classical Geometry

We begin by asking what control-flow primitives a geometric system should have.
[This post][construct-post] explores a programming language, Construct, for
classical geometry that allows for non-recursive functions that contain
sequences of operations. Notably it does not included if-statements, and more
importantly it includes **no** kind of loops. The justification for these
choices is that classical geometric writing (e.g. Euclid) includes sequences of
steps and functions (propositions like "given a triangle, construct the
orthocenter"), but little branching and no examples of repeatedly doing the
same thing (draw $$ n $$ segments). Construct also clarifies that doing geometry
isn't just about using a compass or straightedge to draw shapes, it's about
finding the `intersection` of those shapes as well, to fix the locations of new
points.

This allows us to more clearly specify the two geometric computational systems
whose powers we'll be comparing:

| System A     | System B      |
|--------------|---------------|
| compass      | compass       |
| straightedge |               |
| intersection | intersection  |
| sequencing   | sequencing    |
| functions    | functions     |

Functions serve to encode constructions themselves. A construction like "given
a triangle construct its orthocenter" would be encoded as a function that takes
a triangle (represented, perhaps, by it's three corners) and returns its
orthocenter (the point equidistant from the three corners).

If the two computational systems are equally powerful, then any function
writable using system A should also be writable using system B. We claim that
the systems are in fact not equally powerful, that there is a function
(construction) writable with system A that is not writable with system B.
Specifically, the construction that takes points $$A$$, $$B$$, $$C$$, and $$D$$
and returns the intersection of lines $$AB$$ and $$CD$$.

In system A, which provides a straightedge to draw lines directly, the
construction is *straight*forward:

```
construction line_line_intersection
given points A, B, C, D
let line1 = line(A, B)
let line2 = line(C, D)
let P = intersection(line1, line2)
return P
```

However, writing such a construction without using the straightedge (i.e.
without using `line`) will prove impossible.

# Line-Line Intersections With Only Circles

Our analysis of whether the intersection of two lines can be computed in
Construct using only circles & intersections will be based on one critical idea:
the *diameter* of a construction, which we will define in a moment.

When a construction begins, some set of points are taken as givens into the
construction. Perhaps that set is a pair of points, and you seek to find a point
halfway between them. Perhaps that set is a triangle, and you seek to find the
orthocenter.

As the construction progresses more points are added to the construction.
Perhaps a circle is drawn, which adds uncountably many new points. Perhaps an
intersection between circles is found, which adds no new points (but labels
some).

Throughout this process, the *diameter* of the construction is the maximum
distance between any pair of points in the construction. Since points may be
added to a construction as it progresses, the diameter may grow over time; it
may never shrink.

Importantly, the only operation that increases the diameter is drawing new
circles, and because a circle must be drawn from its center and a point on its
edge, doing so can increase the diameter of the construction by at most a factor
of two (triangle inequality). Because constructions are sequences of
instructions (e.g. draw circle) of finite length without loops or recursion, any
given construction can only draw a finite number of circles, and thus the
diameter of a given construction can only grow by at most a finite factor.

Now, we suppose (to obtain a contradiction) some construction `con` can take any
points $$A$$, $$B$$, $$C$$, and $$D$$ and find the intersection of $$AB$$ and
$$CD$$.  That construction could increase the diameter by at most some factor of
$$N$$.  Thus if $$m$$ is the maximum of the distances between any pair from $$
\{A, B, C, D\} $$ then the construction `con` could construct no point further
than $$ Nm $$ from point $$ A $$, and thus return no point further than $$Nm$$
from point $$A$$. Roughly speaking, this means no construction can return a
point arbitrarily far away from its starting points.

Now consider the following points $$ A = (0, 1), B = (1, 1), $$ and $$ C = (0,
0)$$ Also consider the sequence of points $$ D_n = \left(1, \frac{1}{n}\right)
$$. Notice that the intersection of $$ AB $$ and $$ CD_n $$ is $$ (1, n) $$.

Thus when `con` correctly finds the intersection of $$ AB $$ and $$ CD_n $$ that
intersection is at $$ (0, n) $$, which is further than $$n$$ from $$A$$. Also notice
that the maximum distance between any of $$ \{A, B, C, D_n\}$$ is always less
than $$ 2 $$, so `con` can never find (or return) a point further than $$2N$$
from $$ A $$. But $$ n $$ can be any integer, in particular, $$ 2N $$, so then
`con` would have to return a point further than $$ 2N $$ from the $$ A $$, a
contradiction.

Our only assumption was that such a construction `con` existed, so in fact that
assumption must have been wrong -- there is no construction that computes the
intersection of lines using only circles. Thus the `line_line_intersection`
construction can be written in system A, but not system B, proving that A is
more powerful.

# Conclusion

We began by building a case for considering the control-flow constructs
available to a geometric computational system. We then describe a reasonable set
of such constructs (sequencing and non-recursive functions) and then showed
that given that set of control-flow constructs, a geometric computer with a
compass does not allow for a generic function to be written which finds the
intersection of any two lines, given as points $$ A, B $$ and $$ C, D $$. In
particular, this shows that in a control-flow aware sense, a compass is not
always as computationally powerful as a compass and straightedge together.

# Credits and Future Work

Jackson Warley and I figured out this argument together one weekend after
Shīyuè Lǐ pointed me towards an algebraic treatment of geometric constructions
the weekend before. Ironically, our final argument uses no algebraic machinery.

This argument considers the reduction in power produced by removing a
straightedge from a geometric computational system when that system uses only
sequencing and non-recursive functions as its control-flow structures. Geometric
systems with other control-flow structures remain an open question: What happens
if you give a finite state machine a compass?

[construct-post]: /posts/2017-05-19-construct-mohr-mascheroni
[ackermann]: https://en.wikipedia.org/wiki/Ackermann_function
[prim-rec]: https://en.wikipedia.org/wiki/Primitive_recursive_function
