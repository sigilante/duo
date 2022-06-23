---
title: Gates
nodes: 185, TODO
objectives:
  - "Identify tanks, tangs, wains, walls, and similar formatted printing data structures."
  - "Interpret logging message structures (`%leaf`, `$rose`, `$palm`)."
  - "Interpolate to tanks with `><` syntax."
  - "Produce useful error annotations using `~|` sigbar."
---

#   Text Processing II

_This module will elaborate on text representation in Hoon, including formatted text, `%ask` generators, and text parsing.  It may be considered optional and skipped if you are speedrunning Hoon School._

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

A `+$tank` is a formatted print tree.  Error messages and the like are built of `tank`s.  `tank`s are defined in `hoon.hoon`:

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

The [`++ram:re`](https://urbit.org/docs/hoon/reference/stdlib/4c#ramre) arm is used to convert these to actual formatted output as a `tape`, e.g.

```hoon
> ~(ram re leaf+"foo")
"foo"
> ~(ram re [%palm ["|" "(" "!" ")"] leaf+"foo" leaf+"bar" leaf+"baz" ~])
"(!foo|bar|baz)"
> ~(ram re [%rose [" " "[" "]"] leaf+"foo" leaf+"bar" leaf+"baz" ~])
"[foo bar baz]"
```

Many generators build sophisticated output using `tank`s and the short-format cell builder `+`, e.g. in `/gen/azimuth-block/hoon`:

```hoon
[leaf+(scow %ud block)]~
```

which is equivalent to

```hoon
~[[%leaf (scow %ud block)]]
```

`tank`s are the primary output mechanism for more advanced generators.  Even if you don't end up writing them much, you will encounter them as you delve into the Urbit codebase.

#### Tutorial:  Deep Dive into `ls.hoon`

The `+ls` generator shows the contents at a particular path in Clay:

```hoon
> +cat /===/gen/ls/hoon
/~nec/base/~2022.6.22..17.25.54..1034/gen/ls/hoon
::  LiSt directory subnodes
::
::::  /hoon/ls/gen
  ::
/?    310
/+    show-dir
::
::::
  ::
~&  %
:-  %say
|=  [^ [arg=path ~] vane=?(%g %c)]
=+  lon=.^(arch (cat 3 vane %y) arg)
tang+[?~(dir.lon leaf+"~" (show-dir vane arg dir.lon))]~
```

Let's go line by line:

```hoon
/?    310
/+    show-dir
```

The first line `/?` faswut represents now-future functionality which will allow the version number of the kernel to be pinned.  It is currently non-functioning but you will see it in many Urbit-shipped files.

Then the `show-dir` library is imported.

```hoon
~&  %
```

A separator `%` is printed.

```hoon
:-  %say
```

A `%say` generator is a cell with a metadata tag `%say` as the head and the gate as the tail.

```hoon
|=  [^ [arg=path ~] vane=?(%g %c)]
```

This generator requires a path argument in its sample and optionally accepts a vane tag (`%g` Gall or `%c` Clay).  Most of the time, `+cat` is used with Gall, so `%g` as the last entry in the type union serves as the bunt value.

```hoon
=+  lon=.^(arch (cat 3 vane %y) arg)
```

We saw [`.^` dotket](https://urbit.org/docs/hoon/reference/rune/dot#-dotket) for the first time in [the previous module](./N-subject.md), where we learned that it performs a _peek_ or _scry_ into the state of an Arvo vane.  Most of the time this functionality is used to ask `%c` Clay or `%g` Gall for information about a path, desk, agent, etc.  In this case, `(cat 3 %c %y)` is a fancy way of collocating the two `@tas` terms into `%cy`, a Clay file or directory lookup.  The type of this lookup is `+$arch`, and the location of the file or directory is given by `arg` from the sample.

```hoon
tang+[?~(dir.lon leaf+"~" (show-dir vane arg dir.lon))]~
```

The result of the lookup on the previous line is adapted into a formatted text block with a head of `%tang` and different results depending on whether the request was `~` null or not.

#### Tutorial:  Deep Dive into `cat.hoon`

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

- What is the top-level structure of the generator?  (A cell of `%say` and the gate, previewing `%say` generators. TODO)
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


##  `%ask` Generators

Previously, we introduced the concept of a `%say` generator to produce a more versatile form of standalone single computation than a simple naked generator (gate) allowed.  Another elaboration, the `%ask` generator, takes things further.

We use an `%ask` generator when we want to create an interactive program that prompts for inputs as it runs, rather than expecting arguments to be passed in at the time of initiation.



- [Hoon Guide, “CLI apps”](https://urbit.org/docs/hoon/guides/cli-tutorial)
- [`~wicdev-wisryt`, “Input and Output in Hoon”](https://urbit.org/blog/io-in-hoon)

##### `%ask` example
TODO
The code below is an `%ask` generator that checks if the user inputs "blue" when prompted. Save it as `axe.hoon` in the `/gen` directory of your `%base` desk.

```hoon
/-  sole
/+  generators
=,  [sole generators]
:-  %ask
|=  *
^-  (sole-result (cask tang))
%+  print    leaf+"What is your favorite color?"
%+  prompt   [%& %prompt "color: "]
|=  t=tape
%+  produce  %tang
?:  =(t "blue")
  :~  leaf+"Oh. Thank you very much."
      leaf+"Right. Off you go then."
  ==
:~  leaf+"Aaaaagh!"
    leaf+"Into the Gorge of Eternal Peril with you!"
==
```

Run the generator from the Dojo:
> +axe

What is your favorite color?

: color:

Something new happened. Instead of simply returning something, your Dojo's prompt changed from `~your-urbit:dojo>` to `~your-urbit:dojo: color:`, and now expects additional input. Let's give it an input:

: color: red

Into the Gorge of Eternal Peril with you!

Aaaaagh!

Let's go over what exactly is happening in this code.

/-  sole

/+  generators

=,  [sole generators]

Here we bring in some of the types we are going to need from `/sur/sole` and gates we will use from `/lib/generators`. We use some special runes for this.

`/-` is a Ford rune used to import types from `sur`.

`/+` is a Ford rune used to import libraries from `lib`.

`=,` is a rune that allows us to expose a namespace. We do this to avoid having to write `sole-result:sole` instead of `sole-result` or `print:generators` instead of `print`.

:-  %ask

|=  *

This code might be familiar. Just as with their `%say` cousins, `%ask` generators need to produce a `cell`, the head of which specifies what kind of generator we are running.

With `|= *`, we create a gate and ignore the standard arguments we are given, because we're not using them.

^-  (sole-result (cask tang))

`%ask` generators need to have the second half of the cell be a gate that produces a `sole-result`, one that in this case contains a `cask` of `tang`. We use the `^-` rune to constrain the generator's output to such a `sole-result`.

A `cask` is a pair of a `mark` name and a noun. Recall that a `mark` can be thought of as an Arvo-level MIME type for data.

A `tang` is a `list` of `tank`, and a `tank` is a structure for printing data. There are three types of `tank`: `leaf`, `palm`, and `rose`. A `leaf` is for printing a single noun, a `rose` is for printing rows of data, and a `palm` is for printing backstep-indented lists.

%+  print    leaf+"What is your favorite color?"

%+  prompt   [%& %prompt "color: "]

|=  t=tape

%+  produce  %tang

Because we imported `generators`, we can access its contained gates, three of which we use in `axe.hoon`: `print`, `prompt`, and `produce`.

**`print`** is used for printing a `tank` to the console.

In our example, `%+` is the rune to call a gate, and our gate `print` takes one argument which is a `tank` to print. The `+` here is syntactic sugar for `[%leaf "What is your favorite color?"]` that just makes it easier to write.

**`prompt`** is used to construct a prompt for the user to provide input. It takes a single argument that is a tuple. Most `%ask` generators will want to use the `prompt` gate.

The first element of the `prompt` sample is a flag that indicates whether what the user typed should be echoed out to them or hidden. `%&` will produce echoed output and `%|` will hide the output (for use in passwords or other secret text).

The second element of the `prompt` sample is intended to be information for use in creating autocomplete options for the prompt. This functionality is not yet implemented.

The third element of the `prompt` sample is the `tape` that we would like to use to prompt the user. In the case of our example, we use `"color: "`.

**`produce`** is used to construct the output of the generator. In our example, we produce a `tang`.

|=  t=tape

Our gate here takes a `tape` that was produced by `prompt`. If we needed another type of data we could use `parse` to obtain it.

The rest of this generator should be intelligible to those with Hoon knowledge at this point.

One quirk that you should be aware of, though, is that `tang` prints in reverse order from how it is created. The reason for this is that `tang` was originally created to display stack trace information, which should be produced in reverse order. This leads to an annoyance: we either have to specify our messages backwards or construct them in the order we want and then `flop` the `list`.

### Example:  Parsing `tape`s (`;~` micsig)

We need to build a tool to accept a `tape` containing some characters, then turn it into—something else, something computational.

For instance, a calculator could accept an input like `3+4` and return `7`.  A command-line interface may look for a program to evaluate (like Bash and `ls`).  A search bar may apply logic to the query (like Google and `-` for `NOT`).

The basic problem is this:

1. You need to accept a `tape` containing some characters.
2. You need to ingest one or more characters and decide what they “mean”, including storing the result of this meaning.
3. You need to loop back to #1 again and again until you are out of characters.

We could build a simple parser out of a trap and `++snag`, but it would be brittle and difficult to extend.  The Hoon parser is very sophisticated, since it has to take a file of ASCII characters (and some UTF-8 strings) and turn it via an AST into Nock code.  What makes parsing challenging is that we have to wade directly into a sea of new types and processes.  To wit:

-   A `tape` is the string to be parsed.
-   A `hair` is the position in the text the parser is at, as a cell of column & line, `[p=@ud q=@ud]`.
-   A `nail` is parser input, a cell of `hair` and `tape`.
-   An `edge` is parser output, a cell of `hair` and a `unit` of `hair` and `nail`.  (There are some subtleties around failure-to-parse here that we'll defer a moment.)
-   A `rule` is a parser, a gate which applies a `nail` to yield an `edge`.

Basically, one uses a `rule` on `[hair tape]` to yield an `edge`.

A substantial swath of the standard library is built around parsing for various scenarios, and there's a lot to know to effectively use these tools.  **If you can parse arbitrary input using Hoon after this lesson, you're in fantastic shape for building things later.**  It's worth spending extra effort to understand how these programs work.

- [Hoon Guide, “Parsing”](https://urbit.org/docs/hoon/guides/parsing)

### Scanning Through a `tape`

[`++scan`](https://urbit.org/docs/hoon/reference/stdlib/4g#scan) parses a `tape` or crashes, simple enough.  It will be our workhorse.  All we really need to know in order to use it is how to build a `rule`.

Here we will preview using `++shim` to match characters with in a given range, here lower-case.  If you change the character range, e.g. putting `' '` in the `++shim` will span from ASCII `32`, `' '` to ASCII `122`, `'z'`.

```hoon
> `(list)`(scan "after" (star (shim 'a' 'z')))  
~[97 102 116 101 114]  

> `(list)`(scan "after the" (star (shim 'a' 'z')))
{1 6}  
syntax error  
dojo: hoon expression failed
```

### `rule` Building

The `rule`-building system is vast and often requires various components together to achieve the desired effect.

#### `rule`s to parse fixed strings

- [`++just`](https://urbit.org/docs/hoon/reference/stdlib/4f/#just) takes in a single `char` and produces a `rule` that attempts to match that `char` to the first character in the `tape` of the input `nail`.

    ```hoon
    > ((just 'a') [[1 1] "abc"])
    [p=[p=1 q=2] q=[~ [p='a' q=[p=[p=1 q=2] q="bc"]]]]
    ```

- [`++jest`](https://urbit.org/docs/hoon/reference/stdlib/4f/#jest) matches a `cord`.  It takes an input `cord` and produces a `rule` that attempts to match that `cord` against the beginning of the input.

    ```hoon
    > ((jest 'abc') [[1 1] "abc"])
    [p=[p=1 q=4] q=[~ [p='abc' q=[p=[p=1 q=4] q=""]]]]

    > ((jest 'abc') [[1 1] "abcabc"])
    [p=[p=1 q=4] q=[~ [p='abc' q=[p=[p=1 q=4] q="abc"]]]]
    
    > ((jest 'abc') [[1 1] "abcdef"])
    [p=[p=1 q=4] q=[~ [p='abc' q=[p=[p=1 q=4] q="def"]]]]
    ```

    (Keep an eye on the structure of the return `edge` there.)

- [`++shim`](https://urbit.org/docs/hoon/reference/stdlib/4f/#shim) parses characters within a given range. It takes in two atoms and returns a `rule`.

    ```hoon
    > ((shim 'a' 'z') [[1 1] "abc"])
    [p=[p=1 q=2] q=[~ [p='a' q=[p=[p=1 q=2] q="bc"]]]]
    ```

- [`++next`](https://urbit.org/docs/hoon/reference/stdlib/4f/#next) is a simple `rule` that takes in the next character and returns it as the parsing result.

    ```hoon
    > (next [[1 1] "abc"])
    [p=[p=1 q=2] q=[~ [p='a' q=[p=[p=1 q=2] q="bc"]]]]
    ```

#### `rule`s to parse flexible strings

So far we can only parse one character at a time, which isn't much better than just using `++snag` in a trap.

```hoon
> (scan "a" (shim 'a' 'z'))  
'a'  

> (scan "ab" (shim 'a' 'z'))  
{1 2}  
syntax error  
dojo: hoon expression failed
```

How do we parse multiple characters in order to break things up sensibly?

- [`++star`](https://urbit.org/docs/hoon/reference/stdlib/4f#star) will match a multi-character list of values.

    ```hoon
    > (scan "a" (just 'a'))
    'a'

    > (scan "aaaaa" (just 'a'))
    ! {1 2}
    ! 'syntax-error'
    ! exit

    > (scan "aaaaa" (star (just 'a')))
    "aaaaa"
    ```

- [`++plug`](https://urbit.org/docs/hoon/reference/stdlib/4e/#plug) takes the `nail` in the `edge` produced by one rule and passes it to the next `rule`, forming a cell of the results as it proceeds.

    ```hoon
    > (scan "starship" ;~(plug (jest 'star') (jest 'ship')))
    ['star' 'ship']
    ```

- [`++pose`](https://urbit.org/docs/hoon/reference/stdlib/4e/#pose) tries each `rule` you hand it successively until it finds one that works.

    ```hoon
    > (scan "a" ;~(pose (just 'a') (just 'b')))
    'a'
    
    > (scan "b" ;~(pose (just 'a') (just 'b')))
    'b'
    
    > (;~(pose (just 'a') (just 'b')) [1 1] "ab")
    [p=[p=1 q=2] q=[~ u=[p='a' q=[p=[p=1 q=2] q=[i='b' t=""]]]]]
    ```

- [`++glue`](https://urbit.org/docs/hoon/reference/stdlib/4e/#glue) parses a delimiter in between each `rule` and forms a cell of the results of each `rule`.  Delimiter names hew to the aural ASCII pronunciation of symbols, plus `prn` for printable characters and

    ```hoon
    > (scan "a b" ;~((glue ace) (just 'a') (just 'b')))  
    ['a' 'b']

    > (scan "a,b" ;~((glue com) (just 'a') (just 'b')))
    ['a' 'b']
    
    > (scan "a,b,a" ;~((glue com) (just 'a') (just 'b')))
    {1 4}
    syntax error
    
    > (scan "a,b,a" ;~((glue com) (just 'a') (just 'b') (just 'a')))
    ['a' 'b' 'a']
    ```

- The [`;~` micsig](https://urbit.org/docs/hoon/reference/rune/mic/#-micsig) will create `;~(combinator (list rule))` to use multiple `rule`s.

    ```hoon
    > (scan "after the" ;~((glue ace) (star (shim 'a' 'z')) (star (shim 'a' 'z'))))  
    [[i='a' t=<|f t e r|>] [i='t' t=<|h e|>]
    
    > (;~(pose (just 'a') (just 'b')) [1 1] "ab")  
    [p=[p=1 q=2] q=[~ u=[p='a' q=[p=[p=1 q=2] q=[i='b' t=""]]]]]
    ```

At this point we have two problems:  we are just getting raw `@t` atoms back, and we can't iteratively process arbitrarily long strings.  `++cook` will help us with the first of these:

- [`++cook`](https://urbit.org/docs/hoon/reference/stdlib/4f/#cook) will take a `rule` and a gate to apply to the successful parse.

    ```hoon
    > ((cook ,@ud (just 'a')) [[1 1] "abc"])
    [p=[p=1 q=2] q=[~ u=[p=97 q=[p=[p=1 q=2] q="bc"]]]]

    > ((cook ,@tas (just 'a')) [[1 1] "abc"])
    [p=[p=1 q=2] q=[~ u=[p=%a q=[p=[p=1 q=2] q="bc"]]]]

    > ((cook |=(a=@ +(a)) (just 'a')) [[1 1] "abc"])
    [p=[p=1 q=2] q=[~ u=[p=98 q=[p=[p=1 q=2] q="bc"]]]]

    > ((cook |=(a=@ `@t`+(a)) (just 'a')) [[1 1] "abc"])
    [p=[p=1 q=2] q=[~ u=[p='b' q=[p=[p=1 q=2] q="bc"]]]]
    ```

However, to parse iteratively, we need to use the [`++knee`]() function, which takes a noun as the bunt of the type the `rule` produces, and produces a `rule` that recurses properly.  (You'll probably want to treat this as a recipe for now and just copy it when necessary.)

```hoon
|-(;~(plug prn ;~(pose (knee *tape |.(^$)) (easy ~))))
```

There is an example of a calculator [in the docs](https://urbit.org/docs/hoon/guides/parsing#recursive-parsers) that's worth a read.  It uses `++knee` to scan in a set of numbers at a time.

```hoon
|=  math=tape
|^  (scan math expr)
++  factor
  %+  knee  *@ud
  |.  ~+
  ;~  pose
    dem
    (ifix [pal par] expr)
  ==
++  term
  %+  knee  *@ud
  |.  ~+
  ;~  pose
    ((slug mul) tar ;~(pose factor term))
    factor
  ==
++  expr
  %+  knee  *@ud
  |.  ~+
  ;~  pose
    ((slug add) lus ;~(pose term expr))
    term
  ==
--
```

Once you can parse input, you can start to build some fun CLI agents.  (We'd love to see [Zork](https://en.wikipedia.org/wiki/Zork) on Mars someday.)  If you are interested in producing command-line-based tools, look more into the `%shoe` and `%sole` libraries.

#### Example:  Parse a String of Numbers

A simple `++shim`-based parser:

```hoon
> (scan "1234567890" (star (shim '0' '9')))  
[i='1' t=<|2 3 4 5 6 7 8 9 0|>]
```

A refined `++cook`/`++cury`/`++jest` parser:

```hoon
> ((cook (cury slaw %ud) (jest '1')) [[1 1] "123"])  
[p=[p=1 q=2] q=[~ u=[p=[~ 1] q=[p=[p=1 q=2] q="23"]]]]  

> ((cook (cury slaw %ud) (jest '12')) [[1 1] "123"])
[p=[p=1 q=3] q=[~ u=[p=[~ 12] q=[p=[p=1 q=3] q="3"]]]]
```
