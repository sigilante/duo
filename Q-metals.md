---
title: Gates
nodes: 288, 299
objectives:
  - "Distinguish dry and wet cores."
  - "Describe use cases for wet gates (using genericity)."
  - "Enumerate and distinguish use cases for dry cores (using variance):"
  - "- Covariant (`%zinc`)"
  - "- Contravariant (`%iron`)"
  - "- Bivariant (`%lead`)"
  - "- Invariant (`%gold`)"
---

#   Adaptive Cores

Cores can expose and operate with many different assumptions about their inputs and structure.  `[battery payload]` describes the top-level structure of a core, but within that we already know other requirements can be enforced, like `[battery [sample context]]` for a gate, or no `sample` for a trap.  Cores can also expose and operate on their input values with different relationships.  This lesson is concerned with examining _polymorphism_, which allows flexibility in type, and _variance_, which allows cores to use different sets of rules as they evaluate.

If cores never changed, we wouldn't need polymorphism.  Of course, nouns are immutable and never change, but we use them as templates to construct new nouns around.

Suppose we take a core, a cell `[battery payload]`, and replace `payload` with a different noun.  Then, we invoke an arm from the battery.

Is this legal?  Does it make sense?  Every function call in Hoon does this, so we'd better make it work well.

The full core stores _both_ payload types:  the type that describes the `payload` currently in the core, and the type that the core was compiled with.

In the Bertrand Meyer tradition of type theory, there are two forms of polymorphism:  _variance_ and _genericity_.  In Hoon this choice is per core:  a core can be either `%wet` or `%dry`.  Dry polymorphism relies on variance; wet polymorphism relies on genericity.

This lesson discusses both genericity and variance for core management.  These two sections may be read separately or in either order, and all of this content is not a requirement for working extensively with Gall agents.  If you're just starting off, wet gates (genericity) make the most sense to have in your toolkit now.


##  Genericity

Polymorphism is a programming concept that allows a piece of code to use different types at different times.  It's a common technique in most languages to make code that can be reused for many different situations, and Hoon is no exception.

A dry gate is the kind of gate that you're already familiar with:  a one-armed [core](https://urbit.org/docs/glossary/core/) with a sample.  A wet gate is also a one-armed [core](https://urbit.org/docs/glossary/core/) with a sample, but there is a difference in how types are handled.  With a dry gate, when you pass in an argument and the code gets compiled, the type system will try to cast to the type specified by the gate; if you pass something that does not fit in the specified type, for example a `cord` instead of a `cell` you will get a `nest-fail` error.  When you pass arguments to a wet gate, their types are preserved and type analysis is done at the definition site of the gate rather than at the call site.

