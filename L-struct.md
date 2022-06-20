---
title: Data Structures
nodes: 183
objectives:
  - "Explain what an Urbit ship is."
  - "Distinguish a fakeship from a liveship."
  - "Pronounce ASCII characters per standard Hoon developer practice."
---

#   Data Structures

### `set`

A `set` is rather like a `list` except that each entry can only be represented once.  As with a `map`, a `set` is typically associated with a particular type, such as `(set @ud)` for a collection of decimal values.  (`set`s also don't have an order, so they're basically a bag of unique values.)

`set` operations are provided by `++in`.  Most names are similar to `map`/`++by` operations when applicable.

`++sy` produces a `set` from a `list`:

```hoon
> =primes (sy ~[2 3 5 7 11 13])
```

`++put:in` adds a value to a `set` (and null-ops when the value is already present):

```hoon
> =primes (~(put in primes) 17)
> =primes (~(put in primes) 13)
```

`++del:in` removes a value from a `set`:

```hoon
> =primes (~(put in primes) 18)
> =primes (~(del in primes) 18)
```

`++has:in` checks for existence:

```hoon
> (~(has in primes) 15)
%.n
> (~(has in primes) 17)
%.y
```

`++tap:in` yields a `list` of the values:

```hoon
> ~(tap in primes)  
~[3 2 7 5 11 13 17]  
> (sort ~(tap in primes) lth)  
~[2 3 5 7 11 13 17]
```

`++run:in` applies a function across all values:

```hoon
> (~(run in primes) dec)  
{10 6 12 1 2 16 4}
```

### `unit` Redux (and `vase`)

We encountered the `unit` as a tool for distinguishing null results from actual zeroes:  “using a `unit` allows you to specify something that may not be there.”

You can build a `unit` using the tic special notation or [`++some`](https://urbit.org/docs/hoon/reference/stdlib/2a#some):

```
> `%mars
[~ %mars]
> (some %mars)
[~ u=%mars]
```

While `++got:by` is one way to get a value back without wrapping it in a `unit`, it's better practice to use the [`unit` logic](https://urbit.org/docs/hoon/reference/stdlib/2a) gates to manipulate gates to work correctly with `unit`s.

For one thing, use [`++need`](https://urbit.org/docs/hoon/reference/stdlib/2a#need) to unwrap a `unit`, or crash if the `unit` is `~` null.

```hoon
> =colors `(map @tas @ux)`(my ~[[%red 0xed.0a3f] [%yellow 0xfb.e870] [%green 0x1.a638] [%blue 0x66ff]])
> (need (~(get by colors) %yellow))
0xfb.e870
> (need (~(get by colors) %teal))  
dojo: hoon expression failed
```

Rather than unwrap a `unit`, one can modify gates to work with `unit`s directly even if they're not natively set up that way.  For instance, one cannot decrement a `unit` because `++dec` doesn't accept a `unit`.  [`++bind`](https://urbit.org/docs/hoon/reference/stdlib/2a#bind) can bind a non-`unit` function

```hoon
> (bind ((unit @ud) [~ 2]) dec)  
[~ 1]
> (bind (~(get by colors) %orange) red)  
[~ 0xff]
```

(There are several others tools listed [on that page](https://urbit.org/docs/hoon/reference/stdlib/2a) which may be potentially useful to you.)

A `+$vase` is a pair of type and value, such as that returned by `!>` zapgar.  A `vase` is useful when transmitting data in a way that may lose its type information.

### `jar` and `jug`

`map`s and `set`s are frequently used in the standard library and in the extended ecosystem (such as in `graph-store`).  There are a couple of common patterns which recur often enough that they have their own names:

- [`++jar`](https://urbit.org/docs/hoon/reference/stdlib/2o#jar) is a mold for a `map` of `list`s.

- [`++jug`](https://urbit.org/docs/hoon/reference/stdlib/2o#jug) is a mold for a `map` of `set`s.

(There's an example in the slides of a `jar`.)

These are supported by the [`++ja`](https://urbit.org/docs/hoon/reference/stdlib/2j#ja) core and the [`++ju`](https://urbit.org/docs/hoon/reference/stdlib/2j#ju) core.

mip

---

> ##  Adding an Arm
> 
> Add an arm to the door which calculates the linear function
> _a×x + b_.
> 
> Add another arm which calculates the derivative of the first
> quadratic function, _2×a×x + b_.
{: .challenge}

In the above example we created a door `poly` with sample `[a=@ud b=@ud c=@ud]`.  If we investigated, we would find that the initial value of each is `0`, the bunt value of `@ud`.  What if we wish to define a door with a chosen sample value directly?  We can make use of the `$_` rune, whose irregular form is simply `_`.  To create the door `poly` with the sample set to have certain values in the Dojo, we would write

```unknown
> =poly |_  [a=_5 b=_4 c=_3]
++  quad
  |=  x=@ud
  (add (add (mul a (mul x x)) (mul b x)) c)
--

> (quad:poly 2)  
31
```
---

#   Type Checking

in-depth atom stuff
https://urbit.org/docs/hoon/hoon-school/type-checking-and-type-inference
https://urbit.org/docs/hoon/hoon-school/lists#an-aside-about-casting


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


Note: there is currently a type system issue that causes some of these functions to fail when passed a list b after some type inference has been performed on b. For an illustration of the bug, let's set b to be a (list @) of ~[11 22 33 44] in the Dojo:

> =b `(list @)`~[11 22 33 44]

> b
~[11 22 33 44]
Now let's use ?~ to prove that b isn't null, and then try to snag it:

> ?~(b ~ (snag 0 b))
nest-fail
The problem is that snag is expecting a raw list, not a list that is known to be non-null.

You can cast b to (list) to work around this:

> ?~(b ~ (snag 0 `(list)`b))
11

TODO with ?: here and refer to ?~ in the next lesson

(This can be done more idiomatically and more tersely using `?~` wutsig, which is introduced in the next module of Hoon School.)
