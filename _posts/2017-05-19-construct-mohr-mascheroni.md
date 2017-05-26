---

title: "Construct & The Mohr-Mascheroni Theorem"
date: 2017-05-19 22:26:25 -0800
categories: math geometry
excerpt: "Exploring Construct: a domain-specific-language for classical geometry"
published: true

---

{::options parse_block_html="true" /}

# Overview
{:.no_toc}

![Construct Logo][construct-logo]

[Construct][construct] is a domain-specific language  & interactive tool for
expressing classical geometry. In this post we learn the basics of Construct,
and in the process explore the Mohr-Mascheroni Thereom which states that any
point constructible using a compass and straightedge is constructible using the
compass alone.

# Contents
{:.no_toc}

   * I will become the TOC
{:toc}

# Classical Geometry

Geometry, the study of space & distance, was one of the first domains of
mathematics to be formalized in a form recognizable to a modern audience. In the
4th century B.C. Euclid, a greek mathematician, wrote *Elements* which
formalized the greek conception of geometry as an axiomatic system. *Elements*
showed how geometric knowledge could be derived from 5 fundamental axioms.
In modern language the axioms read:

   1. Two points may be connected by a straight segment.
   2. A straight segment may be extended indefinitely into a straight line.
   3. A circle may be drawn about one point and through another.
   4. All right angles are congruent.
   5. If two lines are not parallel they intersect precisely once.

The interesting thing about these axioms is that they're less propositional than
one might think and more... *constructive*. Euclid's axioms don't just state
5 properties of different geometric objects, three of them are dedicated to
stating how geometric objects can be constructed. In particular axioms 1, 2,
and 3 describe how segments, lines, and circles can be constructed from,
ultimately, points.[^zfc]

[^zfc]:
    It turns out that having substantial constructive character in mathematical
    axioms is not unusual. As an example, Zermelo-Fraenkel set theory is an
    axiomatic system used to discuss the most foundational and logically
    dangerous parts of mathematics. It too is constructive in nature.

This emphasis reveals that classical geometry is not only a system for proving
propositions, but also a system for construction. That is, classical geometry is
a system for constructing geometric objects (points, lines, circles, ...) with
particular relationships to one another. It is this facet of classical geometry
that Construct expresses.

# Construct

Construct is a computer langauge which allows the user to express how geometric
constructions should be done. In this section we do two constructions to give
you a sense of how the language works[^disclaimer].

[^disclaimer]:
    Don't take this to be a working introduction to the language. I'm giving you
    a taste of some exotic dish -- not the recipe for it.

## Interactive Mode

Construct begins simply --  the program starts up presenting you with a prompt
and an empty canvas. You click twice on the canvas and there are two points, $$
A $$ and $$ B $$. Perhaps you're looking for the perpendicular bisector...  Hmm,
you need two circles, their intersection, a line.

```
let C1 = circle(A, B)
let C2 = circle(B, A)
let D, E = intersection(C1, C2)
let l = line(D, E)
```

![The perpendicular bisector construction][perp]

Construct's primitives are exactly the primitives of classical geometry: the
construction of fundamental geometric objects, the ability to find their
intersections.

## Reusing Constructions

This is exciting, but a programming language is more than its primitives, in the
same way that mathematics is more than the axioms we accept. The interesting
stuff is in how primitives and axioms can be combined into new results that are
worthy of their own names... We tell Construct that we want to `:write` this
construction to a file for future use.

```
given point A, point B
return l
:write perpendicular_bisector my_constructions.con
```

"What future use?", you wonder to yourself. You recall that the *orthocenter*,
the center of a triangle's [circumcircle][circumcircle], can be found using
perpendicular bisectors. You open a text file to write the construction, but
stumble -- "What is a triangle?".

## Custom Geometric Objects

While the idea of a triangle is clear to you, triangles are not discussed by any
of Euclid's axioms, and are not primitives in Construct. Your problem is that
the idea in your mind is not primitive, but composite: a construction of
mathematicians who have come before you. You solve this problem by importing the
standard library, which defines `triangle`.

