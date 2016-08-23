---

title: "Unsafe in Rust: The Abstraction Safety Contract and Public Escape"
date: 2016-08-07 09:46:25 -0700
categories: rust unsafe

---

{::options toc_levels="1,2" /}

After the work I did [analyzing syntactic patterns that `unsafe` is found
in][last], I spent the rest of my summer working on a tool to detect a
particular way that `unsafe` may be misused. In this post and the next I'd like
to explain what motivated this analysis, and provide a brief overview of how it
works.

# Overview
{:.no_toc}

This post begins with a review how `unsafe` facilitates the denotation of
abstraction boundaries, and proposes a set of rules for how these abstractions
should work: the _Abstraction Safety Contract_. We then discuss a particular
method by which the Abstraction Safety contract can be violated: Public Escape.
We conclude by beginning to design a static analysis that can reason about
Public Escape.

# Contents
{:.no_toc}

   * I will become the TOC
{:toc}

# Unsafe as an Abstraction Boundary

Early this summer Niko Matsakis [wrote][unsafe-abstractions-niko]
[about][unsafe-rules-niko] how Rust's `unsafe` feature should be seen as a tool
to declare and interact with abstraction boundaries that have more requirements
than Rust's type system allows them express. This perspective sheds light on the
meaning of both unsafe blocks and unsafe functions, so I'll summarize the key
takeaways here while introducing a new term which will help us talk about them:
_safety conditions_.

When a function is labelled as `unsafe`, some responsibility for memory safety
is being _deferred_. The `unsafe` flag that means that some inputs would cause
the function to execute in a memory-unsafe way. Its memory safety is contingent
on additional requirements being met and callers of the function should read
its documentation and make sure that the inputs satisfy those requirements.
We'll talk about these additional requirements frequently, so let's give them a
name: safety conditions.

A great example of this is the builtin slice method:

```rust
unsafe fn get_unchecked(&self, index: usize) -> &T;
```

The reason that this function is unsafe is given in its
[documentation][get-unchecked]:

> Returns a pointer to the element at the given index, **without doing bounds
> checking**"

That is, the safety condition for this function is that the index is in-bounds
for the slice. Any caller of this function must ensure this safety condition is
met.

This is where unsafe blocks come in. When a programmer uses an unsafe block,
they're effectively saying "I understand all the unsafe operations I'm
performing, I know their safety conditions, and I'll make sure those conditions
are met". In this way, unsafe blocks take responsibility for the memory safety
of the code that they contain.

An example of an unsafe block that guarantees that `get_unchecked`'s safety
conditions are being met is this one:

```rust
fn get_from_slice<T>(slice: &[T], index: usize) -> Option<&T> {
    if index < slice.len() {
        unsafe { Some(slice.get_unchecked(index)) }
    } else {
        None
    }
}
```

The `index < slice.len()` check guarantees that when the unsafe block runs, the
required safety condition ("index is in-bounds for slice") is met.

These observations about how unsafe blocks meet safety conditions and unsafe
functions defer safety conditions can help us build a precise specification for
how unsafe constructs should work. Specifically, I'd like to introduce a
contract for the correct use of unsafe, a sort of "Abstraction Safety Contract"
which specifies how unsafe blocks and functions should interact with safety
conditions:

## The Abstraction Safety Contract

* unsafe blocks should
   * guarantee they'll meet the safety conditions of contained code
* unsafe functions should
   * guarantee they'll meet the safety conditions of contained code
      * so long as their own (documented) safety conditions are met

In contexts where the meaning is clear, we'll abbreviate Abstraction Safety
Contract as ASC.

# Violations of the Abstraction Safety Contract: Escape Through Public Interfaces

Now that we have a contract for code that uses `unsafe`, we can start thinking
about whether particular examples respect or violate this contract. In
particular, we'll explore a particular class of violations of the ASC: safety
conditions escaping through public interfaces. The central motivation for
thinking about public escape is that if safety conditions escape a safe public
interface, then that interface cannot uphold the ASC.

To get a sense of what public escape is and see why it constitutes a violation
of the ASC, we'll look at a few examples.

### Example A

