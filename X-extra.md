### Other Functions in the Hoon Standard Library

Let's look once more at the parent core of the `add` arm in the Hoon standard library:

```hoon
> ..add
<46.hgz 1.pnw %140>
```

The battery of this core contains 46 arms, each of which evaluates to a gate in the standard library.  This “library” is nothing more than a core containing useful basic functions that Hoon often makes available as part of the subject.  You can see the Hoon code defining these arms near the beginning of [hoon.hoon](https://github.com/urbit/urbit/blob/master/pkg/arvo/sys/hoon.hoon), starting with [`++ add`](https://github.com/urbit/urbit/blob/master/pkg/arvo/sys/hoon.hoon#L21).  (The Hoon standard library is written entirely in Hoon.)

Here are some of the other gates that can be generated from this core in the Hoon standard library.  We hope it is fairly obvious what each one of these does, but check the docs if any of them are obscure.  <!-- TODO really? rework this -->

```hoon
> (dec 18)
17

> (dec 17)
16

> (gth 11 7)
%.y

> (gth 7 11)
%.n

> (gth 11 11)
%.n

> (lth 11 7)
%.n

> (lth 7 11)
%.y

> (lth 11 11)
%.n

> (max 12 14)
14

> (max 14 14)
14

> (max 14 432)
432

> (mod 11 7)
4

> (mod 22 7)
1

> (mod 33 7)
5

> (sub 234 123)
111
```

---

!:
=<
|=  n=@ud  :: n is the number of values to return
^-  (list @rs)
=/  values  (rip 5 (~(lcg gen 20.220.524) n))
=/  mask-clear           0b111.1111.1111.1111.1111.1111
=/  mask-fill   0b11.1111.0000.0000.0000.0000.0000.0000
=/  mask-sign           0b1000.0000.0000.0000.0000.0000
=/  clears    (turn values |=(a=@rs (dis mask-clear a)))
=/  uniforms  (turn clears |=(a=@ (con mask-fill a)))
=/  signs     (turn values |=(a=@rs (dis mask-sign a)))
=/  signs     (turn signs |=(a=@rs (lsh [0 8] a)))
=/  uniforms  (turn2 uniforms signs |=([a=@ b=@] (con a b)))
(turn uniforms normal)
|%
++  turn2  :: turn on a gate of two variables
  |=  [[a=(list @rs) b=(list @rs)] f=$-([@rs @rs] @rs)]
  ^-  (list @rs)
  ?+  +<-  ~|(%turn2-length !!)
    [~ ~]  ~
    [^ ^]  [(f i.a i.b) $(a t.a, b t.b)]
  ==
++  sgn
  |=  x=@rs
  ^-  @rs
  ?:  (lth:rs x .0)  .-1
  ?:  (gth:rs x .0)  .1
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


### Arbitrary Bases

One can utilize [any number as a radix or base](https://en.wikipedia.org/wiki/Radix) for a number system.  Commonly we employ base-10 (decimal), base-2 (binary), and base-16 (hexadecimal).  Other bases include base-8 (octal), base-12 (duodecimal), and base-60 (sexagesimal).  Only the common bases enjoy direct support in the core tooling for Hoon, so how can we implement a different base if we wanted to?

Let's demonstrate the [`++fo`](https://urbit.org/docs/hoon/reference/stdlib/3a#fo) core, which supports “modular arithmetic”, or 


 ++runt to prepend something n times but seemingly no library function to append n times?
New messages below
~master-morzod
10:42 AM
you can prepend in constant time, but appending is linear time, so you generally want to avoid it
we often build lists backwards and then +flop them when we're done
(but also (weld a (reap 3 b)) will append b to a three times)
