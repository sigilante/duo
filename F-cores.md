---
title: Cores
nodes: 130, 133
objectives:
  - "Employ a trap to produce a reentrant block of code."
  - "Produce a recursive gate."
  - "Distinguish head and tail recursion."
  - "Consider Hoon structures as cores."
  - "Identify the special role of the `$` buc arm in many cores."
  - "Order neighboring cores within the subject for addressibility."
---

#   Cores

The Hoon subject is a noun.  One way to look at this noun is to denote each fragment of is as either a computation or data.  By strictly separating these two kinds of things, we derive the data structure known within Hoon as a _core_.

Cores are the most important data structure in Hoon.  They allow you to solve many coding problems by identifying a pattern and supplying a proper data structure apt to the challenge.  You have already started using cores with `|=` bartis gate construction and use.

This lesson will introduce another core to solve a specific use case, then continue with a general discussion of cores.  Getting cores straight will be key to understanding why Hoon has the structure and internal logic it does.


##  Repeating Yourself Using a Trap

Computers were built and designed to carry out tasks which were too dainty and temperamental for humans to repeat consistently, or too prodigiously numerous for humans to ever complete.  At this point, you know how to build code that can make a decision between two branches, two different Hoon expressions.  Computers can decide between alternatives, but they also need to carry out a task until some condition is met.  (We can think of it as a recipe step, like “crack five eggs into a bowl”.  Until that process is complete, we as humans continue to carry out the equivalent action again and again until the process has been completed.)

In programming, we call this behavior a “loop”.  A loop describes the situation in which we set up some condition, and repeat a process over and over until something we do meets that condition.  _Most_ of the time, this means counting once for each item in a collection, like a list.

