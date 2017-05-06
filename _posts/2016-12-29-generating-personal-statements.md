---

title: "Generating Personal Statements"
date: 2016-12-29 16:27:25 -0800
categories: linguistics generation
published: true
excerpt: Generating natural language using a syntax-aware model.

---

{::options toc_levels="1,2" /}

# Overview
{:.no_toc}

Generating texts of natural language is a hard but fun problem. Some friends and
I set forth to apply a few text generation models to the problem of generating
essays for graduate school applications.


# Contents
{:.no_toc}

   * I will become the TOC
{:toc}

# Motivation

Next year I'm planning to pursue graduate work in Computer Science.
Consequently, I've spent much of the past few weeks applying to various graduate
programs. Since doing research is among a graduate student's greatest
responsibilities, the biggest part of these applications is the "Personal
Statement" or "Statement of Purpose" in which you're supposed to detail your
research experience and interests.

My friends and I found writing these to be somewhat of a pain, and so after
completing our applications we decided to explore ways of generating them
automatically. We didn't set out to make a particularly good
statement-generator, we just thought it would be fun.

# Framework

We'll define the problem like this: Given a set of sentences $$ S $$, construct
a system that will generate sentences _like_ those in $$ S $$. By _like_, we
mean sentences that are written in the same style and have a valid, novel meaning.

To obtain our set of sentences, we emailed Harvey Mudd's current senior class,
explaining our idea and requesting statements. We ended up with 11 statements,
which collectively contained 9,800 words. Most were statements of purpose
written for a graduate school application, but some were written for REU
applications, fellowship applications, or were diversity statements.

# Four Models

## Do Word Frequencies Model Language?

The simplest way that one could imagine generating text is randomly selecting
words based on their frequency in the sample text.

To do this, one would compute a function $$ f_S : W \to \mathbb{N} $$ which maps the set of words in
the sample (call this set $$ W $$) to the number of times they occur in $$ S $$. This gives us the
*pdf* (probability density function) of the words in $$ S $$. One could then
invert this pdf, $$ f $$ to produce a generator $$ g_S : \mathcal{R} \to W $$ which maps a source of
randomness $$ \mathcal{R} $$ to a word from $$ W $$.

Given this generator $$ g_S $$, One could construct a sentence of length $$ n $$
by using $$ g_S $$ to generate the first word, the second word, and so on until
$$ n $$ words had been produced.

Intuitively this model seems bad because it does not account for the *structure*
of language. Sentences are not just bags of words -- the meaning of a sentence
depends on both the words in the sentence and their ordering. Any given
sentences can have its words re-arranged in a way that is meaningless or has
different meaning than the original. The sentence "The cute cat slept" is meaningful
but the sentence "The cat cute slept" is not: you can't put adjectives after
their nouns in English! Since our model is blind to word order, it would be
equally likely to produce both these sentences, even though the first is
grammatically acceptable and the second isn't.

Given that this model clearly does not account for the structure of language, we
did not implement it, and moved on to better models.

## Do Markov Chains Model Language?

One way to incorporate language's structure into our model is to use the idea of
a *Markov Chain*. A Markov Chain is a sequence of stochastic events in which the
outcome of each event varies according to a distribution determined by a finite
number of prior events[^realmarkov]. Sentences can be modeled as Markov Chains
by regarding them as sequences of words where the probability of each word
occurring depends on the words that occurred before it.

[^realmarkov]:
    The precise definition of a Markov process is slightly different: that the
    probability of each event can be determined by some finite amount of state
    stored in the process. This isn't quite equivalent to the definition we
    give, but it's close.


While it's unclear exactly how good this model is, it is definitely an
improvement over our prior model. After all, language does have restrictions
about what words can follow other words, and a Markov Chain can at least
partially capture that behavior.  Consider our problem case from before: the
sentences "The cute cat slept" and "The cat cute slept". Using a Markov model
the first sentence would be generated with much higher probability because the
second sentence includes a relatively unlikely sequence: the adjective "cute"
after the adjective "cat".

One can build this model into a generator as follows. Pick some constant $$ k
$$: the number of past words the model will consider in generating each future
word. Then, for each sequence of $$ k $$ words appearing consecutively in the
sample text $$ S $$, compute the pdf for the words that follow this $$ k
$$-tuple of words. That is, compute $$ f_{k, S}: W^k \to (W \to \mathbb{N}) $$.
Then, invert this function to produce a generator $$ g_{k, S} : W^k \to
(\mathcal{R} \to W) $$ which takes in a source of randomness and the last $$ k
$$ words in the generated sentence, and probabilistically produces the next word
for the sentence.

Given that this model seemed promising and is fairly easy to
implement[^cs5-markov], we went ahead and implemented it. We found that setting
$$ k = 2 $$ worked best for our data-set, and produced okay results. So far the
best is

