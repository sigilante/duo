---
title: Gates
nodes: 288, 299
objectives:
  - "Distinguish dry and wet gates."
  - "Describe use cases for wet gates (using genericity)."
  - "Enumerate and distinguish use cases for dry cores (using variance):"
  - "- Covariant (`%zinc`)"
  - "- Contravariant (`%iron`)"
  - "- Bivariant (`%lead`)"
  - "- Invariant (`%gold`)"
---

#   Adaptive Cores

Cores can expose and operate with many different assumptions about their inputs and structure.  `[battery payload]` describes the top-level structure of a core, but within that we already know other requirements can be enforced, like `[battery [sample context]]`.  Cores can also expose and operate on their input values with different relationships.  This lesson is concerned with examining _polymorphism_, which allows flexibility in type, and _variance_, which allows cores to use different sets of rules as they evaluate.

If cores never changed, we wouldn't need polymorphism.  Of course, nouns are immutable and never change, but we use them as templates to construct new nouns around.

Suppose we take a core, a cell `[battery payload]`, and replace `payload` with a different noun.  Then, we invoke an arm from the battery.

Is this legal?  Does it make sense?  Every function call in Hoon does this, so we'd better make it work well.

The full core stores _both_ payload types:  the type that describes the `payload` currently in the core, and the type that the core was compiled with.

In the Bertrand Meyer tradition of type theory, there are two forms of polymorphism:  _variance_ and _genericity_.  In Hoon this choice is per core:  a core can be either `%wet` or `%dry`.  Dry polymorphism relies on variance; wet polymorphism relies on genericity.

This lesson discusses both genericity and variance for core management.  These two sections may be read separately or in either order, and all of this content is not a requirement for working extensively with Gall agents.  If you're just starting off, wet gates (genericity) make the most sense to have in your toolkit now.


##  Genericity

Polymorphism is a programming concept that allows a piece of code to use different types at different times.  It's a common technique in most languages to make code that can be reused for many different situations, and Hoon is no exception.

For a wet core, we ask:  “Suppose this core was actually _compiled_ using the modified payload instead of the one it was originally built with?  Would the Nock formula we generated for the original template actually work for the modified `payload`?”  Basically, wet gates allow you to hot-swap code at runtime and see if it “just works”.

Wet gates essentially defer the actual substitution in the `sample`, using the Hoon expression of the `payload` essentially as a macro.  Consider a function like `++turn` which transforms each element of a list. To use `++turn`, we install a list and a transformation function in a generic core.  The type of the list we produce depends on the type of the list and the type of the transformation function.  But the Nock formulas for transforming each element of the list will work on any function and any list, so long as the function's argument is the list item.

Wet gates are very powerful; they're enough rope to hang yourself with.  Don't use them unless you have a specific reason to do so.  (If you see `mull-*` errors then something has gone wrong with using wet gates.)

TODO


Let's take a look at a gate from the Hoon standard library as an example; we'll be passing a few different types in. The code below is an excerpt from hoon.hoon and, as such, will not run as-is by itself.



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

TODO kind of like aura nesting rules, but at the core level

### `%zinc` Covariance

Covariance means that specific types nest inside of generic types:  `tree` nests inside of `plant`.  Covariant data types are sources, or read-only values.

A zinc core `z` has a read-only sample (payload head, `+6.z`) and an opaque context (payload tail, `+7.z`).  (_Opaque_ here means that the faces and arms are not exported into the namespace, and that the values of faces and arms can't be written to.  The object in question can be replaced by something else without breaking type safety.)  A core `y` which nests within it must be a gold or zinc core, such that `+6.y` nests within `+6.z`.  Hence, **covariant**.

If type `x` nests within type `xx`, and type `y` nests within type `yy`, then a core accepting `yy` and producing `x` nests within an iron core accepting `y` and producing `xx`. TODO not adjusted yet

Informally, a function fits an interface if the function has a more specific result and/or a less specific argument than the interface.

The [`^&` ketpam](https://urbit.org/docs/hoon/reference/rune/ket#-ketpam) rune converts a core to a `%zinc` covariant core.

### `%iron` Contravariance

Contravariance means that generic types nest inside of specific types.  Contravariant data types are sinks, or write-only values.

An `%iron` core `i` has a write-only sample (payload head, `+6.i`) and an opaque context (payload tail, `+7.i`).  A core `j` which nests within it must be a `%gold` or `%iron` core, such that `+6.i` nests within `+6.j`. Hence, **contravariant**.

If type `x` nests within type `xx`, and type `y` nests within type `yy`, then a core accepting `yy` and producing `x` nests within an iron core accepting `y` and producing `xx`.

Informally, a function fits an interface if the function has a more specific result and/or a less specific argument than the interface.

The [`^|` ketbar](https://urbit.org/docs/hoon/reference/rune/ket#-ketbar) rune converts a `%gold` invariant core to an iron core.

### `%lead` Bivariance

Bivariance means that both covariance and contravariance apply.  Bivariant data types have an opaque `payload` that can neither be read or written to.

A lead core `l` has an opaque `payload` which can be neither read nor written to.  There is no constraint on the payload of a core `m` which nests within it.  Hence, **bivariant**.

If type `x` nests within type `xx`, a lead core producing `x` nests within a lead core producing `xx`.

Bivariant data types are neither readable nor writeable.  So what's the point?  These are commonly used for `/mar` marks and `/sur` structure files.  They are useful as examples which produce types.

Informally, a more specific generator can be used as a less specific generator.

The [`^?` ketwut](https://urbit.org/docs/hoon/reference/rune/ket#-ketwut) rune converts any core to a `%lead` bivariant core.

### `%gold` Invariance

Invariance means that type nesting is disallowed.  Invariant data types have a read-write `payload`.

A `%gold` core `g` has a read-write payload; another core `h` that nests within it (i.e., can be substituted for it) must be a `%gold` core whose `payload` is mutually compatible (`+3.g` nests in `+3.h`, `+3.h` nests in `+3.g`).  Hence, **invariant**.

By default, cores are `%gold` invariant cores.

### Examples

#### `%iron` Contravariant Polymorphism

https://urbit.org/docs/hoon/hoon-school/iron-polymorphism

TODO

#### `%lead` Bivariant Polymorphism

https://urbit.org/docs/hoon/hoon-school/lead-polymorphism

TODO
