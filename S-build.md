---
title: Building Code
nodes: 
objectives:
  - "Review floating-point mathematics including IEEE-754."
  - "Examine `@r` atomic representation of floating-point values."
  - "Manipulate and convert floating-point values using the `@r` operations."
  - "Examine `@s` atomic representation of signed integer values."
  - "Use `+si` to manipulate `@s` signed integer values."
  - "Define entropy and its source."
  - "Utilize `eny` in a random number generator (`og`)."
  - "Distinguish insecure hashing (`mug`) from secure hashing (`shax` and friends)."
---

#   Building Code

_This lesson introduces how non-`@ud` mathematics are instrumented in Hoon.  It may be considered optional and skipped if you are speedrunning Hoon School._

As a functional language with a powerful build engine, Hoon exposes much of the build process to the developer who knows where to look.  This family of properties, with names like [introspection](https://en.wikipedia.org/wiki/Type_introspection) and [reflection](https://en.wikipedia.org/wiki/Reflective_programming), means that aspects of code building can be exposed and modified by the developer.  While we have small need to modify code [beyond wet gates](./Q-metals.md), we can learn a lot by observing how text becomes Hoon becomes Nock.

Formally, Hoon can be described as a purely functional systems language. It’s statically typed, purely functional, and strictly evaluated.  Hoon is designed to compile and run arbitrary code in a typesafe way, allow hot code reloading, and permit metaprogramming.  Urbit is also completely homoiconic: code and data are the same thing—a noun—and even the environment is a Nock binary tree.  You can manipulate code as if it were data, and vice versa.

dataflow computing?
https://twitter.com/rovnys/status/1349442422850334721


##  How Code is Built

The essential steps of the Hoon parser are:

1. Parser
2. AST Builder
3. AST Reduction
4. Nock Builder

AST
compile-time
etc.

mint, ream, vast, etc.

##  Affordances

`!=` zaptis

---
Now let's try to define a core with a sample:

```hoon
> =a =>  ~  =|  b=@  |%  ++  foo  b  --

> a
<1.zgd [b=@ %~]>
```

Dojo gives us the same result!  But is the pretty printer hiding something from us?  Recall that Hoon compiles down to Nock, Urbit's purely-functional assembly-level language.  The `!=` zaptis rune returns the Nock of any Hoon expression. Let's try it on these expressions:

> !=  =>  ~  |_  b=@  ++  foo  b  --

[7 [1 0] 8 [1 0] [1 0 6] 0 1]

> !=  =>  ~  =|  b=@  |%  ++  foo  b  --

[7 [1 0] 8 [1 0] [1 0 6] 0 1]

You don't need to know any Nock to see that these two expressions are identical. Thus we have witnessed that doors really are just cores with samples.

---

`;;` micmic

The [`^~` ketsig](https://urbit.org/docs/hoon/reference/rune/ket#-ketsig) rune

[++make](https://urbit.org/docs/hoon/reference/stdlib/5d#make)

cook
etc.

jamfiles

boot process
