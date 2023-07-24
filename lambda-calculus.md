Skeleton
---

- Introduction
    - A brief introduction to lambda calculus history and what it actually is
    - Talk about currying and high-order functions
    - Highlight the functional nature of lambda calculus
- The need for extensions
    - Talk about the limitations of pure lambda calculus
    - Explain why extensions make it pratical and expressive
- Conditionals
    - Show how conditionals can be represented
    - Explore how conditionals can be used for branching
    - Introduce basic binary operations like `and`, `or` and `not`
- Church numerals
    - Explain what church numerals are and how they can be used to represent natural numbers
    - Show how basic arithimetic works and how `succ` and `plus` can be implemented
- Constants
    - Explain how constants are already built in into the pure lambda calculus
    - Show side to side comparissons to ocaml and how let binding is just sugar
- Recursion
    - Explain how to achieve recursion
    - Show both the u-comb and the y-comb and explain what they do and how they work

<!-- talk about by who, when and why it were created. -->
## Introduction to lambda calculus

Lambda calculus was developed in the 1930s by the mathematician Alonzo Church, none other than the teacher of Alan Turing, atheist and homosexual, father of the computer science. It was introduced as a way to explore the foundations of mathematics and computation.

I would like to quote Thomas Garitty on ["Functions describe the world"](https://www.youtube.com/watch?v=PAZTIAfaNr8). This couldn't be truer when we talk about lambda calculus. Keeping it short, lambda calculus is a model of computation where the only thing you can do is define functions with one argument that returns something else. Even though it's simple, lambda calculus is Turing complete, which basically means that if you give it enough time and resources, it can solve any computational problem.

How would we write a real program with only functions? What about branching and recursion? Well, you actually can do those things with just pure lambda calculus. 

# Currying

But first, how do we handle more than one value at a time?

```javascript
// Only one function with multiple arguments
(x, y) => add(x, y)
// Multiple functions with only one argument
(x => (y => add(x, y)))
```
We can reduce this by hand to make things clear:

```javascript
((x, y) => add(x, y))(1, 2)
add(1, 2)
3
// Multiple functions with only one argument
((x => (y => add(x, y))) 1) 2
(y => add(1, y)) 2
add(1, 2)
3
```

This is a method called currying, named after some random guy called Haskell Curry, which invented it. It allows us to break down a function that takes multiple arguments into a chain of 
single-argument functions.

# Binary Operations

Okay, but how do we make conditional operations, like ifs and elses? 
Let's define our booleans first:

```
True: λx.λy.x
False: λx.λy.y
```

Those are called Church booleans, abstractions that receive two arguments, and if True, the first one are returned; if False, the last.

With church booleans in place, we can now simulate a branching behavior using Church booleans and lambda abstractions. Here's how you can define a conditional expression in lambda calculus:

```
λcondition.λdo_this.λdo_that.(condition do_this do_that)
```

The condition as a parameter representing one of the church booleans, that will determine with of the branches will be returned. You will get it when we reduce it again.

```
true = λx.λy.x

(λcondition.λdo_this.λdo_that.(condition do_this do_that)) true true_case false_case
(λdo_this.λdo_that.(true do_this do_that)) true_case false_case
(λdo_that.(true true_case do_that)) false_case
true true_case false_case
(λx.λy.x) true_case false_case

λtrue_case.λfalse_case.true_case
``````

# Church Numerals

This is already looking nice, but a crucial thing is missing to make lambda calculus usable. Being able to deal with numbers.
The most popular way to represent natural numbers in lambda calculus are using the Church numberals enconding. In summary, to define the numeral N, we apply a given function N times to a given value. Here is a small list of definitions, from 0-3.

```
0: λf.λx.x
1: λf.λx.f x
2: λf.λx.f (f x)
3: λf.λx.f (f (f x))
```

Lets use the definition of `3` for example. If you think about it, in most cases, the end result of `λf.λx.f (f x)` will not be the number three, unless the function we are providing are the `succ` (It's defined bellow) and the 0 numeral as the value. This is simply a way of doing something three times, which means that not the end result, but the function itself is the church numberal three.

But how do we get the successor of a numeral?

```
succ: λn.λf.λx.f (n f x)
```

The `succ` function works by applying `f` to `x` one more time, returning the definition of the Church numeral that represents the successor of `n`. If you reached this point, you should be able to reduce it by hand and confirm this by yourself. It's fun, I promise.

# Arithimetic Operations

Now that we have numbers, we should be able to operate over them.

```
add: λm.λ.n.m succ n
```

A good way to explain what's happenning on the `add` definition, is that we are 1 to `n` `m` times. We can partially reduce it, so it will make sense:

```
add: λm.λ.n.m succ n
2: λf.λx.f (f x)

add 2 1
    = ((λm.λn.m succ n) 2) 1
    = (λn.2 succ n) 1
    = 2 succ 1
    = ((λf.λx.f (f x)) succ) 1
    = (λx.succ (succ x)) 1
    = succ (succ 1)
```

We applied `succ` at `1` two times.

