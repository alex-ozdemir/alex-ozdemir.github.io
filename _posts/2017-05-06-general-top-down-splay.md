---

title: "General Top-Down Splay Trees"
date: 2017-05-06 10:31:55 -0800
categories: algorithms splay
excerpt: Exploring generalizations of splay trees.
published: true

---

# Overview

I've recently completed my senior thesis which explores the _splay tree_: a type
of binary search tree which uses a set of rules to rearrange itself whenever a
lookup is done. The rules are such that insert, find, and delete can all be done
in amortized logarithmic time.

Splay trees were invented in 1985, and came from the minds of Daniel Sleator and
Robert Tarjan like a rabbit comes from a magician's hat. The authors wrote a
proof that splay trees provided strong asymptotic performance gaurantees, but
the set of rules splay trees used seemed strange and arbitrary. Not only that,
but there were **two** splay trees -- a bottom-up and top-down variant -- and
the relationship between, though somewhat intuitive, was never precisely stated.

In time other researchers would explore bottom-up splay trees and describe
exactly what is was about their rules that lead to such good performance.
Unforunately, even after this further investigation the question of the
essential nature of top-down splay trees and their relationship to bottom up
splay trees remained open.

My thesis addresses these questions, and can be found [here][thesis].

For those who are unfamiliar with amortized analysis and/or splay trees:

   * [Amortized Analysis & Splay Trees][proof]
   * [Interactive Visualization of Splay Trees][visualization]


[visualization]: https://www.cs.usfca.edu/~galles/visualization/SplayTree.html
[proof]: https://www.cs.princeton.edu/~fiebrink/423/AmortizedAnalysisExplained_Fiebrink.pdf
[thesis]: /documents/thesis.pdf
