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
Lambda calculus was developed in the 1930s by the mathematician Alonzo Church, none other than the teacher of Alan Turing, atheist and homosexual, father of the computer science. It was introduced as a way to explore the foundations of mathematics and computation.

I would like to quote Thomas Garitty on ["Functions describe the world"](https://www.youtube.com/watch?v=PAZTIAfaNr8). This couldn't be truer when we talk about lambda calculus. Keeping it short, lambda calculus is a model of computation where the only thing you can do is define functions with one argument that returns something else. Even though it's simple, lambda calculus is Turing complete, which basically means that if you give it enough time and resources, it can solve any computational problem.

How would we write a real program with only functions? What about branching and recursion? Well, you actually can do those things with just pure lambda calculus. 

But first, how do we handle more than one value at a time?

```javascript
// Only one function with multiple arguments
(x, y) => x + y
// Multiple functions with only one argument
(x => (y => x + y))
```
We can reduce this by hand to make things clear:

```javascript
((x, y) => x + y)(1, 2)
(1, 2) => 1 + 2
3
// Multiple functions with only one argument
((x => (y => x + y))1)2
(1 => (y => 1 + y))2
(2 => 1 + 2)
1 + 2
3
```

This is a method called currying, named after some random guy called Haskell Curry, which invented it. It allows us to break down a function that takes multiple arguments into a chain of 
single-argument functions.

Okay, but how do we make conditional operations, like ifs and elses? 
Let's define our booleans first:

```
True: λx.λy.x
False: λx.λy.y
```

Those abstractions are called Church booleans. They are high-order functions, which are functions that receive a function and return another function. 
True and False are abstractions that receive two arguments, and if true, the first one are returned; if not, the last.

With church booleans in place, we can now simulate a branching behavior using Church booleans and lambda abstractions. Here's how you can define a conditional expression in lambda calculus:

```
λcondition.λdo_this.λdo_that.(condition do_this do_that)
```

The condition represents one of our church booleans that will determine with of the branches will be executed. You will get it when we reduce it again.

```
true = λx.λy.x

(λcondition.λdo_this.λdo_that.(condition do_this do_that)) true true_case false_case
(λdo_this.λdo_that.(true do_this do_that)) true_case false_case
(λdo_that.(true true_case do_that)) false_case
true true_case false_case
(λx.λy.x) true_case false_case
(λtrue_case.λy.true_case)
true_case
``````


