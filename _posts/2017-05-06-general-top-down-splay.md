---

title: "General Top-Down Splay Trees"
date: 2017-05-06 10:31:55 -0800
categories: algorithms splay
published: true

---

# Overview

I've recently completed my senior thesis which explores the _splay tree_: a type
of binary search tree which rearranges itself whenever a lookup is done. It does
this such that insert, find, and delete can all be done in amortized logarithmic
time.

For those who are unfamiliar with amortized analysis and/or splay trees:

   * [Amortized Analysis & Splay Trees][proof]
   * [Interactive Visualization of Splay Trees][visualization]

My thesis can be found [here][thesis].


[visualization]: https://www.cs.usfca.edu/~galles/visualization/SplayTree.html
[proof]: https://www.cs.princeton.edu/~fiebrink/423/AmortizedAnalysisExplained_Fiebrink.pdf
[thesis]: /documents/thesis.pdf
