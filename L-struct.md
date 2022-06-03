---
title: Data Structures
nodes: 183
objectives:
  - "Explain what an Urbit ship is."
  - "Distinguish a fakeship from a liveship."
  - "Pronounce ASCII characters per standard Hoon developer practice."
---

#   Data Structures


---

#   Type Checking

in-depth atom stuff
https://urbit.org/docs/hoon/hoon-school/type-checking-and-type-inference

## Auras as 'Soft' Types

It's important to understand that Hoon's type system doesn't enforce auras as strictly as it does other types. Auras are 'soft' type information. To see how this works, we'll take you through the process of converting the aura of an atom to another aura.

Hoon makes **some** effort to enforce that the correct aura is produced by an expression:

```unknown
> ^-(@ud 0x10)
nest-fail

> ^-(@ud 0b10)
nest-fail

> ^-(@ux 100)
nest-fail
```

But there are ways around this. First, you can cast to a more general aura, as long as the current aura nests under the cast aura. E.g., `@ub` to `@u`, `@ux` to `@u`, `@u` to `@`, etc. By doing this you're essentially telling Hoon to throw away some aura information:

```unknown
> ^-(@u 0x10)
16

> ? ^-(@u 0x10)
  @u
16

> ^-(@u 0b10)
2

> ? ^-(@u 0b10)
  @u
2
```

In fact, you can cast any atom all the way to the most general case `@`:

```unknown
> ^-(@ 0x10)
16

> ? ^-(@ 0x10)
  @
16

> ^-(@ 0b10)
2

> ? ^-(@ 0b10)
  @
2
```

Anything of the general aura `@` can, in turn, be cast to more specific auras. We can show this by embedding a cast expression inside another cast:

```unknown
> ^-(@ud ^-(@ 0x10))
16

> ^-(@ub ^-(@ 0x10))
0b1.0000

> ^-(@ux ^-(@ 10))
0xa
```

Hoon uses the outermost cast to infer the type:

```unknown
> ? ^-(@ub ^-(@ 0x10))
  @ub
0b1.0000
```

As you can see, an atom with one aura can be converted to another aura. For a convenient shorthand, you can do this conversion with irregular cast syntax, e.g. `` `@ud` ``, rather than using the `^-` rune twice:

```unknown
> `@ud`0x10
16

> `@ub`0x10
0b1.0000

> `@ux`10
0xa
```

This is what we mean when we call auras 'soft' types. The above examples show that the programmer can get around the type system for auras by casting up to `@` and then back down to the specific aura, say `@ub`; or by casting with `` `@ub` `` for short.
