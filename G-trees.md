---
title: Gates
nodes: 135, 140, 156
objectives:
  - "Explain what an Urbit ship is."
  - "Distinguish a fakeship from a liveship."
  - "Pronounce ASCII characters per standard Hoon developer practice."
---

trees
lists




#### Example:  Number to Digits

- Compose a generator which accepts a number as `@ud` unsigned decimal and returns a list of its digits.

One verbose Hoon program 

```hoon
!:
|=  [n=@ud]
=/  values  *(list @ud)
|-  ^-  (list @ud)
?:  (lte n 0)  values
%=  $
  n       (div n 10)
  values  (weld ~[(mod n 10)] values)
==
```

Save this as a file `/gen/num2digit.hoon`, `|commit %base`, and run it:

```hoon
> +num2dig 1.000
~[1 0 0 0]

> +num2dig 123.456.789
~[1 2 3 4 5 6 7 8 9]
```

A more idiomatic solution would use the `^` ket infix to compose a cell and build the list from the head first.  (This saves a call to `++weld`.)

```hoon
!:
|=  [n=@ud]
=/  values  *(list @ud)
|-  ^-  (list @ud)
?:  (lte n 0)  values
%=  $
  n       (div n 10)
  values  (mod n 10)^values
==
```

A further tweak maps to `@t` ASCII characters instead of the digits.

```hoon
!:
|=  [n=@ud]
=/  values  *(list @t)
|-  ^-  (list @t)
?:  (lte n 0)  values
%=  $
  n       (div n 10)
  values  (@t (add 48 (mod n 10)))^values
==
```

(Notice that we apply `@t` as a mold gate rather than using the tic notation.  This is because `^` ket is a rare case where the order of evaluation of operators would cause the intuitive writing to fail.)

- Extend the above generator so that it accepts a cell of type and value (a `vase` as produced by the [`!>` zapgar](https://urbit.org/docs/hoon/reference/rune/zap#-zapgar) rune).  Use the type to determine which number base the digit string should be constructed from; e.g. `+num2dig !>(0xdead.beef)` should yield `~['d' 'e' 'a' 'd' 'b' 'e' 'e' 'f']`.
