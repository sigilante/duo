---
title: Gates
nodes: 184
objectives:
  - "Explain what an Urbit ship is."
  - "Distinguish a fakeship from a liveship."
  - "Pronounce ASCII characters per standard Hoon developer practice."
---

#   Conditional Logic

Although you've been using various of the `?` wut runes for a while now, let's wrap up some loose ends.

##  Making Choices

You are familiar in everyday life with making choices on the basis of a decision expression.  For instance, you can compare two prices for similar products and select the cheaper one for purchase.

Essentially, we have to be able to decide whether or not some value or expression evaluates as `%.y` true (in which case we will do one thing) or `%.n` false (in which case we do another).  Some basic expressions are mathematical, but we also check for existence, for equality of two values, etc.

- [`++gth`](https://urbit.org/docs/hoon/reference/stdlib/1a#gth) (greater than `>`)                   
- [`++lth`](https://urbit.org/docs/hoon/reference/stdlib/1a#lth) (less than `<`)  
- [`++gte`](https://urbit.org/docs/hoon/reference/stdlib/1a#gte) (greater than or equal to `≥`)
- [`++lte`](https://urbit.org/docs/hoon/reference/stdlib/1a#lte) (less than or equal to `≤`)
- [`.=` dottis](https://urbit.org/docs/hoon/reference/rune/dot#-dottis), irregularly `=()` (check for equality)

The key conditional decision-making rune is [`?:` wutcol](https://urbit.org/docs/hoon/reference/rune/wut#-wutcol), which lets you branch between an `expression-if-true` and an `expression-if-false`.  [`?.` wutdot](https://urbit.org/docs/hoon/reference/rune/wut#-wutdot) inverts the order of `?:`.  Good Hoon style prescribes that the heavier branch of a logical expression should be lower in the file.

There are also two long-form decision-making runes, which we will call [_switch statements_](https://en.wikipedia.org/wiki/Switch_statement) by analogy with languages like C.

- [`?-` wuthep](https://urbit.org/docs/hoon/reference/rune/wut#-wuthep) lets you choose between several possibilities, as with a type union.  Every case must be handled and no case can be unreachable.

    Since `@tas` terms are constants first, and not `@tas` unless marked as such, `?-` wuthep switches over term unions can make it look like the expression is branching on the value.  It's actually branching on the _type_.  These are almost exclusively used with term type unions.

    ```hoon
    |=  p=?(%1 %2 %3)
    ?-  p
      %1  1
      %2  2
      %3  3
    ==
    ```

- [`?+` wutlus](https://urbit.org/docs/hoon/reference/rune/wut#-wutlus) is similar to `?-` but allows a default value in case no branch is taken.  Otherwise these are similar to `?-` wuthep switch statements.

    ```hoon
    |=  p=?(%0 %1 %2 %3 %4)
    ?+  p  0
      %1  1
      %2  2
      %3  3
    ==
    ```

##  Loobean Logic

Throughout Hoon School, you have been using `%.y` and `%.n`, often implicitly, every time you have asked a question like `?:  =(5 4)`.  The `=()` expression returns a loobean, a member of the type union `?(%.y %.n)`.  (There is a proper aura `@f` but unfortunately it can't be used outside of the compiler.)  These can also be written as `&` (`%.y`, true) and `|` (`%.n`, false), which is common in older code but should be avoided for clarity in your own compositions.

What are the actual values of these, _sans_ formatting?

```hoon
> `@`%.y
0

> `@`%.n
1
```

Pretty much all conditional operators rely on loobeans, although it is very uncommon for you to need to unpack them.


##  Logical Operators

Mathematical logic allows the collocation of propositions to determine other propositions.  In computer science, we use this functionality to determine which part of an expression is evaluated.  We can combine logical statements pairwise:

- [`?&` wutpam](https://urbit.org/docs/hoon/reference/rune/wut#-wutpam), irregularly `&()`, is a logical `AND` _p_ ∧ _q_ over loobean values, e.g. both terms must be true.

    | `AND` | `%.y` | `%.n` |
    |-------|-------|-------|
    | `%.y` | `%.y` | `%.n` |
    | `%.n` | `%.n` | `%.n` |

    ```hoon
    > =/  a  5
      &((gth a 4) (lth a 7))
    %.y
    ```

- [`?|` wutbar](https://urbit.org/docs/hoon/reference/rune/wut#-wutbar), irregularly `|()`, is a logical `OR` _p_ ∨ _q_  over loobean values, e.g. either term may be true.

    | `OR`  | `%.y` | `%.n` |
    |-------|-------|-------|
    | `%.y` | `%.y` | `%.y` |
    | `%.n` | `%.y` | `%.n` |

    ```hoon
    > =/  a  5
      |((gth a 4) (lth a 7))
    %.y
    ```

- [`?!` wutzap](https://urbit.org/docs/hoon/reference/rune/wut#-wutzap), irregularly `!`, is a logical `NOT` ¬_p_.  Sometimes it can be difficult to parse code including `!` because it operates without parentheses.

    |       | `NOT` |
    |-------|-------|
    | `%.y` | `%.n` |
    | `%.n` | `%.y` |

    ```hoon
    > !%.y
    %.n

    > !%.n
    %.y
    ```

From these primitive operators, you can build other logical statements at need.

#### Exercise:  Design an `XOR` Function

The logical operation `XOR` _p_⊕_q_ exclusive disjunction yields true if one but not both operands are true.  `XOR` can be calculated by (_p_ ∧ ¬_q_) ∨ (¬_p_ ∧ _q_).

| `XOR` | `%.y` | `%.n` |
|-------|-------|-------|
| `%.y` | `%.n` | `%.y` |
| `%.n` | `%.y` | `%.n` |

- Implement `XOR` as a gate in Hoon.

    ```hoon
    |=  [p=?(%.y %.n) q=?(%.y %.n)]
    ^-  ?(%.y %.n)
    |(&(p !q) &(!p q))
    ```

#### Exercise:  Design a `NAND` Function

The logical operation `NAND` _p_ ↑ _q_ produces false if both operands are true.  `NAND` can be calculated by ¬(_p_ ∧ _q_).

| `NAND` | `%.y` | `%.n` |
|--------|-------|-------|
| `%.y`  | `%.n` | `%.y` |
| `%.n`  | `%.y` | `%.y` |

- Implement `NAND` as a gate in Hoon.

#### Exercise:  Design a `NOR` Function

The logical operation `NOR` _p_ ↓ _q_ produces true if both operands are false.  `NOR` can be calculated by ¬(_p_ ∨ _q_).

| `NOR` | `%.y` | `%.n` |
|-------|-------|-------|
| `%.y` | `%.n` | `%.n` |
| `%.n` | `%.n` | `%.y` |

- Implement `NAND` as a gate in Hoon.


##  Pattern Matching and Assertions

As values get passed around and checked at various points, the Hoon compiler tracks what the possible data structure or mold looks like.


One consequence of 

- [`?>` wutgar](https://urbit.org/docs/hoon/reference/rune/wut#-wutgar) is a positive assertion (`%.y%` or crash).

- [`?<` wutgal](https://urbit.org/docs/hoon/reference/rune/wut#-wutgal) is a negative assertion (`%.n` or crash).

- [`?~` wutsig](https://urbit.org/docs/hoon/reference/rune/wut#-wutket) asserts non-null.

- [`?^` wutket](https://urbit.org/docs/hoon/reference/rune/wut#-wutket) asserts cell.

- [`?@` wutpat](https://urbit.org/docs/hoon/reference/rune/wut#-wutpat) asserts atom.

Logical operators:

Pattern matching:

- [`?=` wuttis](https://urbit.org/docs/hoon/reference/rune/wut#-wuttis) tests for a pattern match in type, someday to be superseded or supplemented by a planned `?#` wuthax rune.


#### Exercise:  Implement the Boxcar Function
