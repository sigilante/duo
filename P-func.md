---
title: Functional Programming
nodes: 233, 283, 383
objectives:
  - "Reel, roll, turn a list."  
  - "Curry, cork functions."  
  - "Change arity of a gate."
  - "Tokenize text simply using `find` and `trim`."
  - "Identify elements of parsing:  `nail`, `rule`, etc."
  - "Use `++scan` to parse `tape` into atoms."
  - "Construct new rules and parse arbitrary text fields."
---

#   Functional Programming

Given a gate, you can manipulate it to accept a different number of values than its sample formally requires, or otherwise modify its behavior.  These techniques mirror some of the common tasks used in other [functional programming languages](https://en.wikipedia.org/wiki/Functional_programming) like Haskell, Clojure, and OCaml.

Functional programming, as a paradigm, tends to prefer rather mathematical expressions with explicit modification of function behavior.  It works as a formal system of symbolic expressions manipulated according to given rules and properties.  FP was derived from the [lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus), a cousin of combinator calculi like Nock.  (See also [APL](https://en.wikipedia.org/wiki/APL_%28programming_language%29).)

##  Changing Arity

If a gate accepts only two values in its sample, for instance, you can chain together multiple calls automatically using the [`;:` miccol](https://urbit.org/docs/hoon/reference/rune/mic#-miccol) rune.

```hoon
> (add 3 (add 4 5))
12

> :(add 3 4 5)
12

> (mul 3 (mul 4 5))
60

> :(mul 3 4 5)
60
```

This is called changing the [_arity_](https://en.wikipedia.org/wiki/Arity) of the gate.  (Does this work on `++mul:rs`?)


##  Binding the Sample

[_Currying_](https://en.wikipedia.org/wiki/Currying) describes taking a function of multiple arguments and reducing it to a set of functions that each take only one argument.  _Binding_, an allied process, is used to set the value of some of those arguments permanently.

If you have a gate which accepts multiple values in the sample, you can fix one of these.  To fix the head of the sample (the first argument), use [`++cury`](https://urbit.org/docs/hoon/reference/stdlib/2n#cury); to bind the tail, use [`++curr`](https://urbit.org/docs/hoon/reference/stdlib/2n#curr).

Consider calculating _a x² + b x + c_, a situation we earlier resolved using a door.  We can resolve the situation differently using currying:

```hoon
> =full |=([x=@ud a=@ud b=@ud c=@ud] (add (add (mul (mul x x) a) (mul x b)) c))

> (full 5 4 3 2)
117

> =one (curr full [4 3 2])  

> (one 5)  
117
```

One can also [`++cork`](https://urbit.org/docs/hoon/reference/stdlib/2n#cork) a gate, or arrange it such that it applies to the result of the next gate.  This pairs well with `;:` miccol.  (There is also [`++corl`](https://urbit.org/docs/hoon/reference/stdlib/2n#corl), which composes backwards rather than forwards.)  This example converts a value to `@ux` then decrements it by corking two molds:

```hoon
> ((cork dec @ux) 20)  
0x13
```

#### Exercise:  Bind Gate Arguments

- Create a gate `++inc` which increments a value in one step, analogous to `++dec`.

#### Exercise:  Chain Gate Values

- Write an expression which yields the parent galaxy of a planet's sponsoring star by composing two gates.

##  Working Across `list`s

turn
The turn function takes a list and a gate, and returns a list of the products of applying each item of the input list to the gate. For example, to add 1 to each item in a list of atoms:

> (turn `(list @)`~[11 22 33 44] |=(a=@ +(a)))
~[12 23 34 45]
Or to double each item in a list of atoms:

> (turn `(list @)`~[11 22 33 44] |=(a=@ (mul 2 a)))
~[22 44 66 88]
turn is Hoon's version of Haskell's map.

We can rewrite the Caesar cipher program using turn:

|=  [a=@ b=tape]
^-  tape
?:  (gth a 25)
  $(a (sub a 26))
%+  turn  b
|=  c=@tD
?:  &((gte c 'A') (lte c 'Z'))
  =.  c  (add c a)
  ?.  (gth c 'Z')  c
  (sub c 26)
?:  &((gte c 'a') (lte c 'z'))
  =.  c  (add c a)
  ?.  (gth c 'z')  c
  (sub c 26)
c


[`++roll`](https://urbit.org/docs/hoon/reference/stdlib/2b#roll) and [`++reel`](https://urbit.org/docs/hoon/reference/stdlib/2b#reel) are used to left-fold and right-fold a list, respectively.  To fold a list is similar to [`++turn`](https://urbit.org/docs/hoon/reference/stdlib/2b#turn), except that instead of yielding a `list` with the values having had each applied, `++roll` and `++reel` produce an accumulated value.

```hoon
> (roll `(list @)`[1 2 3 4 5 ~] add)
q=15

> (reel `(list @)`[1 2 3 4 5 ~] mul)
120
```

#### Exercise:  

- Use `++reel` to produce a gate which calculates the factorial of a number.


##  Parsing Text

Text parsers
TODO


btw you should almost always avoid recursive welding cos weld has to traverse the entire first list in order to weld it
so you potentially end up traversing the list thousands of times
which involves chasing a gorillion pointers
as a rule of thumb you wanna avoid the recursive use of stdlib list functions in general
