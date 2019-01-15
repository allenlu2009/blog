---
layout: post
title: "37 percent of HyperRogue's compilation time is due to `std::function`"
date: 2019-01-06 00:01:00 +0000
tags:
  hyperrogue
  type-erasure
---

HyperRogue, [the most mind-expanding roguelike in
existence](https://web.archive.org/web/20171004073813/https://www.rockpapershotgun.com/2017/09/28/hyperrogue-non-euclidean-roguelike/),
is written in C++11 and uses a "unity build" model — every source file is `#include`d into
a single translation unit and compiled all in one go. That single compiler invocation takes about
142 seconds on my MacBook.

Yesterday [I replaced](https://github.com/zenorogue/hyperrogue/pull/67)
all of the `std::function`s in HyperRogue with a [hand-rolled minimalistic
`hyper_function`](https://github.com/Quuxplusone/hyperrogue/commit/4b3b2fd15c7cfc1be3202e290eff3b654d829fae#diff-2c224a5db38493856fb3f2083e7a9d68).
This new version compiles in about 90 seconds!

By replacing `std::function`, we eliminate 37 percent of the compile time;
23 percent of the linker symbols; 8.3 percent of the size of our object file;
and 7.5 percent of the size of our final executable.
(These specific numbers are of course unique to HyperRogue, which uses a *lot* of first-class functions
and thus benefits a *lot* from replacing `std::function`. Your benefits may vary.)

My somewhat self-serving conclusion: It's really important to keep teaching people how to
do type erasure! If you're comfortable whipping up a type-erased wrapper in 70 lines of code,
you can get some pretty nice improvements over the stuff that ships in the standard library.

----

Incidentally, today I learned that `std::function<void()>` does not complain about being
assigned a value-returning function:

    std::function<void()> f = []() { return false; };  // oops, void function shouldn't return anything

Replacing `std::function` with my hand-rolled version turned up a lot of nitpicky little bugs
like this in the HyperRogue code, which was another neat benefit of the change.