---
title: Gates
nodes: 185
objectives:
  - "Explain what an Urbit ship is."
  - "Distinguish a fakeship from a liveship."
  - "Pronounce ASCII characters per standard Hoon developer practice."
---


-   [4b: text processing](https://urbit.org/docs/hoon/reference/stdlib#4b-text-processing)
  -   [`++cass`](https://urbit.org/docs/hoon/reference/stdlib/4b/#cass "To lowercase") [`++crip`](https://urbit.org/docs/hoon/reference/stdlib/4b/#crip "Tape to cord") [`++cuss`](https://urbit.org/docs/hoon/reference/stdlib/4b/#cuss "To uppercase") [`++mesc`](https://urbit.org/docs/hoon/reference/stdlib/4b/#mesc "Escape special characters") [`++runt`](https://urbit.org/docs/hoon/reference/stdlib/4b/#runt "Prepend n times") [`++sand`](https://urbit.org/docs/hoon/reference/stdlib/4b/#sand "Softcast by aura") [`++sane`](https://urbit.org/docs/hoon/reference/stdlib/4b/#sane "Check aura validity") [`++teff`](https://urbit.org/docs/hoon/reference/stdlib/4b/#teff "UTF8 length") [`++trim`](https://urbit.org/docs/hoon/reference/stdlib/4b/#trim "Tape split") [`++trip`](https://urbit.org/docs/hoon/reference/stdlib/4b/#trip "Cord to tape") [`++tuba`](https://urbit.org/docs/hoon/reference/stdlib/4b/#tuba "UTF8 to UTF32 tape") [`++tufa`](https://urbit.org/docs/hoon/reference/stdlib/4b/#tufa "UTF32 to UTF8 tape") [`++tuft`](https://urbit.org/docs/hoon/reference/stdlib/4b/#tuft "UTF32 to UTF8 text") [`++taft`](https://urbit.org/docs/hoon/reference/stdlib/4b/#taft "UTF8 to UTF32 cord") [`++wack`](https://urbit.org/docs/hoon/reference/stdlib/4b/#wack "Knot escape") [`++wick`](https://urbit.org/docs/hoon/reference/stdlib/4b/#wick "Knot unescape") [`++woad`](https://urbit.org/docs/hoon/reference/stdlib/4b/#woad "Unescape cord") [`++wood`](https://urbit.org/docs/hoon/reference/stdlib/4b/#wood "Escape cord")
-   [4m: parsing (formatting functions)](https://urbit.org/docs/hoon/reference/stdlib#4m-parsing-formatting-functions)
  -   [`++scot`](https://urbit.org/docs/hoon/reference/stdlib/4m/#scot "Render dime as cord") [`++scow`](https://urbit.org/docs/hoon/reference/stdlib/4m/#scow "Render dime as tape") [`++slat`](https://urbit.org/docs/hoon/reference/stdlib/4m/#slat "Curried slaw") [`++slav`](https://urbit.org/docs/hoon/reference/stdlib/4m/#slav "Demand: parse cord with input aura") [`++slaw`](https://urbit.org/docs/hoon/reference/stdlib/4m/#slaw "Parse cord to input aura") [`++slay`](https://urbit.org/docs/hoon/reference/stdlib/4m/#slay "Parse cord to coin") [`++smyt`](https://urbit.org/docs/hoon/reference/stdlib/4m/#smyt "Render path as tank") [`++spat`](https://urbit.org/docs/hoon/reference/stdlib/4m/#spat "Render path as cord") [`++spud`](https://urbit.org/docs/hoon/reference/stdlib/4m/#spud "Render path as tape") [`++stab`](https://urbit.org/docs/hoon/reference/stdlib/4m/#stab "Parse cord to path") [`++stap`](https://urbit.org/docs/hoon/reference/stdlib/4m/#stap "Path parser")

Let's examine some specific implementations:

How do we convert text into all upper-case?
- [`++cass`](https://urbit.org/docs/hoon/reference/stdlib/4b#cass)

How do we turn a `cord` into a `tape`?
- [`++trip`](https://urbit.org/docs/hoon/reference/stdlib/4b#trip)

How can we make a list of a null-terminated tuple?
- [`++le:nl`](https://urbit.org/docs/hoon/reference/stdlib/2m#lenl)

How can we evaluate Nock expressions?
- [`++mink`](https://urbit.org/docs/hoon/reference/stdlib/4n#mink)

(If you see a `|*` bartar rune in there, it's similar to a `|=` bartis, but what's called a _wet gate_.)

### `zuse.hoon`

When you open `zuse.hoon`, you'll see that it is composed with some data structures from `%lull`, but that by and large it consists of a core including arms organized into “engines”.

Most of these are internal Arvo conventions, such as conversion between Unix-epoch times and Urbit-epoch times.  The main one you are likely to work with is the `++html` core, which contains important tools for working with web-based data, such as [MIME types](https://en.wikipedia.org/wiki/Media_type) and [JSON strings](https://en.wikipedia.org/wiki/JSON).

To convert a `@ux` hexadecimal value to a `cord`:

```hoon
> (en:base16:mimes:html [3 0x12.3456])  
'123456'
```

To convert a `cord` to a `@ux` hexadecimal value:

```hoon
> `@ux`q.+>:(de:base16:mimes:html '123456')
0x12.3456
```

There are tools for working with Bitcoin wallet base-58 values, JSON strings, XML strings, and more.

```hoon
> (en-urlt:html "https://hello.me")
"https%3A%2F%2Fhello.me"
```

What seems to be missing from the standard library?


##  Formatted Text

A `+$tank` is a formatted print tree.  Error messages and the like are built of `tank`s.  They are defined in `hoon.hoon`:

```hoon
::  $tank: formatted print tree
::
::    just a cord, or
::    %leaf: just a tape
::    %palm: backstep list
::           flat-mid, open, flat-open, flat-close
::    %rose: flat list
::           flat-mid, open, close
::
+$  tank
  $~  leaf/~
  $@  cord
  $%  [%leaf p=tape]
      [%palm p=(qual tape tape tape tape) q=(list tank)]
      [%rose p=(trel tape tape tape) q=(list tank)]
  ==
+$ tang (list tank) :: bottom-first error
```

The [`++ram:re`](https://urbit.org/docs/hoon/reference/stdlib/4c#ramre) arm is used to convert these to actual formatted output, e.g.

```hoon
> ~(ram re leaf+"foo")
"foo"
> ~(ram re [%palm ["|" "(" "!" ")"] leaf+"foo" leaf+"bar" leaf+"baz" ~])
"(!foo|bar|baz)"
> ~(ram re [%rose [" " "[" "]"] leaf+"foo" leaf+"bar" leaf+"baz" ~])
"[foo bar baz]"
```

Many generators build sophisticated output using `tank`s and the short-format builder `+`, e.g. in `/gen/azimuth-block/hoon`:

```hoon
[leaf+(scow %ud block)]~
```

At this point we aren't going to use `tank`s directly yet, but you'll see them more and more as you move into more advanced generators.

### Deep Dive:  `cat.hoon`

For instance, how does `+cat` work?  Let's look at the structure of `/gen/cat/hoon`:

```hoon
::  ConCATenate file listings
::
::::  /hoon/cat/gen
  ::
/?    310
/+    pretty-file, show-dir
::
::::
  ::
:-  %say
|=  [^ [arg=(list path)] vane=?(%g %c)]
=-  tang+(flop `tang`(zing -))
%+  turn  arg
|=  pax=path
^-  tang
=+  ark=.^(arch (cat 3 vane %y) pax)
?^  fil.ark
  ?:  =(%sched -:(flop pax))
    [>.^((map @da cord) (cat 3 vane %x) pax)<]~
  [leaf+(spud pax) (pretty-file .^(noun (cat 3 vane %x) pax))]
?-     dir.ark                                          ::  handle ambiguity
    ~
  [rose+[" " `~]^~[leaf+"~" (smyt pax)]]~
::
    [[@t ~] ~ ~]
  $(pax (welp pax /[p.n.dir.ark]))
::
    *
  =-  [palm+[": " ``~]^-]~
  :~  rose+[" " `~]^~[leaf+"*" (smyt pax)]
      `tank`(show-dir vane pax dir.ark)
  ==
==
```

- What is the top-level structure of the generator?  (A cell of `%say` and the gate, previewing `%say` generators.)
- What don't you recognize?
  - `/?` faswut pins the expected Arvo kelvin version; right now it doesn't do anything
  - [`.^` dotket](https://urbit.org/docs/hoon/reference/rune/dot#-dotket) loads a value from Arvo (called a “scry”)
  - `++smyt` pretty-prints a path
  - [`=-` tishep](https://urbit.org/docs/hoon/reference/rune/tis#--tishep) combines with the subject, inverted relative to `=+`/`=/`.
  - There are some `tape` interpolation and `list` construction tools we haven't used yet:
      
      ```hoon
      > [<56>]
      "56"
      > [<56>]~
      ["56" ~]
      > ~[<56>]
      ["56" ~]
      > /[4]  
      [4 ~]
      ```
  - `?-` wuthep we saw up above:  it's a `switch` statement.
