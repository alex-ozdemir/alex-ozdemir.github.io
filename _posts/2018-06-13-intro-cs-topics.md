---

title: "Student Opinions on Intro CS Topics"
date: 2018-07-17
categories: education cs
excerpt: "Data and reflection after teaching introductory computer science at
Loomis Chaffee"
published: true

---

{::options parse_block_html="true" /}

# Contents
{:.no_toc}

   * I will become the TOC
{:toc}

# Background

I chose to take a gap year before starting graduate school and spent that year
teaching computer science and math at [Loomis Chaffee][loomis]: a small boarding
school in Windsor, Connecticut. It was a great decision. I got some teaching
experience, learned from excellent mentors (in particular [Hudson
Harper][hudson]), observed the cultural phenomenon of East Coast boarding
schools, and lived through a real winter!

One of my favorite parts of the job was building and teaching an introductory
computer science class. In this article I want to share some data about student
opinions on the curriculum of that class, and reflect on the conclusions that
lie within.

# The Curriculum

My formal introduction to computer science was through a survey of computational
ideas and programming paradigms. This approach is *very* successful at Harvey
Mudd, and it had a strong influence on my course. I went into the year with a
few guiding ideas:

   * to explore different computational ideas and paradigms
   * to do a lot of projects
   * to focus on creative problem solving (although toned down from Mudd's
       standards)
   * to spend some time doing web programming

This lead to the following curriculum, split over three terms:

   * Fall: Programming Bootcamp
      1. Python (emphasis on functional programming)
   * Winter: Inside the Computer
      1. digital circuits
      2. assembly
      3. Arduino
   * Spring: Types and Web Applications
      1. the Elm programming language
      2. [reactive][reactive] front-end web applications

Each unit featured some kind of test, and each term included an individual,
student-proposed project.

# Student Opinions


The general opinion was that the class was interesting but challenging.
Throughout the year I heard the following (paraphrased, in some cases):
   * "This class's homework takes many times longer than my assignments for
       other classes"
   * "I really think your expectations are too high"
   * "Last unit was really interesting, but so is this one"
   * "I was really proud of my work for [project X, Y and Z]"

These comments, and other information about the course as whole, are very
interesting to me, but less useful for other instructors[^changes]. However, I
also asked the students about their relative opinions on the different units
within the course, which might be more broadly applicable.

[^changes]:
    All in all it seemed to be a time-intensive but potentially rewarding
    course. If I were to teach it again I'd preserve most of the core structure,
    while focussing my attention on smoothing out some of the rougher homework
    assignments and enhancing the scaffolding for the homework to help students
    who are struggling get back in the game.

## Data Collection

At the end of the course I asked the students to rank each of the five units we
covered (python, digital circuits, assembly, arduino, and Elm)[^lastunit] in
terms of how interesting they were, how challenging they were, and how well the
students ended up understanding the material.

[^lastunit]:
    I taught a sixth unit: finite automata and computability, but this
    unit is omitted from the data because most of my class graduated halfway
    through it.

I converted the rankings into numbers by giving a unit a 1 in some category if
it was ranked last in that category, and a 5 if it was ranked first. The results
are shown graphically below. 

## Interest

{: .justify-center }
<div id="interest-chart"></div>

I found the results here quite surprising. My impression from teaching the
course was the students really enjoyed working with Python (because the language
allowed them to say what they were thinking) and Arduino (because they liked
the physicality of it). On the other hand, they seemed very frustrated with Elm
much of the time. However, it seems that regardless of these different levels of
perceived enjoyment, the class found Elm extremely interesting. Possible
explanations include the inherently visual nature of web applications, the
complexity of the applications that the students built using Elm, or perhaps
just that Elm was the last unit covered.

Assembly is the clear loser here, although a few students added notes saying that
they thought is was certainly a valuable unit. Somewhat surprising was the low
ranking of Python. It was certainly true that our Python programs were less
sophisticated, and a few students added that they had some prior Python
experience.

## Difficulty

{: .justify-center }
<div id="difficulty-chart"></div>

