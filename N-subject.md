---
title: Gates
nodes: 165, 180
objectives:
  - "Explain what an Urbit ship is."
  - "Distinguish a fakeship from a liveship."
  - "Pronounce ASCII characters per standard Hoon developer practice."
---

https://urbit.org/docs/hoon/hoon-school/the-subject-and-its-legs

randomness in here
%say generators?

~master-morzod:
> eny and now are both just axes of the subject, no performance difference in accessing them. eny is 512 bits of entropy, sourced from a CSPRNG and then hash-iterated (sha-512) in arvo. it's shared between vanes during a event, but unique within each gall agent activation (also hash-iterated by gall)
now is a 128-bit timestamp, currently sourced from wall clock time (gettimeofday(), low resolution)