Hoon effects the concept of a loop using recursion, return to a particular point in an expression (presumably with some different values).  One way to do this is using the [`|-` barhep](https://urbit.org/docs/hoon/reference/rune/bar#-barhep) rune, which creates a structure called a _trap_.  (Think of the “trap” in the bottom of your sink.)  It means a point to which you can return again, perhaps with some key values (like a counter) changed.  Then you can repeat the calculation inside the trap again.  This continues until some single value, some noun, results, thereby handing a value back out of the expression.  (Remember that every Hoon expression results in a value.)

This program adds 1+2+3+4+5 and returns the sum:

```hoon
=/  counter  1
=/  sum  0
|-
?:  (gth counter 5)
  sum
%=  $
  counter  (add counter 1)
  sum      (add sum counter)
==
```

(The last two lines happen “simultaneously”.)

Let's unroll it:

0.  `counter = 1`
    `sum = 0`
1.  `(gth counter 5) = %.n`
    `counter ← (add counter 1) = 2`
    `sum ← (add sum counter) = 0 + 1 = 1`
2.  `(gth counter 5) = %.n`
    `counter ← (add counter 1) = 3`
    `sum ← (add sum counter) = 1 + 2 = 3`
3.  `(gth counter 5) = %.n`
    `counter ← (add counter 1) = 4`
    `sum ← (add sum counter) = 3 + 3 = 6`
4.  `(gth counter 5) = %.n`
    `counter ← (add counter 1) = 5`
    `sum ← (add sum counter) = 6 + 4 = 10`
5.  `(gth counter 5) = %.n`
    `counter ← (add counter 1) = 6`
    `sum ← (add sum counter) = 10 + 5 = 15`
6.  `(gth counter 5) = %.y`

(And thus `sum` has the final value of `15`.)

It is frequently helpful, when constructing these, to be able to output the values at each step of the process.  Use the [`~&` sigpam](https://urbit.org/docs/hoon/reference/rune/sig#-sigpam) rune to create output without changing any values:

```hoon
=/  counter  1
=/  sum  0
|-
~&  "counter:"
~&  counter
~&  "sum:"
~&  sum
?:  (gth counter 5)
  sum
%=  $
  counter  (add counter 1),
  sum      (add sum counter))
==
```

You can do even better using _interpolation_:

```hoon
=/  counter  1
=/  sum  0
|-
~&  "counter: {<counter>}"
~&  "sum: {<sum>}"
?:  (gth counter 5)
  sum
%=  $
  counter  (add counter 1),
  sum      (add sum counter))
==
```

Another example:  let's calculate a factorial.  (This is not the most efficient way to do this!)  We will introduce a couple of new bits of syntax and a new gate (`++dec`).  Make this into a generator `fact.hoon`:

```hoon
|=  n=@ud
|-
~&  n
?:  =(n 1)
  n
%+  mul
n
%=  $
  n  (dec n)
==
```

- We are using the `=` irregular syntax for `.=` dottis, test for equality of two values.
- We are using the `+` irregular syntax for `.+` dotlus, increment a value (add one to a value).
- Why do we return the result (`product` in Hoon parlance) at 1 instead of 0?

One more thing:  as we write more complicated programs, it is helpful to learn to read the runes:

```
=/
  n
  15
  |-
    ~&
      n
      ?:
        =(n 1)      :: .=  n  1
        n
      %+
        mul
        n
        %=
          $
          n
          (dec n)   :: %-  dec  n
        ==
```

As we move on from this lesson, we are going to revert to the irregular form.  If you would like to see exactly how one is structured, you can use the [`!,` zapcom](https://urbit.org/docs/hoon/reference/rune/zap#-zapcom) rune.  `!,` zapcom produces an annotated _abstract syntax tree_ (AST) which labels every value and expands any irregular syntax into the regular runic form.

```hoon
> !,  *hoon  (add 5 6)
[%cncl p=[%wing p=~[%add]] q=~[[%sand p=%ud q=5] [%sand p=%ud q=6]]]
```

```hoon
> !,  *hoon  |=  n=@ud  
 |-  
 ~&  n  
 ?:  =(n 1)  
   n  
 %+  mul  
 n  
 %=  $  
   n  (dec n)  
 ==  
[ %brts  
 p=[%bcts p=term=%n q=[%base p=[%atom p=~.ud]]]  
   q  
 [ %brhp  
     p  
   [ %sgpm  
     p=0  
     q=[%wing p=~[%n]]  
       r  
     [ %wtcl  
       p=[%dtts p=[%wing p=~[%n]] q=[%sand p=%ud q=1]]  
       q=[%wing p=~[%n]]  
         r  
       [ %cnls  
         p=[%wing p=~[%mul]]  
         q=[%wing p=~[%n]]  
         r=[%cnts p=~[%$] q=~[[p=~[%n] q=[%cncl p=[%wing p=~[%dec]] q=~[[%wing p=~[%n]]]]]]]  
       ]  
     ]  
   ]  
 ]  
]
```

(_There's a lot going on in there._  Focus on the four-letter runic identifiers:  `%sgpm` for `~&` sigpam, for instance.)

> ##  Calculate a sequence of numbers
>
> Produce a gate (generator) which accepts a `@ud` value and
> calculates the series where each term is described by
> 
> $$
> n_{i} = i^{2}
> \textrm{,}
> $$
>
> that is, the first numbers are 0, 1, 4, 9, 16, 25, etc.
>
> You do not need to store these values in a list; simply output
> them at each step using `~&` sigpam and return the final value.
{: .challenge}

> ##  Output each letter in a `tape`
>
> Produce a gate (generator) which accepts a `tape` value and
> prints out each letter in order on a separate line.
>
> For example, given the `tape` `"hello"`, the generator should 
> print out
> 
> 'h'
> 'e'
> 'l'
> 'l'
> 'o'
>
> You do not need to store these values in a list; simply output
> them at each step using `~&` sigpam and return the final value.
> 
> You can retrieve the _n_-th element in a `tape` using the 
> `++snag` gate:
> 
> ```
> > =/  n  0  (snag n "hello")
> 'h'
> ```
> 
> (Note that `++snag` counts starting at zero, not one.)
{: .challenge}


##  Cores

So far we have introduced and worked with a few key structures:

1. Nouns
2. Molds (types)
3. Gates
4. Traps

Some of them are _data_, like raw values:  `0x1234.5678.abcd` and `[5 6 7]`.  Others are _code_, programs that do something.  What unifies all of these under the hood?

A core is a cell pairing operations to data.  (Think back to Lesson -1:  we have state, data, and operations.  Cores represent two of these.)  Formally, we'll say a core is a cell `[battery payload]`, where `battery` describes the things that can be done (the operations) and `payload` describes the data on which those operations rely.

(I feel like “battery” evokes the voltaic pile more than a bank of guns, but the latter actually does something directly.  Actually, come to think of it this is entirely an artillery metaphor.)

**Cores are the most important structural concept for you to grasp in Hoon.**  Everything nontrivial is a core.  Some of the runes you have used already produce cores, like the gate.  That is, a gate marries a `battery` (the operating code) to the `payload` (the input values AND the “subject” or operating context).

Urbit adopts an innovative programming paradigm called “subject-oriented programming.”  By and large, Hoon (and Nock) is a functional programming language in that running a piece of code twice will always yield the same result.

However, Hoon also very carefully bounds the known context of any part of the program as the _subject_.  Basically, the subject is the noun against which any arbitrary Hoon code is evaluated.

For instance, when we first composed generators, we made what are called “naked generators”:  that is, they do not have access to any information outside of the base subject (Arvo, Hoon, and `%zuse`) and their sample (arguments).  Other generators (such as `%say` generators, described below) can have more contextual information, including random number generators and optional arguments, passed to them to form part of their subject.

Cores have two kinds of values attached:  arms and legs, both called limbs.  Arms describe known labeled addresses (with `++` luslus or `+$` lusbuc) which carry out computations.  Legs are limbs which store data.

![](https://davis68.github.io/martian-computing/img/08-nubret.png)

### Arms

An [_arm_](https://urbit.org/docs/glossary/arm) is a Hoon expression to be evaluated against the core subject (i.e. its parent core is its subject).

Within a core, we label arms as Hoon expressions (frequently `|=` bartis gates) using the [`++` luslus](https://urbit.org/docs/hoon/reference/rune/lus#-luslus) digraph.  (`++` isn't formally a rune because it doesn't actually change the structure of a Hoon expression.)

```hoon
|%
++  add-one
  |=  a=@ud
  ^-  @ud
  (add a 1)
++  sub-one
  |=  a=@ud
  ^-  @ud
  (sub a 1)
--
```

(The `--` hephep limiter is used because `|%` barcen can have any number of arms attached.)

We can also define custom types using [`+$` lusbuc](https://urbit.org/docs/hoon/reference/rune/lus#-lusbuc) digraphs.  We won't do much with these yet but they will come in handy for custom types later on.

This core defines a set of types intended to work with playing cards:

```hoon
|%
+$  suit  ?(%hearts %spades %clubs %diamonds)
+$  rank  ?(1 2 3 4 5 6 7 8 9 10 11 12 13)
+$  card  [sut=suit val=rank]
+$  deck  (list card)
---
```

When we write generators, we can include helpful tools as arms either before the main code (with `=>` tisgar) or after the main code (with `=<` tisgal):

```hoon
|=  n=@ud
=<
(add-one n)
|%
++  add-one
  |=  a=@ud
  ^-  @ud
  (add a 1)
--
```

A library (a file in `/lib`) is typically structured as a `|%` barcen core.

### Legs

A [_leg_](https://urbit.org/docs/hoon/hoon-school/the-subject-and-its-legs) is a data value.  They tend to be rather trivial but useful ways to pin constants.  `=/` tisfas values are legs.

TODO

https://urbit.org/docs/hoon/hoon-school/gates#what-is-a-gate
https://urbit.org/docs/hoon/hoon-school/gates#anatomy-of-a-gate

#### Exercise:  Three Ways to Calculate a Factorial

trap, then gate, then tail-call gate

```hoon
|=  n=@ud
?:  =(n 1)
  1
(mul n $(n (dec n)))
```

##  Tutorial:  List of Numbers

_This tutorial is intended to familiarize you with the basics of Hoon syntax.  It's okay if you don't understand everything immediately; some concepts may be beyond your grasp for now.  What's important is that you become accustomed to the elements of the code and their look once combined into a valid program._

Below is a simple Hoon program that takes a single number `n` as input with the `=/` tisfas and produces a list of numbers from `1` up to (but not including) `n`. So, if the user gives the number `5`, the program will produce: `~[1 2 3 4]`.

```hoon
|=  end=@                                               ::  1
=/  count=@  1                                          ::  2
|-                                                      ::  3
^-  (list @)                                            ::  4
?:  =(end count)                                        ::  5
  ~                                                     ::  6
:-  count                                               ::  7
$(count (add 1 count))                                  ::  8
```

As we mentioned in the previous lesson, the easiest way to use such a program is to run it as a _generator_. Mount your `%base` desk with `|mount %base` (if you didn't do it before), saving a file in the `base/gen` directory of your ship and commiting these changes `|commit %base` allows you to run it from your ship's Dojo (command line) as a generator. Save the above code there as `list.hoon`.

**Note**: If you're using VS Code on Windows, you might need to manually change the line endings from CRLF to LF in the status bar at the bottom. Urbit requires Unix-style line endings for Hoon files.

Now you can run it in the Dojo with the command below (remember, without the `>`):

`> +list 5`

Try it! You can choose any natural number to put after `+list`, but it can't be `0` or blank. The `+` lets the Dojo know that it's looking for a generator with the name that follows. You don't write out the `.hoon` part of the file name when running it in the Dojo.

But what do all these squiggly symbols in the program _do_? It probably isn't immediately clear. Let's go through each component of this code.

## The Code

You probably noticed that there's a column of colons and numbers in our example. The `::` digraph tells the compiler to ignore the rest of the text on the line. Such text is referred to as a "comment" because, instead of performing a computation, it exists to explain things to human readers of the source code.

In our example program, we use comments with line numbers for convenient reference, so that we can dig into the code line by line. Important Hoon concepts will be bolded when first mentioned.

### Line 1

```hoon
|=  end=@
```

There's a few things going on in our first line. The first part of it, `|=`, is a kind of **rune**. Runes are the building blocks of all Hoon code, represented as a pair of non-alphanumeric ASCII characters. Runes form expressions; runes are used how keywords are used in other languages. In other words, all computations in Hoon ultimately require runes. Runes and other Hoon expressions are all separated from one another by either two spaces or a line break.

All runes take a fixed number of "children". Children can themselves be runes with children, and Hoon programs work by chaining through these until a value -- not another rune -- is arrived at. For this reason, we very rarely need to close expressions. Keep this scheme in mind when examining Hoon code.

The specific purpose of the `|=` rune is to create a **gate**. A [gate](https://urbit.org/docs/glossary/gate/) is what would be called a function in other languages: it takes an input, performs a specified computation, and then produces an output.

Because we're only on line 1, all we're doing with the gate is creating it, and then specifying what kind of input the gate takes with that rune's first child: `end=@`. The `end` part of our code is simply a name that we give to the user's input so that we can use the number later. `=@` means that we restrict the kind of input that our gate accepts to the **atom** type, or `@` for short. An [atom](https://urbit.org/docs/glossary/atom/) is a natural number.

Our program is simple, so the _entire program_ is the gate that's being created here. The rest of our lines of code are part of the second child of our gate, and they determine how our gate produces an output.

### Line 2

```hoon
=/  count=@  1
```

This line begins with the `=/` rune, which stores a value with a name and specifies its type. It takes three children.

`count=@` (the first child) stores `1` (the second child) as `count` and specifies that it has the `@` type.

We're using `count` to keep track of what numbers we're including in the list we're building. We'll use it later in the program.

### Line 3

```hoon
|-
```

The `|-` rune functions as a "restart" point for recursion that will be defined later. It takes one child.

### Line 4

```hoon
^-  (list @)
```

The `^-` rune constrains output to a certain type. It takes two children.

In this case, the rune specifies that our gate's output must be `(list @)` -- that is, a list of atoms.

### Lines 5 and 6

```hoon
?:  =(end count)
  ~
```

`?:` is a rune that evaluates whether its first child is `true` or `false`. If that child is `true`, the program branches to the second child. If it's `false`, it branches to the third child. `?:` takes three children.

`=(end count)` checks if the user's input equals to the `count` value that we're incrementing to build the list. If these values are equal, we want to end the program, because our list has been built out to where it needs to be. Note that this expression is, in fact, a rune expression, just written a different way than you've seen so far. `=(end count)` is an _irregular form_ of `.= end count`, different in looks but identical in operation. `.=` is a rune that checks for the equality of its two children, and produces a `true` or `false` based on what it finds.

`~` simply represents the `null` value. The program branches here if on line 5 it finds that `end` equals `count`. Lists in Hoon always end with `~`, so we need this to be the last thing we put in our list.

## Line 7

`:-` is a rune that creates a **cell**, an ordered pair of two values, such as `[1 2]`. It takes two children.

In our case, `:- count` creates a cell out of whatever value is stored in `count`, and then with the product of line 8.

## Line 8

```hoon
$(count (add 1 count))
```

The above code is, once again, a compact way of writing a rune expression. All you need to know is that this line of code restarts the program at `|-`, except with the value stored in `count` incremented by `1`. The construction of `(count (add 1 count))` tells the computer, "replace the value of `count` with `count+1`".

You'll notice that we use an unfamiliar word here: `add`. Unlike `count` and `end`, `add` is not defined anywhere in our program. That's because it's a gate that's predefined in the Hoon **standard library**. The standard library is filled with pre-defined gates that are generally useful, and these gates can be used just like something that you defined in your own program. You can see this gate, and other mathematical operators, in [section 1a](https://urbit.org/docs/hoon/reference/stdlib/1a) of the standard library documentation.

## Explanation

If you aren't clear on how the program is building the list, that's okay.

Our program works by having each iteration of the list creating a cell. In each of these cells, the head -- the cell's first position -- is filled with the current-iteration value of `count`. The tail of the cell, its second position, is filled with _the product of a new iteration of our code_ that starts at `|-`. This iteration will itself create another cell, the head of which will be filled by the incremented value of `count`, and the tail of which will start another iteration. This process continues until `?:` branches to `~` (`null`). When that happens, the list is terminated and the program doesn't have anything else to do, so it ends. So, a built-out list of nested cells can be visualized like this:

```unknown
   [1 [2 [3 [4 ~]]]]

          .
         / \
        1   .
           / \
          2   .
             / \
            3   .
               / \
              4   ~
```

If you still don't intuit how this is working, don't worry. We'll take a deeper look into recursion later with our [Recursion Walkthrough](https://urbit.org/docs/hoon/hoon-school/recursion).
