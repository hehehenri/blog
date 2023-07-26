+++
title = "Functions Describe the World"
date = "2023-07-24T00:00:00-00:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["lambdacalculus", "functionalprogramming"]
keywords = []
description = ""
showFullContent = false
readingTime = true
hideComments = false
color = "" #color from the theme settings
+++

## A Quick Historical Context

The lambda calculus was developed in the 1930s by the mathematician [Alonzo Church](https://wikipedia.org/wiki/Alonzo_Church), none other than the teacher of [Alan Turing](https://wikipedia.org/wiki/Alan_Turing), atheist, homosexual, father of the computer science. It was introduced as a way to explore the foundations of mathematics and computation.

## Introduction to the Lambda Calculus

The title of the post is a quote from the introduction to Thomas Garrity's ["Mathematical Maturity"](https://www.youtube.com/watch?v=PAZTIAfaNr8) lecture (I recommend you watch it before continuing. It's 40 seconds long and very funny). This quote perfectly encapsulates the essence of the lambda calculus. 

Keeping it short, the lambda calculus is a model of computation where the only thing you can do is define functions with one argument that returns another function (You can think of a function as a way to map an input to an output).

Even though it's simple, the lambda calculus is [Turing complete](https://en.wikipedia.org/wiki/Turing_completeness), which means that if you give it enough time and resources, it can solve any solvable computational problem.

But how would we write a real program with only functions? What about branching and recursion? Well, you actually can do those things with just pure lambda calculus. 

## Currying

But first, how we can use more than one value inside a function?

```javascript
// Only one function with multiple arguments
(x, y) => add(x, y)
add(
// Multiple functions with only one argument
(x => (y => add(x, y)))
add(x)(y)
```