```rust
pub fn deref(ptr: *const i32) -> i32 {
    unsafe {
        *ptr
    }
}
```

In Example A, `deref` is a public, safe function that takes an arbitrary
pointer an dereferences it. It does so using an unsafe block, and we can reason
about whether this block is being used in a way that conforms to the Abstraction
Safety Contract.

{::comment}
However, we can't figure this out by just looking at the block. On its own, the
unsafe block isn't inherently good or bad --- it all depends on whether `ptr`
points to valid data that this function can read, and whether this condition is
met depends on the state of the program when the unsafe block runs (in
particular on the address stored in `ptr`). To understand this state, we must
reason about the code that executes before the block: the parts of the block's
enclosing abstraction that establish the program state prior to the block's
execution.

In example A, since `deref` is a public function, it is a public interface unto
itself. This means that to understand the program state before the unsafe block
runs, we need only look at `deref`. When we do this we see that `deref` takes
`ptr` as one of its inputs and because it knows nothing about its inputs, there
is no guarantee that the pointer dereference will succeed.

We can formalize this by saying that dereferencing `ptr` has a safety condition:
"`ptr` must be a valid pointer", and because `deref` gets `ptr` from its caller,
it cannot guarantee that this safety condition is met. Thus, the unsafe block
in `deref` violates the Abstraction Safety Contract.

In particular, this is a violation because a safety condition depends on the
input to a public, safe function.
{:/}

Dereferencing the raw pointer inside the unsafe block creates a safety
condition: "`ptr` must be dereferenceable". Since the value of `ptr` come from
the caller of `deref`, and `deref` is a public function, this critical value
escapes through a public interface. For this reason, we identify this as an
example of "Public Escape" - the value a safety condition depends on enters the
code through a public interface. Furthermore, since `deref` claims to be
'safe', its effectively places no restrictions on how it should be called. This
means that `deref` has failed to ensure the above safety condition is met - it
has no way to ensure that `ptr` is dereferenceable, since it doesn't even know
`ptr`'s origin. Because the safety condition escapes, `deref` is in violation of
the Abstraction Safety Contract[^techn].

[^techn]:
    Technically we might say that the unsafe block inside of `deref` is
    violating the ASC, but since it is `deref` that sets up the state of the
    program when the unsafe block runs, it's convenient to "hold `deref`
    accountable".

### Example B

```rust
/// Dereferences the input `ptr`.
/// NB: `ptr` _must_ be valid for this function to be memory safe!
unsafe fn deref_helper(ptr: *const i32) -> i32 {
    *ptr2
}
fn identity(ptr: *const i32) -> *const i32 {
    ptr
}
pub fn deref(ptr: *const i32) -> i32 {
    unsafe {
        let ptr2 = identity(ptr);
        deref_helper(ptr2)
    }
}
```

Example B is essentially just a re-factoring of Example A:

   * the unsafe dereference has been pulled out into a private, unsafe function
     (`deref_helper`)
   * an identity function (`identity`) has been inserted, copying `ptr` into
     `ptr2`

Doing this refactoring shouldn't make `deref` any safer, so we hope that this
should still qualify as a violation of the Abstraction Safety Contract. To see
that it does we note that 

   * the safety condition for `deref_helper` is that `ptr2` be dereferenceable
   * `identity` is used to set `ptr2` to `ptr`, so `ptr` must also be
     dereferenceable
   * `ptr`, as an argument to the public `deref`, escapes through a public
     interface.

So once again the value a safety condition depends on escapes through a public
interface, making it impossible to guarantee that the value in question will
satisfy the safety condition. For that reason, this function too is failing to
uphold the Abstraction Safety Contract.

### Example C

Now lets consider a negative example: a case where a safety condition doesn't
escape.

```rust
pub fn identity(x: i32) -> i32 {
    let p = &x as *const i32;
    unsafe {
        *p
    }
}
```

In this function a raw pointer is dereference, creating a safety condition: "`p`
must be dereferenceable". However, immediately above `p` is set to the address
of a local variable. For this reason, the safety condition does not escape the
function, and furthermore we'd agree that while `identity` is rather bizarre, it
does indeed uphold the Abstraction Safety Contract (even though we _could_ write
examples of programs that avoid public escape but still violate the ASC!).