> In developing quantum mechanics, a new systems programming language that is
> typical of laboratory coursework.

but unfortunately the vast majority of its productions are ungrammatical and
totally non-sensible.

[^cs5-markov]:
    It's in fact so easy to implement that it's an
    [assignment](https://www.cs.hmc.edu/twiki/bin/view/CS5/MarkovMaterialGold)
    for the first computer science class at Mudd. We, of course, re-implemented
    it for fun.

In some sense this doesn't surprise us: linguists have long since agreed that
language is best modeled as a tree-structure rather than a linear structure.
That is, the sentence "The cute cat slept" shouldn't be seen as a sequence of
words but as a tree:

![(S (NP (D The) (NP (Adj cute) (N cat))) (VP (V slept))][cute-cat-tree]

where:

   * S: Sentance
   * NP: Noun Phrase
   * VP: Verb Phrase
   * D: Determiner
   * Adj: Adjective
   * N: Noun
   * V: Verb

While a Markov Chain model captures some of the rules of language, it also
necessarily misses some because it fundamentally describes language as a linear
structure: a sequence of words.

## Do Probabilistic Grammars Model Languages?

To address the fundamental tree structure of language, we need to think about
language generation in a new way. Instead of looking at language as a sequence
of words, with each word being probabilistically generated, we'll look at it as
a tree, with each branching being probabilistically generated. The sentence
starts with just a root syntactic node, which probabilistically expands into
child nodes, which in turn expand into their children, until the expansion
terminates with actual words.

This model is known as a ["Probabilistic Context-Free Grammar"][pcfg]. The model
allows us to generate new sentence trees given a set $$ T $$ of reference
sentence trees. We create our reference trees by using the [Stanford
parser][stanford-parser] to parse our bank of personal statements[^errors]. We
then look at every branching in the resulting trees: every node and its
children, and we compute the expansion probabilities for each node. That is we
compute the probabilities of any given node expanding to any given list of child
nodes, $$ f_{S}: L \to ([L] \to \mathbb{N}) $$ where $$ L $$ is the set of node
labels (every node is labelled with a grammatical construct like "NP" or a word
like "cat") and $$ [L] $$ is the set of lists of items from $$ L $$. That is,
for each parent node $$ l $$, $$ f_S(l) $$ is a pdf mapping each potential child
list of $$l$$ to how often the $$l$$ expands to it.

To help understand what $$f_S$$ is, consider the tree from before:

![(S (NP (D The) (NP (Adj cute) (N cat))) (VP (V slept))][cute-cat-tree]

If this tree was the only tree in our reference set $$S$$, then $$f_S(NP)$$
would map the list ["D", "NP"] to $$1$$ and the list ["Adj", "N"] to $$1$$,
indicating that a Noun Phrase, "NP", is equally likely to expand to ["D", "NP"]
and ["Adj", "N"]. In this way $$f_S$$ encodes the probability that a given
syntactic node expands in different ways.

We can invert $$f_S:  L \to ([L] \to \mathbb{N})$$ to make a branching generator
which randomly decides what children a given syntactic node should expand to.
This generator $$g_S$$ has type $$L \to (\mathcal{R} \to [L])$$. We can then use
$$g_S$$ to produce a sentence by starting with a syntactic root node, "S",
generating its children, the children of it children, and so on, until every
syntactic node has been expanded into an actual word. The process looks like
this:

![A tree is incrementally probabilistically constructed][tree-making]

### Incorporating Ancestry

The resulting sentences don't seem super great though, and a simple thought
experiment starts to show why. Consider this tree:

![A tree][the-the-tree]

It could be generated from the same single-tree reference set from before, with
only slightly less probability (half as likely). However, it is not grammatically
acceptable. The issue here is that while English allows a determiner ("The") to
branch off of an NP leaving another NP within, this doesn't happen twice. In
English, an NP which has a sibling determiner, cannot immediately expand to
another determiner. However, our model has no concept of this.

In essence, one way a English Noun Phrase can expand is like this:

    NP -> (D NP)

But in this expansion rule, the initial and final NP do not have exactly
identical grammatical properties: the former can produce a determiner, but the
latter cannot. This is just one example of how the grammatical classes (NP, VP,
S, etc.) do not uniquely determine a syntactic node's grammatical properties.
We need a way of enriching our node labels so that they more closely map to
grammatical roles.

[^errors]:
    Of course, the Stanford parser isn't perfect, but we hoped that its errors
    wouldn't lead to poor sentence generation.

By labelling each node with not only its grammatical class, but also the classes
of its ancestors, we more closely associate each node with its grammatical
function. A NP that's the child of another NP is unlikely to expand to a
determiner. An NP that's within a VP is unlikely to produce nouns that have
subject case, because that NP is in a predicate.

To condition the expansion probabilities on each node's last $$k-1$$ ancestors
we compute $$ f_{k,S}: L^k \to ([L] \to \mathbb{N})$$, and use these expansion
probabilities to inform our tree generation.

### Remaining Issues

The sentences produced by a generator using this ancestor-informed model are
actually pretty good. However, there's a silly class of mistakes that arise
surprisingly often. These mistakes include verb conjugation mismatches ("He
research") and incorrect selection of "a"/"an" ("a investigation"). These are
essentially failures to satisfy morphological agreement rules. Many languages
have rules that slightly change the words in a sentence depending on other words
in the sentence. These rules include verbs conjugating against nouns and/or
objects, adjectives agreeing with their nouns, and determiners agreeing with
their nouns.

Linguists use some very complicated models (including movement rules and
features[^reading]) to describe how agreement works, but fortunately for us,
English doesn't have a lot of agreement, and most of that agreement is between
parts of a sentence that are close to each other. This encourages us to try
adopting a simple strategy to handle agreement: an adaptation of Markov Chains.

[^reading]:
    For those interested, I'd recommend reading up on X-Bar syntax, the
    Minimalist Program, and movement rules. I have a textbook I can lend you if
    you want.

## Combining Probabilistic Grammars with Markov Chains

Since so much agreement in English (and other languages) is short-range, such as
choosing whether "a" or "an" should go before a noun, and which conjugation of a
verb should follow a particular subject, we attempt to handle this agreement by
conditioning our expansion probabilities on *the last word generated*. That is,
consider the following probabilistic generation in-progress:

!["A" has been generated, but the noun hasn't yet been generated][a-tree]

The way our system chooses how to expand the "N" node shouldn't just be
influenced by the ancestry of the node, but also by the last word produced: "A".
If we condition the expansion probabilities in this way, then producing an
un-agreeing noun like "apple" becomes unlikely, because producing "apple" after
"a" is rare in our reference texts.

The generalization of this is to condition node expansion on both the node's
$$k-1$$ ancestors, and the last $$j$$ words that have been generated. That is we
compute expansion probabilities:

$$ f_{k,j,S}: (L^k, W^j) \to ([L] \to \mathbb{N}) $$

This is a poor-man's approach to handling agreement, but it works surprisingly
well. Setting $$j=1$$ eliminates most obvious agreement errors.

This is the best model I've tried so far, and is the default on the
[site][site].

# Conclusions

We ultimately considered 4 models:

   1. Word frequencies
   2. Markov chains
   3. Probabilistic generative grammars
   4. PGGs enhanced by a Markov model

We found that language is not just a salad of words which occur with varying
frequencies. We verified that a Markov chain model does not tend to produce
grammatically reasonable sentences.

We moved on to probabilistic generative grammars and found that because
grammatical classes alone do not map well to grammatical functions, PGG sentence
generation doesn't produce grammatically reasonable sentences. However, we also
found that by conditioning our PGG on the grammatical class of each node,
together with its ancestors, more correct sentences were produced. Finally, we
reincorporated a Markov model in order to account for morphological agreement in
English.

# Credits & Further Investigation

While I'm excited about this project because it treats language generation as a
tree generation problem rather than a sentence generation problem, I'm less
excited about some of the compromises made along the way (approximating
grammatical function by ancestry chains of grammatical classes and using a
Markov approach to handle agreement). It'd be cool to revisit these issues and
develop ore satisfying resolutions.

I also want to acknowledge a few people who worked on this as well. Tim
Middlemas implemented the first, Markov chain approach with me, and the rest of
the work with generative grammars was all based on an idea from Michael Sheely.

You can find a site with a online version of the generators [here][site] and the
source for the project [here][source]. The data is not online, but if you want
to get a hold of it, let me know.

[markov]: https://en.wikipedia.org/wiki/Markov_chain
[pcfg]: https://en.wikipedia.org/wiki/Stochastic_context-free_grammar
[xbar]: https://en.wikipedia.org/wiki/X-bar_theory
[minimalist-program]: https://en.wikipedia.org/wiki/Minimalist_program
[move-a]: https://en.wikipedia.org/wiki/Move_%CE%B1

[stanford-parser]: http://nlp.stanford.edu/software/lex-parser.shtml

[site]: https://www.cs.hmc.edu/~aozdemir/ps-markov/
[source]: https://github.com/alex-ozdemir/ps-markov/

[cute-cat-tree]: /images/2016-12-29-tree-the-cute-cat.png
{: .align-center }

[tree-making]: /images/2016-12-29-tree-making.gif
{: .align-center }

[the-the-tree]: /images/2016-12-29-tree-the-the-cute-cat.png
{: .align-center }

[a-tree]: /images/2016-12-29-tree-a.png
{: .align-center }