```
include lib/lib.con
include my_constructions.con

construction orthocenter
given triangle t
let triangle(A, B, C) = t
let mC = perpendicular_bisector(A, B)
let mB = perpendicular_bisector(A, C)
let O = intersection(mA, mC)
return O
```

![The orthocenter construction][ortho]

Construct allows you to manipulate composite geometric objects by
*destructuring* them into their constituents.

## Conclusion

While this has been just a fleeting taste of Construct, we've seen that the
language features geometric primitives, functional composition (construction
reuse), user-defined data (custom geometric objects), and an graphical/textual
interactive mode.

This set of features seems to capture classical geometry quite well -- most
constructions I'm aware of can be expressed using Construct's current form. One
might ask: "Are all geometric ideas expressible in Construct?" We answer this
question by discussing an old and fascinating geometric theorem.

# The Mohr-Mascheroni Theorem

Thinking of geometric construction as a system of computation brings to mind
the expansive work that's been done by computer scientists on models of
computation. Memories of Turing machines, linear automata, and finite state
automata resurface, bringing with them theorems about what problems are solvable
by which computational devices. These thoughts about computability beg a natural
set of questions: "What is the relative computational power of different
geometric toolboxes? What happens if we takeaway the straightedge? Or the
compass?"

Enter the Mohr-Mascheroni theorem. This theorem (discovered and proved
independently by Georg Mohr in 1672 and Lorenzo Macheroni in 1797) states that:

> Any point constructible by a straightedge and compass is constructible by a
> compass alone.

The theorem's central statement is fascinating, but first it requires us to look
at constructions in a new light. Often our constructions are intended to produce
certain geometric objects: the circumcircle of a triangle, the perpendicular
bisector of a segment, etc. However, Mohr-Mascheroni restricts the goal of
constructions to the placement of points. This isn't as much of a restriction as
one might think -- all the lines, circles, and other geometric objects one might
be interested in constructing are uniquely determined by a set of points (e.g.
two distinct points determine a line), so constructing complex objects can be
reduced to constructing the points which determine them.

With this understanding of the goal of a construction Mohr-Mascheroni states
that the straightedge is superfluous: the compass alone can construct any point
the compass and straightedge could construct together. One immediately wonders
-- how would you prove such a statement?

The answer is easier than you would think. Since the goal of constructions is
the placement of points, we begin by asking what points straightedge allows us
to place. The answer? That the straightedge allows us to find the intersection
between lines and other objects. Whenever we find the intersection of a line and
a circle, or between two lines, we are using a straightedge, and ultimately,
these are the *only* times we use a straightedge.

Proving Mohr-Masheroni boils down to showing that

   * the intersection(s) of a line and circle
   * the intersection of a line and line

where lines are represented by two distinct points on their extent, can be found
using only a compass. While we will not fully present each construction here,
we'll look briefly at each.

## The Intersection of a Line and Circle

Finding the intersection of a line (determined by two points on it) and circle
(determined by its center and a point on its extent) is at first glance rather
straightforward.
The construction is guided by a core strategy: reflect the circle about the
line, and the circle will intersect its reflection precisely where the circle
intersects the line. Done in construct:

```
construction line_circle_intersect
given line l, circle K
let line(C, D) = l
let circle(B, A) = K
let K2 = circle(C, B)
let K3 = circle(D, B)
let B2 = intersection(K2, K3) - {B}
let A2 = c_move(B2, segment(B, A))
let K4 = circle(B2, A2)
let F, G = intersection(K, K4)
return F, G
```
where we've used `c_move` a construction that places a point a certain distance
away from another[^prop2].

[^prop2]:
    Doing this construction without a straightedge is left to the reader, but is
    relatively *straight*-forward.

![The Line-Circle Construction][lc]

what this construction doesn't immediately show is that **it doesn't always
work**. In particular, if the center of the circle is already on the line, then
the circle reflects onto itself, its self-intersection is the entire circle, and
we fail to find the two points where the circle intersects the line.

