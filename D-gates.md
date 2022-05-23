---
title: Gates
nodes: 100, 103
objectives:
  - "Explain what an Urbit ship is."
  - "Distinguish a fakeship from a liveship."
  - "Pronounce ASCII characters per standard Hoon developer practice."
---


  - "Annotate Hoon code with comments."
  - "Produce a generator to convert a value between auras."

  After completing Hoon School you should understand what the subject is and how to refer to its various parts.  You'll also learn about _cores_, which are the most important data structure in Hoon.  Once you get the hang of cores you'll be able to write your own functions in Hoon.

Chapter 2 covers the type system, writing basic apps, and the workings of the Arvo kernel.
Hoon source files are composed almost entirely of the printable ASCII characters. Hoon does not accept any other characters in source files except for UTF-8 in quoted strings. Hard tab characters are illegal; use two spaces instead.

> "You can put ½ in quotes, but not elsewhere!"
"You can put ½ in quotes, but not elsewhere!"

> 'You can put ½ in single quotes, too.'
'You can put ½ in single quotes, too.'

> "Some UTF-8: ἄλφα"
"Some UTF-8: ἄλφα"


##  Coordinating Files

In pragmatic terms, an Urbit ship is what results when you successfully boot a new ship.  If you are in the host OS, what you see is an apparently-empty folder:

```sh
$ ls zod
$
```

(For this lesson in particular take pains to distinguish the host OS prompt `$ ` from the Urbit Dojo prompt `> `.)

Contrast this with what the `+ls %` command shows you from inside of your Urbit:

```hoon
> +ls %
app/ desk/bill gen/ lib/ mar/ sur/ sys/ ted/
```

Urbit organizes its internal view of data and files as _desks_, which are associated collections of code and data.  These are not visible to the host operating system unless you explicitly mount them, and changes on one side are not made clear to the other until you “commit” them.  (Think of Dropbox, except that you have to explicitly synchronize to see changes somewhere else.)

Inside of your ship (“Mars”), you can mount a particular desk to the host operating system (“Earth”):

```hoon
> |mount %base
```

Now check what happens outside of your ship:

```sh
$ ls zod
base/
$ ls zod/base
app/  desk.bill gen/ lib/ mar/ sur/ sys/ ted/
```

If we make a change in the folder on Earth, the contents will only update on Mars if we explicitly tell the two systems to coordinate.

On Earth:

```sh
$ cp zod/base/desk.bill zod/base/desk.txt
```

On Mars:

```hoon
> |commit %base
+ /~zod/base/2/desk/txt
```

You can verify the contents of the copied files are the same using the `+cat` command:

```hoon
> +cat %/desk/bill
> +cat %/desk/txt
```

(Dojo does know what a `bill` file is, so it displays the contents slightly formatted.)

From Lesson 2 onwards, we will use this mode to store persistent code as files, editing on Earth and then synchronizing to Mars.

### What Makes a Folder into a Ship?

There appears to be nothing special about the `zod/` folder that `urbit` created.  In fact, there is a hidden folder which contains the system log and state.  (Show a hidden folder or file using `ls -a` at the Earth-side command line.)  You can investigate it (but don't change anything!):

```sh
$ cd zod
$ ls -a
base  .http-ports  .urb/  .vere.lock
$ ls .urb
bhk/ chk/ get/ log/ put/
$ ls .urb/chk/
north.bin  south.bin
```

These files contain all of the information necessary for the Urbit runtime program `urbit` to maintain and operate your ship's log, process new events, and persist the system state.


##  Irregular Syntax

Irregular Forms
Some runes are used so frequently that they have irregular counterparts that are easier to write and which mean precisely the same thing. Irregular rune syntax is hence a form of syntactic sugar. All irregular rune syntax is wide. (It follows from this that all tall form expressions are regular.)

Let’s look at another example. The .= rune takes two subexpressions, evaluates them, and tests the results for equality. If they’re equal it produces %.y for “yes”; otherwise %.n for “no”. In tall form:

> .=  22  11
%.n

> .=  22
  (add 11 11)
%.y
And in wide form:

> .=(22 11)
%.n

> .=(22 (add 11 11))
%.y
The irregular form of the .= rune is just =( ):

> =(22 11)
%.n

> =(22 (add 11 11))
%.y
The examples above contain another irregular form: (add 11 11). This is the irregular form of %+, which calls a gate (i.e., a Hoon function) with two arguments for the sample.

> %+  add  11  11
22

> %+(add 11 11)
22
The irregular ( ) gate-calling syntax is versatile -- it is also a shortcut for calling a gate with one argument, which is what the %- rune is for:

> (dec 11)
10

> %-  dec  11
10

> %-(dec 11)
10

The `( )` gate-calling syntax can be used for gates with any number of arguments.

You can find other irregular forms in the irregular expression reference document.



Hoon source files are composed almost entirely of the printable ASCII characters. Hoon does not accept any other characters in source files except for UTF-8 in quoted strings. Hard tab characters are illegal; use two spaces instead.