We can apply [β reduction](https://wiki.haskell.org/Beta_reduction) to make things clear:

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

This is a method called currying, named after some random guy called [Haskell Curry](https://wikipedia.org/wiki/Haskell_Curry), who invented it. It allows us to break down a function that takes multiple arguments into a chain of single-argument functions.

## Binary Operations

Okay, but how do we make conditional operations, like _ifs_ and _elses_ ? 

Let's define our booleans first:

```haskell
True = λx.λy.x
False = λx.λy.y
```

Those are called [Church booleans](https://en.wikipedia.org/wiki/Church_encoding), abstractions that receive two arguments and, If **_True_**, the first argument is returned. If **_False_**, the second argument is returned.

With Church booleans in place, we can now simulate a branching behavior using Church booleans and abstractions. 

Here's how you can define a conditional expression in lambda calculus:

```haskell
if = λcondition.λthen.λelse.(condition then else)
```

The condition is represented as a parameter, one of the church booleans that will determine with of the branches will be returned.

You will get it when we reduce it again. Don't worry about the arithmetic operations, I'll talk about them later.

```haskell
true = λx.λy.x

if true (sum 2 2) (mult 3 3)
    -> (((λcondition.λthen.λelse.(condition then else)) true) (sum 2 2)) (mult 3 3)
    -> ((λthen.λelse.(true then else)) (sum 2 2)) (mult 3 3)
(λelse.(true (sum 2 2) else)) (mult 3 3)
    -> true (sum 2 2) (mult 3 3)
    -> ((λx.λy.x) (sum 2 2)) (mult 3 3)
    -> (λy.(sum 2 2)) (mult 3 3)
    -> (sum 2 2)
    -> 4
```

## Church Numerals
 
The most popular way to represent natural numbers in lambda calculus is using the **Church numerals** encoding. In summary, the definition of the numeral **_N,_** is the application of a given function **_F_** **_N_** times to a given value.

Here is a small list of definitions, from 0-3.

```haskell
0 = λf.λx.x
1 = λf.λx.f x
2 = λf.λx.f (f x)
3 = λf.λx.f (f (f x))
```

Let's use the definition of _3_ for example. If you think about it, in most cases, the result of `λf.λx.f (f (f x))` will not be the number three, unless the function we are providing is the **_succ_** (as defined below) and the _0_ numeral as the value. 

This is simply a way of doing something three times, which means that not the result, but the function itself is the church numeral three.

But how do we get the successor of a numeral?

```haskell
succ = λn.λf.λx.f (n f x)
```

The **_succ_** function works by applying **_f_** to **_x_** one more time, returning the definition of the Church numeral that represents the successor of _n._ 

If you reached this point, you should be able to apply reduction to **_succ 3_** and confirm this by yourself. It's fun, I promise!

## Arithmetic Operations

Now that we have numbers, we should be able to operate over them.

```haskell
add = λm.λn.m succ n
```

At this point, we are already composing functions. A good way to explain what's happening on the `add` definition is that we are applying **_succ_** to _n_ _m_ times. 

Let's partially reduce it to make things clear:

```haskell
add = λm.λn.m succ n
2 = λf.λx.f (f x)

add 2 1
    -> ((λm.λn.m succ n) 2) 1
    -> (λn.2 succ n) 1
    -> 2 succ 1
    -> ((λf.λx.f (f x)) succ) 1
    -> (λx.succ (succ x)) 1
    -> succ (succ 1)
    -> 3
```

We applied **_succ_** at _1_ two times, which evaluates to _3_.

```haskell
mult = λm.λn.m (add n) 0

mult 2 3
    -> ((λm.λn.m (add n) 0) 2) 3
    -> (λn.2 (add n) 0) 3
    -> 2 (add 3) 0
    -> ((λf.λx.f (f x)) (add 3)) 0
    -> (λx.(add 3) ((add 3) x)) 0
    -> (add 3) (add 3) 0
    -> (add 3) 3
    -> 6
```

The multiplication function is as simple as applying addition to **_m_ _n_** times. Similar to what we did on the **_add_** definition.

Opposite to the successor function, we also have the predecessor function. We'll also need it to be able to subtract numbers. 

Predecessor is defined by the following formula:

```haskell
pred = λn.λf.λx.n (λg.λh.h (g f)) (λu.x) (λu.u)

sub = λm.λn.n pred m
```

I won't write the reduction for this case to keep things short, but again, we are just applying **_pred_** to **_m_ _n_** times. You can take a look at **_add_** reduction if you are confused.

## Recursion

Recursion is a powerful concept in programming languages. Even lambda calculus being extremely simple, we can still use some tricks to achieve recursion.

But what is recursion? Take a look at this factorial example in javascript:

```javascript
const fact = (n) => {
    if (n == 0) {
        return 1;
    }

    return n * fact(n-1)
}
```

What makes **_fact_** interesting to us is the fact that it calls itself. That's recursion! Unfortunately, there isn't a direct way of achieving recursion directly.

We'll need to use a little trick called the Y combinator (I'm not talking about Silicon Valley's [Y Combinator](https://www.ycombinator.com/), but the fixed-point one).

The Y combinator takes a function as an argument and returns a fixed-point of that function. In other words, it enables self-reference within a function. Don't worry, I'll explain what a fixed-point is and why it works later.

```haskell
Y = λf.(λx.f (x x)) (λx.f (x x))
```

Let's write the factorial function in the lambda calculus syntax, so we can see the Y in action:

```
fact = λf.λn.(
   if eq(n 0) then 
        1 
    else 
        (mult n (f (sub n 1))))
```

The definition of _fact_ is perfect, but how can we call it? _f_ should be the _fact_ function itself. Can you think of a way to reference it?

First, let's see a fixed-point in action:

```haskell
Y g = (λf.(λx.f (x x)) (λx.f (x x))) g
    -> (λx.g (x x)) (λx.g (x x))
    -> g ((λx.g (x x)) (λx.g (x x)))
    -> g (Y g)
    -> ...
```

That's why **_Y_** is called a fixed-point. Regardless of the value of the function **_g_**, **_Y_** is still the same.

Well, given that, how does this fix the issue with the **_f_** parameter on the **_fact_** function? Let's plug both things together:

```haskell
(Y fact) 3
    = (λf.G = λf.(λx.f (x x)) (λx.f (x x)) fact) 3
    = ((λx.fact (x x)) (λx.fact (x x))) 3
    = fact ((λx.fact (x x)) (λx.fact (x x))) 3
    = fact (Y fact) 3
```

Wait, that was it? Yes! That's exactly what we needed as an argument to the **_f_** parameter on **_fact_**. A reference to itself. We can keep reducing to see if it will work (Don't run away, please! Read it slowly and I promise it will make sense):

```haskell
fact (Y fact) 3 
    -> (λf.λn.(if (eq n 0) then 1 else (mult n (f (sub n 1))))) (Y fact) 3
    -> (if (eq 3 0) then 1 else (mult 3 ((Y fact) (sub 3 1))))
    -> (mult 3 ((Y fact) (sub 3 1)))
    -> mult 3 (Y fact) 2
    -> mult 3 (fact (Y fact) 2)
    -> mult 3 (λf.λn.(if (eq n 0) then 1 else (mult n (f (sub n 1))))) (Y fact) 2
    -> mult 3 (if (eq 2 0) then 1 else (mult 2 ((Y fact) (sub 2 1))))
    -> mult 3 (mult 2 ((Y fact) (sub 2 1))))
    -> mult 3 (mult 2 ((Y fact) 1))
    -> mult 3 (mult 2 (fact (Y fact) 1))
    -> mult 3 (mult 2 ((λf.λn.(if (eq n 0) then 1 else (mult n (f (sub n 1))))) (Y fact) 1))
    -> mult 3 (mult 2 ((if (eq 1 0) then 1 else (mult 1 ((Y fact) (sub 1 1))))))
    -> mult 3 (mult 2 (mult 1 ((Y fact) 0)))
    -> mult 3 (mult 2 (mult 1 (fact (Y fact) 0)))
    -> mult 3 (mult 2 (mult 1 ((λf.λn.(if eq(n 0) then 1 else (mult n (f (sub n 1))))) (Y fact) 0)))
    -> mult 3 (mult 2 (mult 1 ((if (eq 0 0) then 1 else (mult n ((Y fact) (sub n 1)))))))
    -> mult 3 (mult 2 (mult 1 1))))
    -> mult 3 (mult 2 1)
    -> mult 3 2
    -> 6
```

And that's how you achieve recursion in pure lambda calculus.

## Final Thoughts

The lambda calculus, a foundational concept in mathematics and computer science, provides a way of expressing computation using only functions. It shows the elegance and versatility of functional programming, proving its capability to solve complex problems with simplicity. 
It continues to influence modern programming paradigms, making it a timeless and fundamental concept in the realm of computation.

If you made it this far and are still interested, you can see on [this repository](https://github.com/hnrbs/lambda-calculus) an actual implementation of the lambda calculus.

---

References:

- https://wikipedia.org/wiki/Lambda_calculus
- https://wikipedia.org/wiki/Church_encoding
- https://wikipedia.org/wiki/Fixed-point_combinator
