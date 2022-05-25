---
title: Gates
nodes: 234, 236, 284
objectives:
  - "Explain what an Urbit ship is."
  - "Distinguish a fakeship from a liveship."
  - "Pronounce ASCII characters per standard Hoon developer practice."
---

#   Mathematics

All of the math we've done until this point relied on unsigned integers:  there was no negative value possible, and there were no numbers with a fractional part.  How can we work with mathematics that require more than just bare unsigned integers?

TODO Gödel numbering etc.

##  Floating-Point Mathematics

A number with a fractional part is called a “floating-point number” in computer science.  This derives from its solution to the problem of representing the part less than one.

Consider for a moment how you would represent a regular decimal fraction if you only had integers available.  You would probably adopt one of three strategies:

1. [**Rational numbers**](https://en.wikipedia.org/wiki/Fraction).  Track whole-number ratios like fractions.  Thus 1.25 = 5/4, thence the pair `(5, 4)`.  Two numbers have to be tracked:  the numerator and the denominator.
2. [**Fixed-point**](https://en.wikipedia.org/wiki/Fixed-point_arithmetic).  Track the value in smaller fixed units (such as thousandths).  By defining the base unit to be ¹/₁₀₀₀, 1.25 may be written 1250.  One number needs to be tracked:  the value in terms of the scale.  (This is equivalent to rational numbers with only a fixed denominator allowed.)
3. [**Floating-point**](https://en.wikipedia.org/wiki/Floating-point_arithmetic).  Track the value at adjustable scale.  In this case, one needs to represent 1.25 as something like 125 × 10¯².  Two numbers have to be tracked:  the significand (125) and the exponent (-2).

Most systems use floating-point mathematics to solve this problem.  For instance, single-precision floating-point mathematics designate one bit for the sign, eight bits for the exponent (which has 127 subtracted from it), and twenty-three bits for the significand.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d2/Float_example.svg/640px-Float_example.svg.png)

This number, `0b11.1110.0010.0000.0000.0000.0000.0000`, is converted to decimal as (-1)⁰ × 2¹²⁴¯¹²⁷ × 1.25 = 2¯³ × 1.25 = 0.15625.

(If you want to explore the bitwise representation of values, [this tool](https://evanw.github.io/float-toy/) allows you to tweak values directly and see the results.)

### Hoon Operations

Hoon utilizes the [IEEE 754](https://en.wikipedia.org/wiki/IEEE_754) implementation of floating-point math for four bitwidth representations.

| Aura | Meaning | Example |
| ---- | ------- | ------- |
| `@r` | Floating-point value |  |
| `@rh` | Half-precision 16-bit mathematics | `.~~4.5` |
| `@rs` | Single-precision 32-bit mathematics | `.4.5` |
| `@rd` | Double-precision 64-bit mathematics | `.~4.5` |
| `@rq` | Quadruple-precision 128-bit mathematics | `.~~~4.5` |

There are also a few molds which can represent the separate values of the FP representation.  These are used internally but mostly don't appear in userspace code.

As the arms for the four `@r` auras are identical within their appropriate core, we will use [`@rs` single-precision floating-point mathematics](https://urbit.org/docs/hoon/reference/stdlib/3b#rs) to demonstrate all operations.

#### Conversion to and from other auras

Any `@ud` unsigned decimal integer can be directly cast as an `@rs`.

```hoon
> `@ud`.1
1.065.353.216
```

However, as you can see here, the conversion is not “correct” for the perceived values.  Examining the `@ux` hexadecimal and `@ub` binary representation shows why:

```hoon
> `@ux`.1
0x3f80.0000

> `@ub`.1
0b11.1111.1000.0000.0000.0000.0000.0000
```

If you refer back to the 32-bit floating-point example above, you'll see why:  to represent one exactly, we have to use 1.0 = (-1)⁰ × 2¹²⁷¯¹²⁷ × 1 and thus `0b11.1111.1000.0000.0000.0000.0000.0000`.

So to carry out this conversion from `@ud` to `@rs` correctly, we should use the [`++sun:rs`](https://urbit.org/docs/hoon/reference/stdlib/3b#sunrs) arm.

```hoon
> (sun:rs 1)
.1
```

To go the other way requires us to use an algorithm for converting an arbitrary number with a fractional part back into `@ud` unsigned integers.  The `++fl` named tuple representation serves this purpose, and uses the [Dragon4 algorithm](https://dl.acm.org/doi/10.1145/93548.93559) to accomplish the conversion:

```hoon
> (drg:rs .1)
[%d s=%.y e=--0 a=1]

> (drg:rs .3.1415926535)
[%d s=%.y e=-7 a=31.415.927]

> (drg:rs .1000)
[%d s=%.y e=--3 a=1]
```

It's up to you to decide how to handle this result, however!  Perhaps a better option for many cases is to round the answer to an `@s` integer with [`++toi:rs](https://urbit.org/docs/hoon/reference/stdlib/3b#toirs):

```hoon
> (toi:rs .3.1415926535)
[~ --3]
```

(`@s` signed integer math is discussed below.)

#### Floating-point specific operations

As with aura conversion, the standard mathematical operators don't work for `@rs`:

```hoon
> (add .1 1)
1.065.353.217

> `@rs`(add .1 1)
.1.0000001
```

The `++rs` core defines a set of `@rs`-affiliated operations which should be used instead:

```hoon
> (add:rs .1 .1)
.2
```

This includes:

- [`++add:rs`](https://urbit.org/docs/hoon/reference/stdlib/3b#addrs), addition
- [`++sub:rs`](https://urbit.org/docs/hoon/reference/stdlib/3b#subrs), subtraction
- [`++mul:rs`](https://urbit.org/docs/hoon/reference/stdlib/3b#mulrs), multiplication
- [`++div:rs`](https://urbit.org/docs/hoon/reference/stdlib/3b#divrs), division
- [`++gth:rs`](https://urbit.org/docs/hoon/reference/stdlib/3b#gthrs), greater than
- [`++gte:rs`](https://urbit.org/docs/hoon/reference/stdlib/3b#gters), greater than or equal to
- [`++lth:rs`](https://urbit.org/docs/hoon/reference/stdlib/3b#lthrs), less than
- [`++lte:rs`](https://urbit.org/docs/hoon/reference/stdlib/3b#lters), less than or equal to
- [`++equ:rs`](https://urbit.org/docs/hoon/reference/stdlib/3b#equrs), check equality (but not nearness!)
- [`++sqt:rs`](https://urbit.org/docs/hoon/reference/stdlib/3b#sqtrs), square root

#### Exercise:  `++is-close`

The `++equ:rs` arm checks for complete equality of two values.  The downside of this arm is that it doesn't find very close values:

```hoon
> (equ:rs .1 .1)
%.y

> (equ:rs .1 .0.9999999)
%.n
```

Produce an arm which check for two values to be close to each other by an absolute amount.  It should accept three values:  `a`, `b`, and `atol`.  It should return the result of the following comparison:

<img src="https://latex.codecogs.com/svg.image?\large&space;|a-b|&space;\leq&space;\texttt{atol}" title="https://latex.codecogs.com/svg.image?\large |a-b| \leq \texttt{atol}" />


##  Signed Integer Mathematics

Similar to floating-point representations, [signed integer](https://en.wikipedia.org/wiki/Signed_number_representations) representations use an internal bitwise convention to indicate whether a number should be treated as having a negative sign in front of the magnitude or not.  There are several ways to represent signed integers:

1. [**Sign-magnitude**](https://en.wikipedia.org/wiki/Signed_number_representations#Sign%E2%80%93magnitude).  Use the first bit in a fixed-bit-width representation to indicate whether the whole should be multiplied by -1, e.g. `0010.1011` for 43₁₀ and `1010.1011` for -43₁₀.  (This is similar to the floating-point solution.)
2. [**One's complement**](https://en.wikipedia.org/wiki/Ones%27_complement).  Use the bitwise `NOT` operation to represent the value, e.g. `0010.1011` for 43₁₀ and `1101.0100` for -43₁₀.  This has the advantage that arithmetic operations are trivial, e.g. 43₁₀-41₁₀ = `0010.1011` + `1101.0110` = `1.0000.0001`, end-around carry the overflow to yield `0000.0010` = 2.  (This is commonly used in hardware.)
3. [**Offset binary**](https://en.wikipedia.org/wiki/Offset_binary).  This represents a number normally in binary _except_ that it counts from a point other than zero, like `-256`.
4. [**ZigZag**](https://developers.google.com/protocol-buffers/docs/encoding?hl=en#signed-ints).  Positive signed integers correspond to even atoms of twice their absolute value, and negative signed integers correspond to odd atoms of twice their absolute value minus one.

TODO fix in main docs to ZigZag

There are tradeoffs in compactness of representation and efficiency of mathematical operations.

### Hoon Operations

`@u`-aura atoms are _unsigned_ values, but there is a complete set of _signed_ auras in the `@s` series.  ZigZag was chosen for Hoon's signed integer representation because it represents negative values with small absolute magnitude as short binary terms.

| Aura | Meaning | Example |
| ---- | ------- | ------- |
| `@s` | signed integer|  |
| `@sb` | signed binary | `--0b11.1000` (positive) |
|       |               | `--0b11.1000` (negative) |
| `@sd` | signed decimal | `--1.000.056` (positive) |
|       |                | `-1.000.056` (negative) |
| `@sx` | signed hexadecimal | `--0x5f5.e138` (positive) |
|       |                    | `-0x5f5.e138` (negative) |

The [`++si`](https://urbit.org/docs/hoon/reference/stdlib/3a#si) core supports signed-integer operations correctly.  However, unlike the `@r` operations, `@s` operations have different names (likely to avoid accidental mental overloading).

To produce a signed integer from an unsigned value, use [`++new:si`](https://urbit.org/docs/hoon/reference/stdlib/3a#newsi) with a sign flag, or simply use [`++sun:si`](https://urbit.org/docs/hoon/reference/stdlib/3a#sunsi)

```hoon
> (new:si & 2)
--2

> (new:si | 2)
-2

> `@sd`(sun:si 5)
--5
```

To recover an unsigned integer from a signed integer, use [`++old:si`](https://urbit.org/docs/hoon/reference/stdlib/3a#oldsi), which returns the magnitude and the sign.

```hoon
> (old:si --5)
[%.y 5]

> (old:si -5)
[%.n 5]
```

- [`++sum:si`](https://urbit.org/docs/hoon/reference/stdlib/3a#sumsi), addition
- [`++dif:si`](https://urbit.org/docs/hoon/reference/stdlib/3a#difsi), subtraction
- [`++pro:si`](https://urbit.org/docs/hoon/reference/stdlib/3a#prosi), multiplication
- [`++fra:si`](https://urbit.org/docs/hoon/reference/stdlib/3a#frasi), division
- [`++dul:si`](https://urbit.org/docs/hoon/reference/stdlib/3a#dulsi), modulus (remainder after division), b modulo a as `@u` TODO
- [`++rem:si`](https://urbit.org/docs/hoon/reference/stdlib/3a#remsi), modulus (remainder after division), b modulo a as `@s` TODO
- [`++abs:si`](https://urbit.org/docs/hoon/reference/stdlib/3a#abssi), absolute value
- [`++cmp:si`](https://urbit.org/docs/hoon/reference/stdlib/3a#synsi), test for greater value (as index, `>` → `--1`, `<` → `-1`, `=` → `--0`)


##  Date & Time Mathematics

Date and time calculations are challenging for a number of reasons:  What is the correct granularity for an integer to represent?  What value should represent the starting value?  How should time zones and leap seconds be handled?

One particularly complicating factor is that there is no [Year Zero](https://en.wikipedia.org/wiki/Year_zero); 1 B.C. is immediately followed by A.D. 1.
The Julian date system used in astronomy differs from standard time in this regard.

In computing, absolute dates are calculated with respect to some base value; we refer to this as the _epoch_.  Unix/Linux systems count time forward from Thursday 1 January 1970 00:00:00 UT, for instance.  Windows systems count in 10¯⁷ s intervals from 00:00:00 1 January 1601.  The Urbit epoch is `~292277024401-.1.1`, or 1 January 292,277,024,401 B.C.; since values are unsigned integers, no date before that time can be represented.

Time values, often referred to as _timestamps_, are commonly represented by the [UTC](https://www.timeanddate.com/time/aboututc.html) value.  Time representations are complicated by offset such as timezones, regular adjustments like daylight savings time, and irregular adjustments like leap seconds.  (Read [Dave Taubler's excellent overview](https://levelup.gitconnected.com/why-is-programming-with-dates-so-hard-7477b4aeff4c) of the challenges involved with calculating dates for further considerations, as well as [Martin Thoma's “What Every Developer Should Know About Time” (PDF)](https://zenodo.org/record/1443533/files/2018-10-06-what-developers-should-know-about-time.pdf).)

### Hoon Operations

A timestamp can be separated into the time portion, which is the relative offset within a given day, and the date portion, which represents the absolute day.

There are two molds to represent time in Hoon:  the `@d` aura, with `@da` for a full timestamp and `@dr` for an offset; and the [`+$date`](https://urbit.org/docs/hoon/reference/stdlib/2q#date)/[`+$tarp`](https://urbit.org/docs/hoon/reference/stdlib/2q#tarp) structure:

| Aura | Meaning | Example |
| ---- | ------- | ------- |
| `@da` | Absolute date | `~2022.1.1` |
|       |               | `~2022.1.1..1.1.1..0000` |
| `@dr` | Relative date (difference) | `~h5.m30.s12` |
|       |                            | `~d1000.h5.m30.s12..beef` |

```hoon
+$  date  [[a=? y=@ud] m=@ud t=tarp]
+$  tarp  [d=@ud h=@ud m=@ud s=@ud f=(list @ux)]
```

`now` returns the `@da` of the current timestamp (in UTC).

To go from a `@da` to a `+$tarp`, use [`++yell`](https://urbit.org/docs/hoon/reference/stdlib/3c#yell):

```hoon
> *tarp
[d=0 h=0 m=0 s=0 f=~]

> (yell now)
[d=106.751.991.821.625 h=22 m=58 s=10 f=~[0x44ff]]

> `tarp`(yell ~2014.6.6..21.09.15..0a16)
[d=106.751.991.820.172 h=21 m=9 s=15 f=~[0xa16]]

> (yell ~d20)
[d=20 h=0 m=0 s=0 f=~]
```

To go from a `@da` to a `+$date`, use [`++yore`](https://urbit.org/docs/hoon/reference/stdlib/3c#yore):

```hoon
> (yore ~2014.6.6..21.09.15..0a16)
[[a=%.y y=2.014] m=6 t=[d=6 h=21 m=9 s=15 f=~[0xa16]]]

> (yore now)
[[a=%.y y=2.022] m=5 t=[d=24 h=16 m=20 s=57 f=~[0xbaec]]]
```

To go from a `+$date` to a `@da`, use [`++year`](https://urbit.org/docs/hoon/reference/stdlib/3c#year):

```hoon
> (year [[a=%.y y=2.014] m=8 t=[d=4 h=20 m=4 s=57 f=~[0xd940]]])
~2014.8.4..20.04.57..d940

> (year (yore now))
~2022.5.24..16.24.16..d184
```

To go from a `+$tarp` to a `@da`, use [`++yule`](https://urbit.org/docs/hoon/reference/stdlib/3c#yule):

```hoon
> (yule (yell now))
0x8000000d312b148891f0000000000000

> `@da`(yule (yell now))
~2022.5.24..16.25.48..c915

> `@da`(yule [d=106.751.991.823.081 h=16 m=26 s=14 f=~[0xf727]])
~2022.5.24..16.26.14..f727
```

The Urbit date system correctly compensates for the lack of Year Zero:

```hoon
> ~0.1.1
~1-.1.1

> ~1-.1.1
~1-.1.1
```

The [`++yo`](https://urbit.org/docs/hoon/reference/stdlib/3c#yo) core contains constants useful for calculating time, but in general you should not hand-roll time or timezone calculations.


##  Unusual Bases

### Phonetic Base

The `@q` aura is similar to `@p` except for two details:  it doesn't obfuscate names (as planets do) and it can be used for any size of atom without adjust its width to fill the same size.  Prefixes and suffixes are in the same order as `@p`, however.  Thus:

```hoon
> `@q`0
.~zod

> `@q`256
.~marzod

> `@q`65.536
.~nec-dozzod

> `@q`4.294.967.296
.~nec-dozzod-dozzod

> `@q`(pow 2 128)
.~nec-dozzod-dozzod-dozzod-dozzod-dozzod-dozzod-dozzod-dozzod
```

`@q` auras can be used as sequential mnemonic markers for values.

The [`++po`](https://urbit.org/docs/hoon/reference/stdlib/4a#po) core contains tools for directly parsing `@q` atoms.

### Base-32 and Base-64

The base-32 representation uses the characters `0123456789abcdefghijklmnopqrstuv` to represent values.  The digits are separated into collections of five characters separated by `.` dot.

```hoon
> `@uv`0
0v0

> `@uv`100
0v34

> `@uv`1.000.000
0vugi0

> `@uv`1.000.000.000.000
0vt3a.aa400
```

The base-64 representation uses the characters `0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ-~` to represent values.  The digits are separated into collections of five characters separated by `.` dot.

```hoon
> `@uw`0
0w0

> `@uw`100
0w1A

> `@uw`1.000.000
0w3Q90

> `@uw`1.000.000.000
0wXCIE0

> `@uw`1.000.000.000.000
0wez.kFh00
```

### Arbitrary Bases

TODO
[`++fo`](https://urbit.org/docs/hoon/reference/stdlib/3a#fo)


##  Randomness & Entropy

### Entropy

You previously saw entropy introduced when we discussed stateful random number generation.  Let's dig into what's actually going on with entropy.

It is not straightforward for a computer, a deterministic machine, to produce an unpredictable sequence.  We can either use a source of true randomness (such as the third significant digit of chip temperature or another [hardware source](https://en.wikipedia.org/wiki/Hardware_random_number_generator)) or a source of artificial randomness (such as a sequence of numbers the user cannot predict).

For instance, consider the sequence _3 1 4 1 5 9 2 6 5 3 5 8 9 7 9 3_.  If you recognize the pattern as the constant π, you can predict the first few digits, but almost certainly not more than that.  The sequence is deterministic (as it is derived from a well-characterized mathematical process) but unpredictable (as you cannot _a priori_ guess what the next digit will be).

Computers often mix both deterministic processes (called “pseudorandom number generators”) with random inputs, such as the current timestamp, to produce high-quality random numbers for use in games, modeling, cryptography, and so forth.  The Urbit entropy value `eny` is derived from the underlying host OS's `/dev/urandom` device, which uses sources like keystroke typing latency to produce random bits.

### Randomness

Given a source of entropy to seed a random number generator, one can then use the [`++og`](https://urbit.org/docs/hoon/reference/stdlib/3d#og) door to produce various kinds of random numbers.  The basic operations of `++og` are described in [the lesson on subject-oriented programming](./N-subject.md).

#### Exercise:  Implement a random-number generator from scratch

The linear congruential random number generator produces a stream of random bits with a repetition period of 2³¹.  Numericist John Cook [explains how LCGs work](https://www.johndcook.com/blog/2017/07/05/simple-random-number-generator/):

> The linear congruential generator used here starts with an arbitrary seed, then at each step produces a new number by multiplying the previous number by a constant and taking the remainder by 2³¹-1.

**`/gen/lcg.hoon`**

```hoon
|=  n=@ud                 :: n is the number of bits to return
=/  z  20.220.524         :: z is the seed
=/  a  742.938.285        :: a is the multiplier
=/  e  31                 :: e is the exponent
=/  m  (sub (pow 2 e) 1)  :: modulus
=/  index  0
=/  accum  *@ub
|-  ^-  @ub
?:  =(index n)  accum
%=  $
  index  +(index)
  z      (mod (mul a z) m)
  accum  (cat 5 z accum)
==
```

Can you verify that `1`s constitute about half of the values in this bit stream, as Cook illustrates in Python?

#### Exercise:  Produce uniformly-distributed random numbers

Using entropy as the source, [produce uniform random numbers](https://www.omscs-notes.com/simulation/generating-uniform-random-numbers/):  that is, numbers in the range [0, 1] with equal likelihood to machine precision.

We use the LCG defined above, then chop out 23-bit slices using [`++rip`](https://urbit.org/docs/hoon/reference/stdlib/2c#rip) to produce each number, manually compositing the result into a valid floating-point number in the range [0, 1].  (We avoid producing special sequences like [`NaN`](https://en.wikipedia.org/wiki/NaN).)

**`/gen/uniform.hoon`**

```hoon
!:
=<
|=  n=@ud  :: n is the number of values to return
^-  (list @rs)
=/  values  (rip 5 (~(lcg gen 20.220.524) n))
=/  mask-clear           0b111.1111.1111.1111.1111.1111
=/  mask-fill   0b11.1111.0000.0000.0000.0000.0000.0000
=/  clears  (turn values |=(a=@rs (dis mask-clear a)))
(turn clears |=(a=@ (con mask-fill a)))
|%
++  gen
  |_  [z=@ud]
  ++  lcg
    |=  n=@ud                 :: n is the number of bits to return
    =/  a  742.938.285        :: a is the multiplier
    =/  e  31                 :: e is the exponent
    =/  m  (sub (pow 2 e) 1)  :: modulus
    =/  index  0
    =/  accum  *@ub
    |-  ^-  @ub
    ?:  =(index n)  accum
    %=  $
      index  +(index)
      z      (mod (mul a z) m)
      accum  (cat 5 z accum)
    ==
  --
--
```

Convert the above to a `%say` generator that can optionally accept a seed; if no seed is provided, use `eny`.

#### Exercise:  Produce normally-distributed random numbers

The normal distribution, or bell curve, describes the randomness of measurement.  The mean, or average value, is at zero, while points fall farther and farther away with increasingly less likelihood.

![A normal distribution curve with standard deviations marked](https://upload.wikimedia.org/wikipedia/commons/thumb/8/8c/Standard_deviation_diagram.svg/640px-Standard_deviation_diagram.svg.png)

One way to get from a uniform random number to a normal random number is [to use the uniform random number as the _cumulative distribution function_ (CDF)](https://xilinx.github.io/Vitis_Libraries/quantitative_finance/2022.1/guide_L1/RNGs/RNG.html), an index into “how far” the value is along the normal curve.

![A cumulative distribution function for three normal distributions](https://upload.wikimedia.org/wikipedia/commons/thumb/c/ca/Normal_Distribution_CDF.svg/640px-Normal_Distribution_CDF.svg.png)

This is an approximation which is accurate to one decimal place:

<img src="https://latex.codecogs.com/svg.image?\large&space;Z&space;=&space;\frac{U^{0.135}-(1-U)^{0.135}}{0.1975}" title="https://latex.codecogs.com/svg.image?\large Z = \frac{U^{0.135}-(1-U)^{0.135}}{0.1975}" />

where

- sgn is the signum or sign function.

<!--
$$
Z = \frac{U^{0.135}-(1-U)^{0.135}}{0.1975}
$$
-->

To calculate an arbitrary power of a floating-point number, we require a few transcendental functions, in particular the natural logarithm and exponentiation of base _e_.  The following helper core contains relatively inefficient but clear implementations of standard numerical methods.

**`/gen/normal.hoon`**

```hoon
!:
=<
|=  n=@ud  :: n is the number of values to return
^-  (list @rs)
=/  values  (rip 5 (~(lcg gen 20.220.524) n))
=/  mask-clear           0b111.1111.1111.1111.1111.1111
=/  mask-fill   0b11.1111.0000.0000.0000.0000.0000.0000
=/  clears    (turn values |=(a=@rs (dis mask-clear a)))
=/  uniforms  (turn clears |=(a=@ (con mask-fill a)))
(turn uniforms normal)
|%
++  sgn
  |=  x=@rs
  ^-  @rs
  ?:  (lth:rs x .0)
    .-1
  ?:  (gth:rs x .0)
    .1
  .0
++  factorial
  :: integer factorial, not gamma function
  |=  x=@rs
  ^-  @rs
  =/  t=@rs  .1
  |-  ^-  @rs
  ?:  |(=(x .1) (lth x .1))  t
  $(x (sub:rs x .1), t (mul:rs t x))
++  absrs
  |=  x=@rs  ^-  @rs
  ?:  (gth:rs x .0)
    x
  (sub:rs .0 x)
++  exp
  |=  x=@rs
  ^-  @rs
  =/  rtol  .1e-5
  =/  p   .1
  =/  po  .-1
  =/  i   .1
  |-  ^-  @rs
  ?:  (lth:rs (absrs (sub:rs po p)) rtol)  p
  $(i (add:rs i .1), p (add:rs p (div:rs (pow-n x i) (factorial i))), po p)
++  pow-n
  ::  restricted power, based on integers only
  |=  [x=@rs n=@rs]
  ^-  @rs
  ?:  =(n .0)  .1
  =/  p  x
  |-  ^-  @rs
  ?:  (lth:rs n .2)  p
  $(n (sub:rs n .1), p (mul:rs p x))
++  ln
  ::  natural logarithm, z > 0
  |=  z=@rs
  ^-  @rs
  =/  rtol  .1e-5
  =/  p   .0
  =/  po  .-1
  =/  i   .0
  |-  ^-  @rs
  ?:  (lth:rs (absrs (sub:rs po p)) rtol)
    (mul:rs (div:rs (mul:rs .2 (sub:rs z .1)) (add:rs z .1)) p)
  =/  term1  (div:rs .1 (add:rs .1 (mul:rs .2 i)))
  =/  term2  (mul:rs (sub:rs z .1) (sub:rs z .1))
  =/  term3  (mul:rs (add:rs z .1) (add:rs z .1))
  =/  term  (mul:rs term1 (pow-n (div:rs term2 term3) i))
  $(i (add:rs i .1), p (add:rs p term), po p)
++  powrs
  ::  general power, based on logarithms
  ::  x^n = exp(n ln x)
  |=  [x=@rs n=@rs]
  (exp (mul:rs n (ln x)))
++  normal
  |=  u=@rs
  (div:rs (sub:rs (powrs u .0.135) (powrs (sub:rs .1 u) .0.135)) .0.1975)
++  gen
  |_  [z=@ud]
  ++  lcg
    |=  n=@ud                 :: n is the number of bits to return
    =/  a  742.938.285        :: a is the multiplier
    =/  e  31                 :: e is the exponent
    =/  m  (sub (pow 2 e) 1)  :: modulus
    =/  index  0
    =/  accum  *@ub
    |-  ^-  @ub
    ?:  =(index n)  accum
    %=  $
      index  +(index)
      z      (mod (mul a z) m)
      accum  (cat 5 z accum)
    ==
  --
--
```

A more complicated formula uses several constants to improve the accuracy significantly:

<img src="https://latex.codecogs.com/svg.image?\large&space;Z&space;=&space;\text{sgn}\left(U-\frac{1}{2}\right)&space;\left(&space;t&space;-&space;\frac{c_{0}&plus;c_{1}&space;t&plus;c_{2}&space;t^{2}}{1&plus;d_{1}&space;t&plus;d_{2}&space;t^{2}&space;&plus;&space;d_{3}&space;t^{3}}&space;\right)" title="https://latex.codecogs.com/svg.image?\large Z = \text{sgn}\left(U-\frac{1}{2}\right) \left( t - \frac{c_{0}+c_{1} t+c_{2} t^{2}}{1+d_{1} t+d_{2} t^{2} + d_{3} t^{3}} \right)" />

where

- sgn is the signum or sign function;
- _t_ is √-ln[min(_U_, 1-_U_)²]; and
- constants are as below.

<!--
$$
Z = \text{sgn}\left(U-\frac{1}{2}\right) \left( t - \frac{c_{0}+c_{1} t+c_{2} t^{2}}{1+d_{1} t+d_{2} t^{2} + d_{3} t^{3}} \right)
$$
-->

**`/gen/normal2.hoon`**

```hoon
!:
=<
|=  n=@ud  :: n is the number of values to return
^-  (list @rs)
=/  values  (rip 5 (~(lcg gen 20.220.524) n))
=/  mask-clear           0b111.1111.1111.1111.1111.1111
=/  mask-fill   0b11.1111.0000.0000.0000.0000.0000.0000
=/  clears    (turn values |=(a=@rs (dis mask-clear a)))
=/  uniforms  (turn clears |=(a=@ (con mask-fill a)))
(turn uniforms normal)
|%
++  sgn
  |=  x=@rs
  ^-  @rs
  ?:  (lth:rs x .0)
    .-1
  ?:  (gth:rs x .0)
    .1
  .0
++  factorial
  :: integer factorial, not gamma function
  |=  x=@rs
  ^-  @rs
  =/  t=@rs  .1
  |-  ^-  @rs
  ?:  |(=(x .1) (lth x .1))  t
  $(x (sub:rs x .1), t (mul:rs t x))
++  absrs
  |=  x=@rs  ^-  @rs
  ?:  (gth:rs x .0)
    x
  (sub:rs .0 x)
++  exp
  |=  x=@rs
  ^-  @rs
  =/  rtol  .1e-5
  =/  p   .1
  =/  po  .-1
  =/  i   .1
  |-  ^-  @rs
  ?:  (lth:rs (absrs (sub:rs po p)) rtol)  p
  $(i (add:rs i .1), p (add:rs p (div:rs (pow-n x i) (factorial i))), po p)
++  pow-n
  ::  restricted power, based on integers only
  |=  [x=@rs n=@rs]
  ^-  @rs
  ?:  =(n .0)  .1
  =/  p  x
  |-  ^-  @rs
  ?:  (lth:rs n .2)  p
  $(n (sub:rs n .1), p (mul:rs p x))
++  ln
  ::  natural logarithm, z > 0
  |=  z=@rs
  ^-  @rs
  =/  rtol  .1e-5
  =/  p   .0
  =/  po  .-1
  =/  i   .0
  |-  ^-  @rs
  ?:  (lth:rs (absrs (sub:rs po p)) rtol)
    (mul:rs (div:rs (mul:rs .2 (sub:rs z .1)) (add:rs z .1)) p)
  =/  term1  (div:rs .1 (add:rs .1 (mul:rs .2 i)))
  =/  term2  (mul:rs (sub:rs z .1) (sub:rs z .1))
  =/  term3  (mul:rs (add:rs z .1) (add:rs z .1))
  =/  term  (mul:rs term1 (pow-n (div:rs term2 term3) i))
  $(i (add:rs i .1), p (add:rs p term), po p)
++  powrs
  ::  general power, based on logarithms
  ::  x^n = exp(n ln x)
  |=  [x=@rs n=@rs]
  (exp (mul:rs n (ln x)))
++  minrs
  |=  [a=@rs b=@rs]
  ?:  (lth:rs a b)  a  b
++  normal
  |=  u=@rs
  =/  c0  .2.515517
  =/  c1  .0.802853
  =/  c2  .0.010328
  =/  d1  .1.532788
  =/  d2  .0.189268
  =/  d3  .0.001308
  =/  t  (sqt:rs (powrs (sub:rs .1 (ln (minrs u (sub:rs .1 u)))) .2))
  =/  znum  :(add:rs c0 (mul:rs c1 t) (mul:rs c2 (mul:rs t t)))
  =/  zden  :(add:rs .1 (mul:rs d1 t) (mul:rs d2 (mul:rs t t)) (mul:rs d3 (powrs t .3)))
  (mul:rs (sgn (sub:rs u .0.5)) (sub:rs t (div:rs znum zden)))
++  gen
  |_  [z=@ud]
  ++  lcg
    |=  n=@ud                 :: n is the number of bits to return
    =/  a  742.938.285        :: a is the multiplier
    =/  e  31                 :: e is the exponent
    =/  m  (sub (pow 2 e) 1)  :: modulus
    =/  index  0
    =/  accum  *@ub
    |-  ^-  @ub
    ?:  =(index n)  accum
    %=  $
      index  +(index)
      z      (mod (mul a z) m)
      accum  (cat 5 z accum)
    ==
  --
--
```

How would you implement other random number generators?

### Hashing


### Hoon Operations