This situation is straightforward to detect, and there is an alternate
construction that handles it appropriately. We need the ability to take stock of
the situation and chose the appropriate construction. This ability is
[branching][branch], as is typically provided to a programming language by
`if`/`else`, `match`, or `switch` statements. However, Construct has none of
these control flow structures. In fact, Construct has no idea of branching
whatsoever -- all constructions must have linear control flow.

For this reason Construct can not express the line-circle construction for the
Mohr-Macheroni theorem.

## The Intersection of Two Lines

Finding the intersection of two lines is a much more complex process. For that
reason we won't go through it here, but will instead highlight one step in the
construction: the *extension step*.

The extension step takes two segments, $$ A $$ and $$ B $$, and repeated doubles
$$ A $$ until it is longer than $$ B $$.

![Doubling A][archimedean]

The act of doubling $$ A $$ until it's longer than $$ B $$ is an example of an
*iterative process* -- a process that should be repeated until some condition is
met.
In this situation, the Archimedean Principle, which states there are no
infinitely small or large lengths, guarantees that eventually $$ A $$ will be
larger than $$ B $$. This proves that the process terminates.

However, our problem isn't whether the process terminates. Rather, the issue is
that the Construct language has no support for iteration. Programming languages
typically allow for iteration through `for`/`while` loops or recursion, both of
which are absent from Construct.

## Limitations of Construct

Exploring the Mohr-Mascheroni theorem has shown some of the limits of construct:
that it supports neither branching nor iteration. However, this exploration
could be even more important than that. Viewed in a different light, it might
reveal limitations of the Mohr-Mascheroni theorem itself.

The Mohr-Macheroni theorem is a grand statement about the computational power of
two different systems. It says that the system with a compass can do just as
much as the system with a compass and straightedge. However, it makes this
statement without addressing the *computational* character of these systems:
without any discussion of kinds of computational capabilities each of these
systems need  in order to correctly use the tools they're given.

This omission could be inconsequential -- perhaps at the end of the day our two
sets of tools can compute the same points given the same set of computational
primitives. But our work here suggests otherwise. We saw that the
Mohr-Mascheroni constructions rely on branching and iteration, computation
primitives missing from Construct. Should these primitives be included in
Construct?

I suspect the answer is no. I didn't omit them on accident, or because I was
lazy. I omitted them because all the constructions I've seen (save
Mohr-Macheroni) don't use them. It seems that iteration and branching have no
place in a Mohr-Macheroni-free world, suggesting that the classical geometric
tools (a compass and straightedge) can do with only linear control flow what the
compass requires branching & iteration to do. Essentially, the world might look
like this:


| System | Geometric Primitives | Computational Primitives | Constructive Power |
|--------|----------------------|--------------------------|--------------------|
|   A    | compass, straightedge| sequencing               | equal to B         |
|   B    | compass              | sequencing, branching, iteration | equal to A |
|   C    | compass              | sequencing               | less than A and B  |

Which is rather interesting.

# Closing Thoughts
{:.no_toc}

While I'm no longer making large changes to Construct, I'm still very curious
about the computational quandry that it has brought to light. This post glosses
over some of the details, but if you're interested in this, let me know.

If you think I'm totally off-base, let me know about that too :).

[construct]: //github.com/alex-ozdemir/construct
[circumcircle]: //mathworld.wolfram.com/Circumcircle.html
[branch]: https://en.wikipedia.org/wiki/Branch_(computer_science)

[construct-logo]: /images/2017-05-19-construct-logo.png
[perp]: /images/2017-05-20-perp.png
{: .img-ssmall .align-center}
[ortho]: /images/2017-05-20-ortho.png
{: .img-ssmall .align-center}
[lc]: /images/2017-05-21-line-circle.png
{: .img-small .align-center}
[archimedean]: /images/2017-05-26-archimedean.png
{: .align-center}