These results were also bit surprising. Arduino ranks low -- expected since that
unit emphasized experimentation and precision over conceptual difficulty. But
Elm falls in the middle, which I found surprising.

Finding assembly at the top of the list might strike some as unusual, but our
assembly unit was fairly complete. It involved a proper treatment of function
calls, including tail-optimized recursion in one of the class sections.[^tail]
Barely above assembly is the digital circuits unit. Finding both assembly and
circuits at the bottom of the list is intriguing because those two units struck
me as the units that were taught in a manner most typical to that of a typical
high school class: class-time was spent on lectures, and there were tests with
reasonable frequency. Perhaps this is further evidence that lecture-driven
classes aren't a good idea in high school.

[^tail]:
    To my incredible surprise, when asked to write a recursive function in
    assembly on the test (Euclid's algorithm for the greatest common divisor),
    one student wrote a tail-optimized version, even though that was not even
    suggested.

## Understanding


{: .justify-center }
<div id="understanding-chart"></div>

If this chart looks familiar, then it should -- it's nearly a mirror image of
the chart showing difficulty. I had originally asked about both difficulty and
understanding because I wanted to see if there were any situations were students
found a topic difficult, but because of the class and/or their own determination
they were able to master it anyway.

The sole deviation from the mirror-image of the last chart is the
Python/Arduino switch: Python was (barely) ranked as harder, while also (barely)
ranked as more understood. The difference probably isn't significant, but it can
be explained all the same: Python was indeed a deeper unit, but we spent a *lot*
of time on it.

# Conclusion

During the final days of the year I sat down with Hudson (my mentor teacher, who
taught the fall term with me) and we discussed whether the curriculum should
change next year, and if so how. We entertained a wide range of potential
changes, but in the end there were only two changes we seriously considered:

   * Should we use a different language to teach web programming?
   * Should we slow down the class a bit by dropping a small topic?

Given the apparent success of Elm, along with the solid pedagogical reasoning
behind its use[^elm], we decided to keep using it as our primary tool for web
programming. Some other language might have a gentler learning curve, but we
liked to the way that Elm forced us to teach not just web technology, but also
clean application structure. In fact, we wanted to devote even more time to
that by doing the quick HTML/CSS intro in the winter instead of the spring. We
decided that circuits would be the best section to drop.

[^elm]:
    The reasoning behind using Elm was manifold. First it enforces a clean clean
    application structure which emphasises application state, transformations of
    state in response to events, and the graphics as a function of state; this
    way of thinking about applications is extremely useful and is growing in
    popularity (`reactjs`, etc). Elm also follows in the tradition of the
    highly-functional, powerfully-typed languages, which all programmers should
    eventually be exposed to.

Of course, all plans are tentative. I moved back to California in June, and next
year there will be a new computer science teacher with their own plans and
ideas.[^swap] In the fall the winds of change will blow through Windsor,
Connecticut once again.

[^swap]:
    Serendipitously enough the new hire is a young man who recently graduated
    from Stanford -- the school I'll be starting my doctoral studies at next
    year. The coincidence sparked a few draft-swap jokes in the math department
    at Loomis.


<script type="module">
  import notebook from "https://api.observablehq.com/@alex-ozdemir/student-impressions-of-introductory-computer-science-top.js?key=a6ffa2175f2bf95d";

  const renders = {
    "interest": "#interest-chart",
    "difficulty": "#difficulty-chart",
    "understanding": "#understanding-chart",
  };

  import {Inspector, Runtime} from "https://unpkg.com/@observablehq/notebook-runtime@1.2.0?module";
  for (let i in renders) {
    renders[i] = document.querySelector(renders[i]);
  }

  Runtime.load(notebook, (variable) => {
    if (renders[variable.name]) {
      return new Inspector(renders[variable.name]);
    }
  });
</script>


[loomis]: https://http://loomischaffee.org/
[hudson]: http://math.bu.edu/people/hharper/
[reactive]: https://en.wikipedia.org/wiki/Functional_reactive_programming