In other words, for a wet gate, we ask:  “Suppose this core was actually _compiled_ using the modified payload instead of the one it was originally built with?  Would the Nock formula we generated for the original template actually work for the modified `payload`?”  Basically, wet gates allow you to hot-swap code at runtime and see if it “just works”—they defer the actual substitution in the `sample`.  Wet gates are rather like [macros](https://en.wikipedia.org/wiki/Macro_%28computer_science%29) in this sense.

Consider a function like `++turn` which transforms each element of a list. To use `++turn`, we install a list and a transformation function in a generic core.  The type of the list we produce depends on the type of the list and the type of the transformation function.  But the Nock formulas for transforming each element of the list will work on any function and any list, so long as the function's argument is the list item.

A wet gate is defined by a [`|*` bartar](https://urbit.org/docs/hoon/reference/rune/bar#-bartar) rune rather than a `|=` bartis.  More generally, cores that contain wet arms **must** be defined using [`|@` barpat](https://urbit.org/docs/hoon/reference/rune/bar#-barpat) instead of `|%` barcen (`|*` expands to a `|@` core with `$` buc arm).  There is also [`|$` barbuc](https://urbit.org/docs/hoon/reference/rune/bar#-barbuc) which defines the wet gate mold builder (remember, we like gates that build gates).

In a nutshell, compare these two gates:

```hoon
> =dry |=([a=* b=*] [b a])

> =wet |*([a=* b=*] [b a])

> (dry %cat %dog)
[6.778.724 7.627.107]

> (wet %cat %dog)
[%dog %cat]
```

The dry gate does not preserve the type of `a` and `b`, but downcasts it to `*`; the wet gate does preserve the input types.  It is good practice to include a cast in all gates, even wet gates.  But in many cases the desired output type depends on the input type.  How can we cast appropriately? Often we can cast by example, using the input values themselves (using `^+` ketlus).

Wet gates are therefore used when incoming type information is not well known and needs to be preserved.  This includes parsing, building, and structuring arbitrary nouns.  (If you are familiar with them, you can think of C++'s templates and operator overloading, and Haskell's typeclasses.)  Wet gates are very powerful; they're enough rope to hang yourself with.  Don't use them unless you have a specific reason to do so.  (If you see `mull-*` errors then something has gone wrong with using wet gates.)

#### Exercise:  The Trapezoid Rule

The [trapezoid rule](https://en.wikipedia.org/wiki/Trapezoidal_rule) solves a definite integral.  It approximates the area under the curve by a trapezoid or (commonly) a series of trapezoids.  The rule requires a function as one of the inputs, i.e. it applies _for a specific function_.  We will use wet gates to accomplish this without stripping type information of the input gate core.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d1/Integration_num_trapezes_notation.svg/573px-Integration_num_trapezes_notation.svg.png)

<img src="https://latex.codecogs.com/svg.image?\large&space;\int_a^b&space;f(x)&space;\,&space;dx&space;\approx&space;\sum_{k=1}^N&space;\frac{f(x_{k-1})&space;&plus;&space;f(x_k)}{2}&space;\Delta&space;x_k&space;=&space;\tfrac{\Delta&space;x}{2}\left(f(x_0)&space;&plus;&space;2f(x_1)&plus;2f(x_2)&plus;&space;2f(x_3)&plus;2f(x_4)&plus;\cdots&plus;2f(x_{N-1})&space;&plus;&space;f(x_N)\right)" title="https://latex.codecogs.com/svg.image?\large \int_a^b f(x) \, dx \approx \sum_{k=1}^N \frac{f(x_{k-1}) + f(x_k)}{2} \Delta x_k = \tfrac{\Delta x}{2}\left(f(x_0) + 2f(x_1)+2f(x_2)+ 2f(x_3)+2f(x_4)+\cdots+2f(x_{N-1}) + f(x_N)\right)" />

<!--
\int_a^b f(x) \, dx \approx \sum_{k=1}^N \frac{f(x_{k-1}) + f(x_k)}{2} \Delta x_k = \tfrac{\Delta x}{2}\left(f(x_0) + 2f(x_1)+2f(x_2)+ 2f(x_3)+2f(x_4)+\cdots+2f(x_{N-1}) + f(x_N)\right)
-->

- Produce a trapezoid-rule integrator which accepts a wet gate (as a function of a single variable) and a list of _x_ values, and yields the integral as a `@rs` floating-point value.  (If you are not yet familiar with these, you may wish to skip ahead to the next lesson.)

```hoon
++  trapezint
  |*  [a=(list @rs) b=gate]
  =/  n  (lent a)
  =/  k  1
  =/  sum  .0
  |-  ^-  @rs
  ?:  =(+(k) n)  (add:rs sum (b (snag k a)))
  ?:  =(k 1)
    $(k +(k), sum (add:rs sum (b (snag k a))))
  $(k +(k), sum (mul:rs .2 (add:rs sum (b (snag k a)))))
```

The meat of this gate is concerned with correctly implementing the mathematical equation.  In particular, wetness is required because `b` can be _any_ gate (although it should only be a gate with one argument, lest the whole thing `mull-grow` fail).  If you attempt to create the equivalent dry gate (`|=` bartis), Hoon fails to build it with a `nest-fail` due to the loss of type information from the gate `b`.

#### Exercise:  `++need`

Wet gates and wet cores are used in Hoon when type information isn't well-characterized ahead of time, as when constructing `++map`s or `++set`s.  For instance, almost all of the arms in `++by` and `++in`, as well as most `++list` tools, are wet gates.

Let's take a look at a particular wet gate from the Hoon standard library, [`++need`](https://urbit.org/docs/hoon/reference/stdlib/2a#need).  `++need` works with a `unit` to produce the value of a successful `unit` call, or crash on `~`.  (As this code is already defined in your `hoon.hoon`, you do not need to define it in the Dojo to use it.)

```hoon
++  need                                                ::  demand
  |*  a=(unit)
  ?~  a  ~>(%mean.'need' !!)
  u.a
```

Line by line:

1. `|*  a=(unit)` declares a wet gate which accepts a `unit`.

2. `?~  a  ~<(%mean.'need' !!)`.  If `a` is empty, `~`, then the `unit` cannot be unwrapped.  Crash with [`!!` zapzap](https://urbit.org/docs/hoon/reference/rune/zap#-zapzap), but use [`~<` siggal](https://urbit.org/docs/hoon/reference/rune/sig#-siggal) to hint to the runtime interpreter how to handle the crash.

3. `u.a` returns the value in the `unit` since we now know it exists.

`++need` is wet because we don't want to lose type information when we extract from the `unit`.


##  Variadicity

Dry polymorphism works by substituting cores.  Typically, one core is used as the interface definition, then replaced with another core which does something useful.

For core `b` to nest within core `a`, the batteries of `a` and `b` must have the same tree shape, and the product of each `b` arm must nest within the product of the `a` arm.  Wet arms (described above) are not compatible unless the Hoon expression is exactly the same.  But for dry cores we also apply a payload test that depends on the rules of variance.

There are four kinds of cores: `%gold`, `%iron`, `%zinc`, and `%lead`. You are able to use core-variance rules to create programs which take other programs as arguments. Which particular rules depends on which kind of core your program needs to complete.

Before we embark on the following discussion, we want you to know that [variance](https://en.wikipedia.org/wiki/Covariance_and_contravariance_%28computer_science%29) is a bright-line idea, much like cores themselves, which once you “get” illuminates you further about Hoon-nature.  For the most part, though, you don't need to worry about core variance much unless you are writing kernel code, since it impinges on how cores evaluate with other cores as inputs.  Don't sweat it if it takes a while for core variance to click for you.  (If you want to dig into resources, check out Meyer type theory.  The rules should make sense if you think about them intuitively and don't get hung up on terminology.)  [Vadzim Vysotski](https://vadzimv.dev/2019/10/01/generic-programming-part-1-introduction.html) and [Jamie Kyle](https://medium.com/@thejameskyle/type-systems-covariance-contravariance-bivariance-and-invariance-explained-35f43d1110f8) explain the theory of type system variance accessibly, while [Eric Lippert](https://archive.ph/QmiqB) provides a more technical description.  There are many wrinkles that particular languages, such as object-oriented programming languages, introduce which we can elide here.

What trips learners up about variance is that **variance rules apply to the input and output types of a core, not directly to the core itself**.  A core has a variance property, but that property doesn't manifest until cores are used together with each other.

Variance describes the four possible relationships that type rules are able to have to each other.  Hoon imaginatively designates these by metals.  Briefly:

1. **Covariance (`%zinc`)** means that more specific types nest inside of more generic types:  it's like claiming that a tree is a plant.

2. **Contravariance (`%iron`)** means that more generic types are expected to nest inside of less specific types:  this is like claiming that a plant is a tree.  It may be true, but it may break things.

3. **Bivariance (`%lead`)** means that we can allow both covariant and contravariant instances:  use a tree, use a plant, use a vine.  If it works, it works.

4. **Invariance (`%gold`)** means that types must mutually nest compatibly:  a tree is a tree is a tree.

The default is `%gold`; a `%gold` core can be cast or converted to any metal, and any metal can be cast or converted to `%lead`.

<!-- TODO would be nice to explain similar to aura nesting rules, but at the core level -->

### `%zinc` Covariance

Covariance means that specific types nest inside of generic types:  `tree` nests inside of `plant`.  Covariant data types are sources, or read-only values.

A zinc core `z` has a read-only sample (payload head, `+6.z`) and an opaque context (payload tail, `+7.z`).  (_Opaque_ here means that the faces and arms are not exported into the namespace, and that the values of faces and arms can't be written to.  The object in question can be replaced by something else without breaking type safety.)  A core `y` which nests within it must be a gold or zinc core, such that `+6.y` nests within `+6.z`.  Hence, **covariant**.

<!-- If type `x` nests within type `xx`, and type `y` nests within type `yy`, then a core accepting `yy` and producing `x` nests within an iron core accepting `y` and producing `xx`. TODO not adjusted yet -->

You can read from the sample of a `%zinc` core, but not change it:

```hoon
> =mycore ^&(|=(a=@ 1))

> a.mycore
0

> mycore(a 22)
-tack.a
-find.a
ford: %slim failed:
ford: %ride failed to compute type:
```

Informally, a function fits an interface if the function has a more specific result and/or a less specific argument than the interface.

The [`^&` ketpam](https://urbit.org/docs/hoon/reference/rune/ket#-ketpam) rune converts a core to a `%zinc` covariant core.

### `%iron` Contravariance

Contravariance means that generic types nest inside of specific types.  Contravariant data types are sinks, or write-only values.

An `%iron` core `i` has a write-only sample (payload head, `+6.i`) and an opaque context (payload tail, `+7.i`).  A core `j` which nests within it must be a `%gold` or `%iron` core, such that `+6.i` nests within `+6.j`. Hence, **contravariant**.

If type `x` nests within type `xx`, and type `y` nests within type `yy`, then a core accepting `yy` and producing `x` nests within an iron core accepting `y` and producing `xx`.

Informally, a function fits an interface if the function has a more specific result and/or a less specific argument than the interface.

The [`|~` barsig](https://urbit.org/docs/hoon/reference/rune/bar#-barsig) rune produces an iron gate.  The [`^|` ketbar](https://urbit.org/docs/hoon/reference/rune/ket#-ketbar) rune converts a `%gold` invariant core to an iron core.

### `%lead` Bivariance

Bivariance means that both covariance and contravariance apply.  Bivariant data types have an opaque `payload` that can neither be read or written to.

A lead core `l` has an opaque `payload` which can be neither read nor written to.  There is no constraint on the payload of a core `m` which nests within it.  Hence, **bivariant**.

If type `x` nests within type `xx`, a lead core producing `x` nests within a lead core producing `xx`.

Bivariant data types are neither readable nor writeable, but have no constraints on nesting.  These are commonly used for `/mar` marks and `/sur` structure files.  They are useful as examples which produce types.

Informally, a more specific generator can be used as a less specific generator.

The [`|?` barwut](https://urbit.org/docs/hoon/reference/rune/bar#-barwut) rune produces a lead trap.  The [`^?` ketwut](https://urbit.org/docs/hoon/reference/rune/ket#-ketwut) rune converts any core to a `%lead` bivariant core.

### `%gold` Invariance

Invariance means that type nesting is disallowed.  Invariant data types have a read-write `payload`.

A `%gold` core `g` has a read-write payload; another core `h` that nests within it (i.e., can be substituted for it) must be a `%gold` core whose `payload` is mutually compatible (`+3.g` nests in `+3.h`, `+3.h` nests in `+3.g`).  Hence, **invariant**.

By default, cores are `%gold` invariant cores.

### Examples

#### `%iron` Contravariant Polymorphism

Let's take a look at a [gate](https://urbit.org/docs/glossary/gate/) from the Hoon standard library as an example; we'll be passing a few different types in. The code below is an excerpt from `hoon.hoon` and, as such, will not run as-is by itself.  TODO

`++fold` is a _wet_ gate that takes two arguments and produces a _dry_ gate.

```hoon
++  fold  
   |*  [state=mold elem=mold]
   |=  [[st=state xs=(list elem)] f=$-([state elem] state)]
   ^-  state
   |-
   ?~  xs  st
   $(xs t.xs, st (f st i.xs))
```

On the first line of the [arm](https://urbit.org/docs/glossary/arm/), we use `|*` to create a wet gate, with two arguments: `state` and `elem`. These arguments are both `mold`s; that is to say, they are type definitions. We use this to define the types used in the dry gate.

```hoon
|=  [[st=state xs=(list elem)] f=$-([state elem] state)]
```

Here we begin to define our dry gate. The first thing the dry gate takes is a cell, `[st=state xs=(list elem)]`. `state` is the type we passed in first to our wet gate, and will be the type that is produced by the dry gate. `st` is a value of type `state` that will be used as an accumulator, and will be the final value returned when the dry gate is called.

The second argument is `f=$-([state elem] state)`. `$-` is a rune that takes two type arguments, `[state elem]` and `state` in this case, and produces a `mold` of a gate that maps from the first type operand to the second. This will match a gate we provide `fold` for how to map from both the `state`.

The rest of the dry gate is straightforward:

```hoon
   ^-  state
   |-
   ?~  xs  st
   $(xs t.xs, st (f st i.xs))
```

We ensure the output is of type `state`. Then we check if `xs`, the list, is empty. If it is, we return `st`. Otherwise, we call `f` on `i.xs`, the head of the list, and set `xs` to be the tail of the list and repeat.

Lets look at two examples of using `fold`.

```hoon
%+  (fold (list @) @)
 :-  ~
 ~[1 2 3 4]
|=  [s=(list @) e=@]
:_  s
(add 2 e)
```

Here we have a call to `fold` that we can see will produce a `(list @)` from a list of `@`. The first argument is a cell of `~` and `~[1 2 3 4]`.

The gate will simply add two to each element `e` and append that to the front of `s`. Let's run this in the `dojo`:

```hoon
> %+  (fold (list @) @)
   :-  ~
   ~[1 2 3 4]
  |=  [s=(list @) e=@]
  :_  s
  (add 2 e)
~[6 5 4 3]

~zod:dojo>
```

The list is in reverse order simply because it's easier to add to the front of a list than the end. If you needed it in the same order, you could just `flop` it.

But `fold` does not have to produce a `list`. Let's look at another example:

```hoon
%+  (fold @ @)
  [0 ~[1 2 3 4]]
|=  [s=@ e=@]
(add e s)
```

Here `fold` will produce a gate that takes an `atom` and applies a gate to a `list` of `atoms`. The difference here is that this call will produce a sum of the elements of the list, rather than the list itself.

The key takeaway from both of these examples is that the gates provided are _`%iron`-polymorphic_ with respect to the definition of the type in `fold`.  They are iron polymorphic because samples `s` and `e` nest under the types `state` and `elem`.  In the second case, that's because when we provided those to `++fold`, it was was stated they were `@` and `@`.  In the first case, we stated that `state` was `(list @)` and `elem` was `@`. In both cases, the sample of each gate nest inside the types defined when we called the wet gate `fold`.

#### `%lead` Bivariant Polymorphism

- Calculate the Fibonacci series using `%lead` and `%iron` cores.

This program produces a list populated by the first ten elements of the `++fib` arm.  It consists of five arms; in brief:

- `++fib` is a trap (core with no sample and default arm `$` buc)
- `++stream` is a mold builder that produces a trap, a function with no argument.  This trap can yield a value or a `~`.
- `++stream-type` is a wet gate that produces the type of items stored in `++stream`.
- `++to-list` is a wet gate that converts a `++stream` to a `list`.
- `++take` is a wet gate that takes a `++stream` and an atom and yields a modified subject (!) and another trap of `++stream`'s type.

**`/gen/fib.hoon`**

```hoon
=<  (to-list (take fib 10))
|%
++  stream
  |*  of=mold
  $_  ^?   |.
  ^-  $@(~ [item=of more=^$])
  ~
++  stream-type
  |*  s=(stream)
  $_  =>  (s)
  ?~  .  !!
  item
++  to-list
  |*  s=(stream)
  %-  flop
  =|  r=(list (stream-type s))
  |-  ^+  r
  =+  (s)
  ?~  -  r
  %=  $
    r  [item r]
    s  more
  ==
++  take
  |*  [s=(stream) n=@]
  =|  i=@
  ^+  s
  |.
  ?:  =(i n)  ~
  =+  (s)
  ?~  -  ~
  :-  item
  %=  ..$
    i  +(i)
    s  more
  ==
++  fib
  ^-  (stream @ud)
  =+  [p=0 q=1]
  |.  :-  q
  %=  .
    p  q
    q  (add p q)
  ==
--
```

Let's examine each arm in detail.

##### `++stream`

```hoon
++  stream
  |*  of=mold
  $_  ^?  |.
  ^-  $@(~ [item=of more=^$])
  ~
```

`++stream` is a mold-builder. It's a wet gate that takes one argument, `of`, which is a `mold`, and produces a lead trap—a function with no `sample` and an arm `$` buc, with opaque `payload`.

`$_` buccab is a rune that produces a type from an example; `^?` ketwut converts (casts) a core to lead; `|.` bardot forms the trap.  So to follow this sequence we read it backwards:  we create a trap, convert it to a lead trap (making its payload inaccessible), and then use that lead trap as an example from which to produce a type.

With the line `^- $@(~ [item=of more=^$])`, the output of the trap will be cast into a new type.  `$@` bucpat is the rune to describe a data structure that can either be an atom or a cell.  The first part describes the atom, which here is going to be `~`.  The second part describes a cell, which we define to have the head of type `of` with the face `item`, and a tail with a face of `more`.  The expression `^$` is not a rune (no children), but rather a reference to the enclosing wet gate, so the tail of this cell will be of the same type produced by this wet gate.

The final `~` here is used as the type produced when initially calling this wet gate.  This is valid because it nests within the type we defined on the previous line.

Now you can see that a `++stream` is either `~` or a pair of a value of some type and a `++stream`.  This type represents an infinite series.

##### `++stream-type`

```hoon
++  stream-type
  |*  s=(stream)
  $_  =>  (s)
  ?~  .  !!
  item
```

`++stream-type` is a wet gate that produces the type of items stored in the `stream` arm.  The `(stream)` syntax is a shortcut for `(stream *)`; a stream of some type.

Calling a `++stream`, which is a trap, will either produce `item` and `more` or it will produce `~`. If it does produce `~`, the `++stream` is empty and we can't find what type it is, so we simply crash with `!!` zapzap.

##### `++take`

```hoon
++  take
  |*  [s=(stream) n=@]
  =|  i=@
  ^+  s
  |.
  ?:  =(i n)  ~
  =+  (s)
  ?~  -  ~
  :-  item
  %=  ..$
    i  +(i)
    s  more
  ==
```

`++take` is another wet gate. This time it takes a `++stream` `s` and an atom `n`. We add an atom to the subject and then make sure that the trap we are creating is going to be of the same type as `s`, the `++stream` we passed in.

If `i` and `n` are equal, the trap will produce `~`.  If not, `s` is called and has its result put on the front of the subject.  If its value is `~`, then the trap again produces `~`.  Otherwise the trap produces a cell of `item`, the first part of the value of `s`, and a new trap that increments `i`, and sets `s` to be the `more` trap which produces the next value of the `++stream`.  The result here is a `++stream` that will only ever produce `n` items, even if the stream otherwise would have been infinite.

##### `++take`

```hoon
++  to-list
  |*  s=(stream)
  %-  flop
  =|  r=(list (stream-type s))
  |-  ^+  r
  =+  (s)
  ?~  -  r
  %=  $
    r  [item r]
    s  more
  ==
```

`++to-list` is a wet gate that takes `s`, a `++stream`, only here it will, as you may expect, produce a `list`.  The rest of this wet gate is straightforward but we can examine it quickly anyway.  As is the proper style, this list that is produced will be reversed, so `flop` is used to put it in the order it is in the stream.  Recall that adding to the front of a list is cheap, while adding to the back is expensive.

`r` is added to the subject as an empty `list` of whatever type is produced by `s`.  A new trap is formed and called, and it will produce the same type as `r`.  Then `s` is called and has its value added to the subject. If the result is `~`, the trap produces `r`.  Otherwise, we want to call the trap again, adding `item` to the front of `r` and changing `s` to `more`.  Now the utility of `take` should be clear.  We don't want to feed `to-list` an infinite stream as it would never terminate.

##### `++fib`

```hoon
++  fib
  ^-  (stream @ud)
  =+  [p=0 q=1]
  |.  :-  q
  %=  .
    p  q
    q  (add p q)
  ==
```

The final arm in our core is `++fib`, which is a `++stream` of `@ud` and therefore is a lead core.  Its subject contains `p` and `q`, which will not be accessible outside of this trap, but because of the `%=` cenhep will be retained in their modified form in the product trap.  The product of the trap is a pair (`:-` colhep) of an `@ud` and the trap that will produce the next `@ud` in the Fibonacci series.

```hoon
=<  (to-list (take fib 10))
```

Finally, the first line of our program will take the first 10 elements of `fib` and produce them as a list.

```unknown
~[1 1 2 3 5 8 13 21 34 55]
```

This example is a bit overkill for simply calculating the Fibonacci series, but it nicely illustrates how you might use iron cores. Instead of `++fib`, you could supply any infinite sequence and `++stream` would correctly handle it.