### Example D

Now we consider a final example, inspired by the [intermezzOS][intermezzos]
teaching operating system. This code features memory-mapping: the fact that
the pattern by which a CPU can send information to specific hardware (e.g. the
VGA buffer) by writing to specified addresses (e.g. those starting at
`0xb8000`).

```rust
pub struct VGA {
    vga_mem_start: *mut u8,
    // ...
}

impl VGA {
    pub fn new() -> VGA {
        VGA {
            vga_mem_start: 0xb8000 as *mut u8,
            // ...
        }
    }
    pub fn write_first(&self, byte: u8) {
        unsafe {
            *(self.vga_mem_start) = byte;
        }
    }
}
```

At first glance, the `write_first` function seems problematic for the same
reason that `deref` was in Examples A and B: it is a public, safe function that
dereferences some part of its input (specifically `self.vga_mem_start`).
However, this case is also somewhat different --- `write_first` isn't just
dereferencing _any_ input, its dereferencing a private field of `VGA`. Since the
field is private, the fact that the safety condition "`self.vga_mem_start` holds
a write-able address" escapes _a public function_ doesn't immediately mean that
the safety condition has escaped _the public interface_.

To determine whether this condition has escaped the public interface, we're
forced to consider all public code that might write to `VGA`'s `vga_mem_start`
field.  If it turns out that the only such code is in `VGA::new()`, which puts a
magic address into the field, then we know that this safety condition doesn't
escape the public interface.

Of course, we haven't verify that the safety condition is actually met ---  we
don't know whether the address `0xb8000` can actually be written to, and we
definitely don't know that the address gives the correct location of the VGA
Buffer. All we can say is that no verification conditions escape the public
interface of `VGA`.

This asymmetry is important: **If verification conditions escape a public
interface then there must be a violation of the Abstraction Safety Contract, but
a lack of escape does not prove that the ASC is upheld.**

Hopefully by now you have some intuition for what public escape is, and have a
good understanding of why it's problematic for safety conditions to escape. In
the next section we're going to start designing an automatic tool that can
reason about public escape, which should help further solidify these ideas.

# Automating the Analysis

To reason automatically about public escape we're going to need to break out
a fairly powerful tool: dataflow analysis. If you're interested in learning
about dataflow analysis in detail, I'd recommend [this][dataflow-class-steve]
course's lectures, but I've also included a basic introduction that should be
enough to understand this post.

## Dataflow Analysis: An Introduction

I find examples helpful in building intuition for new ideas, so lets begin with
a question that dataflow analysis could be used to answer: "Which variables in a
function are equal to 5?". Consider this function:

```rust
fn my_func(flag: bool) {
    let x = 5;
    let y = 6;
    let z = x;
    let mut a = 0;
    if flag {
        a = x
    } else {
        a = y
    }
    // Rest of the function
}
```

The central technique of dataflow analysis is to step through a program,
determining which _facts_ are true at each point. At branches in control flow
the analysis is often unable to tell which branch occurs, so both are explored
and the resulting facts from each must be reconciled afterwards.

In the case of determining which variables hold the value 5, a simple analysis
would reason about facts such as "var {must,must not,may} be 5", and would use
rules such as:

   * `let var = 5` means "var must be 5"
   * `let var = /* other literal here */` means that "var must not be 5"
   * `let var1 = var2` means that if "var2 {must,must not,may} be 5" then "var1
     {must,must not,may} be 5"
   * At points where control flows merge (e.g. the end of an if/else construct),
     the analyses from the two flows must also be merged. If one branch says
     "var1 must be 5", and the other says "var1 must not be 5", then we must
     conclude that "var1 may be 5", since we can't tell which branch would
     actually run.

As the analysis move forward through `my_func`, it would determine the following:

```rust
fn my_func(flag: bool) {
    let x = 5;     // x must be 5
    let y = 6;     // x must be 5, y must not be 5
    let z = x;     // x must be 5, z must be 5, y must not be 5
    let mut a = 0; // x must be 5, z must be 5, y must not be 5, a must not be 5
    if flag {
        a = x      // x must be 5, z must be 5, y must not be 5, a must be 5
    } else {
        a = y      // x must be 5, z must be 5, y must not be 5, a must not be 5
    }              // Merge: (a must be 5) + (a must not be 5) = (a may be 5)

                   // x must be 5, z must be 5, y must not be 5, a may be 5

    // Rest of the function
}
```

Now, knowing whether an integer variable is 5 may not seem particularly useful,
but this analysis could be generalized to do constant evaluation. In particular,
imagine if we could determine whether boolean variables were true or false: this
could enable a compiler to determine which arm of an if/else gets executed an
omit the unused arm!

Dataflow analyses (and similar static analyses) are also used for many other
purposes: they can determine which variables are live, which pointers alias each
other, and even [how long borrows should last][nll-niko] (in Rust).
{::comment}
If you want
to learn more about dataflow analysis, I'd recommend [these
resources][dataflow-class-steve], and in particular [these
slides][dataflow-slides-steve].
{:/}

## Dataflow Analysis: Public Escape

As hinted at, we can also use dataflow analysis to detect the public escape of
safety conditions. To do this we'll treat safety conditions as our facts, and
reason about how they flow through a program. The trick is, this analysis is
actually a _backward analysis_: safety conditions are created by unsafe
operations and must be satisfied _before_ those operations (not after).

To get a sense of how such an analysis would work, I'll sketch a few of the
rules it would use, and then we'll look at how the analysis handles Examples A
and B from earlier.

   * `*my_raw_ptr` means that "my_raw_ptr must be deref-able" (given that
     my_raw_ptr is a raw pointer).
   * `let var1 = var2` means that any conditions that must hold for var1 must
     hold for var2 and var1 no longer needs to satisfy any conditions[^kill].
   * `let var1 = &var2` means that the condition "`var1` must be deref-able"
     does not flow any further (or escape)[^secure].

[^secure]:
    Note that this doesn't guarantee
    that the condition is met - just that it doesn't escape.

[^kill]:
    The reason that `let var1 = var2` frees var1 of all conditions is that is
    means the value of var1 will be overwritten by var2's value. Once we start
    dealing properly with pointers, arrays, and all the complexities of real
    programs we'll have to be much more careful about whether `lval1 = lval2`
    really overwrites our representation of `lval1` and whether our
    representation of `lval1` might alias some other lvalue representation
    involved with safety conditions. For now a simple understanding makes for a
    smooth exposition.

### Example A

When we use those rules to reason about Example A, our analysis determines the
following (read the comments from bottom to top, since this is a backward
analysis):

```rust
pub fn deref(ptr: *const i32) -> i32 {
             // ^ ptr must be deref-able
    unsafe {
             // ptr must be deref-able
        *ptr
             // (no conditions)
    }
}
```

The analysis determines that when the function begins the argument `ptr` must be
dereferenceable. Because there are safety conditions involving `deref`'s
arguments we say that these conditions escape the function. Furthermore, since
`deref` is a safe public function, this function escape also constitutes a
violation of the Abstraction Safety Contract.

### Example B

Recall example B (documentation omitted):

```rust
unsafe fn deref_helper(ptr: *const i32) -> i32 {
    *ptr2
}
fn identity(ptr: *const i32) -> *const i32 {
    ptr
}
pub fn deref(ptr: *const i32) -> i32 {
    unsafe {
        let ptr2 = identity(ptr);
        deref_helper(ptr2)
    }
}
```

In using a dataflow analysis to understand whether any safety conditions escape
the public function `deref`, we're forced to do an inter-procedural analysis: an
analysis that can reason about the significance of one function calling
another. Let's step through how analysis might do this, starting with the end of
`deref`.

The last operation that `deref` performs is calling `deref_helper` on `ptr2`.
Our analysis would thus have to analyze `deref_helper` at that point, and
determine any safety conditions created by calling it. That analysis would tell
conclude that the first argument to `deref_helper` must be deref-able, a condition
that would be brought back into `deref`'s analysis:

```rust
pub fn deref(ptr: *const i32) -> i32 {
    unsafe {
        let ptr2 = identity(ptr);
                                   // ptr2 must be deref-able
        deref_helper(ptr2)
                                   // (no conditions)
    }
}
```

The analysis would then encounter the call to `identity`, and assignment of
`identity`'s result to `ptr2`. We might be tempted to say that `identity` should
then be analyzed, just as `deref_helper` was, but we have to be careful - we
can't just analyze `identity` in isolation - we have to analyze it with some
extra information from the site where it is called (in this case, the knowledge
that `identitiy`'s return value must be dereferenceable). This extra information
is the _context_ in which `identity` in analyzed, and by handling contexts our
analysis models how data can enter a function through its arguments and leave
through its return value[^escape].

[^escape]:
    A function can also cause externally-visible dataflow by writing to lvalues
    pointed at by its arguments. Consider `fn swap<T>(a: &mut T, b: &mut T);`.
    Nevertheless, thinking about the return value is a good way to start.

When `identity` is analyzed in the context in which it's return value must be
dereferenceable, the result is that its first argument must also be
dereferenceable. Bringing this conclusion back into our analysis of `deref` and
finishing that analysis yields the following:

```rust
pub fn deref(ptr: *const i32) -> i32 {
                                   // ^ ptr must be deref-able
    unsafe {
                                   // ptr must be deref-able
        let ptr2 = identity(ptr);
                                   // ptr2 must be deref-able
        deref_helper(ptr2)
                                   // (no conditions)
    }
}
```

As in Example A, the analysis finds that `deref` requires that its first
argument must be dereferenceable --- a safety condition has escaped the
function. Since `deref` is both public and safe this escape is also an instance
of a safety condition escaping a public interface and indicates a violation of
the Abstraction Safety Contract.

### Example C

Now lets consider how our analysis handles an example with no public escape.
Remember to read the comment backwards, since the analysis is a backward one!

```rust
pub fn identity(x: i32) -> i32 {
                              // ^ (no conditions)
    let p = &x as *const i32;
                              // (p must be deref-able)
    unsafe {
                              // (p must be deref-able)
        *p
                              // (no conditions)
    }
}
```

While a condition "`p` must be dereferenceable" is still generated, that
condition is ultimately killed[^term] by the assignment `let p = &...`. For that
reason the analysis would conclude that this function features no public escape,
which is correct.

# Next Time: Example D and Beyond

We've seen how a public escape analysis might work on a few functions, but Rust
is a complex language and there is much more to do. In particular, we need:

   * to reason about the private fields of structs, as found in Example D.
   * to understand that different fields of structs hold different values.
   * to understand that `p` and `*p` refer to different locations/values
   * to reason about calls to functions that we don't have the source for
     (perhaps in external dependencies or boxed closures)
   * to reason about issues raised by aliasing (what if `x` and `*p` denote the
     same memory location)
   * to make sure that recursive functions and other examples of complex
     dependencies between functions are handled correctly.

In my next post I'll touch on some of these, and (hopefully) include the results
of running the analysis on various Rust codebases.

[unsafe-abstractions-niko]: http://smallcultfollowing.com/babysteps/blog/2016/05/23/unsafe-abstractions/
[unsafe-rules-niko]: http://smallcultfollowing.com/babysteps/blog/2016/05/27/the-tootsie-pop-model-for-unsafe-code/
[nll-niko]: http://smallcultfollowing.com/babysteps/blog/2016/05/09/non-lexical-lifetimes-adding-the-outlives-relation/
[intermezzos]: https://github.com/intermezzOS/kernel/blob/master/src/vga/src/lib.rs#L131
[get-unchecked]: https://doc.rust-lang.org/std/primitive.slice.html#method.get_unchecked
[dataflow-class-steve]: http://www.seas.harvard.edu/courses/cs252/2015fa/schedule.html
[dataflow-slides-steve]: http://www.seas.harvard.edu/courses/cs252/2015fa/lectures/Lec02-Dataflow.pdf
[last]: /rust/unsafe/unsafe-in-rust-syntactic-patterns/
