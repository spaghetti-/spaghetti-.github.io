---
title: Rustcon Asia 2019 - Beijing
date: 2019-04-25 16:01:13
tags:
 - rust
 - conference
---

I recently attended [Rustcon Asia 2019](https://rustcon.asia) in Beijing, China
and these are my thoughts on the experience from the point of view of a

* newbie to Rust
* non Chinese speaker

the latter of which is not really important.

## TL;DR/Summary

The conference was a great opportunity to learn more about Rust in the
workplaces and the kind of workloads it was being put through. Custom derives
and higher order concepts that I am not familiar with were broken in (just on
the top).

Majority of the talks were in Chinese but it was very follow-able for a non
Chinese speaker because of the live translation. The live translation does of
course lose a lot of the tone and humor, but it got the job done. When the
speakers were using slides in Chinese, it was *very very* hard to follow along
even with the translation. At the very least, I'd like to have received an
English copy of the slides that I can refer to.

I appreciated the helpful and kind nature of everyone at the conference. It felt
welcoming. A small improvement that I'd like to suggest for future organizers is
to start maybe an hour later (10 AM local time) and to provide
coffee at the venue!

## Rust

I'm new to Rust but I come from a C++ and Python and (naively) Haskell
background which makes the language sufficiently interesting to follow. It
combines some of the best features that I like, is a statically typed, compiled
language that *I* can grok in my head. I am highlighting it because there are
easier to grok languages that I can't get my head around because it's different
from the *way I think*.

I also come from a Robotics background doing underwater and surface robotics,
which is an area in which C++ is king. Rust can (and should) be the default
choice for the safety it provides (critical when working around people and
expensive installations such as oil and gas pipelines) and performance.

Armed with my curiosity and interest to learn more applications and ideas from
fellow Rust developers, I left for Beijing.

## Conference

The conference had talks for two days which began at 8 AM (first talk being at
9 AM) which is kinda *really* early. Nick Cameron's talk on making rust
ergonomic gave an insight to what the core team thinks and was a great starter
talk. Some of the other talks that I found interesting were

### Implementing a secp256k1 library in pure Rust

Which gave an introduction to elliptic curves: in math, and then in Rust. Math
being an universal language gives the audience a way to relate quickly to what's
happening in the Rust code even if you're not familiar with the higher level
concepts in the language.

### How Rust taught me to think about systems

Entertaining - made you think back to when you were struggling with the borrow
checker (not that I'm not right now but to a lesser degree) and the language in
general. For the new-er folks, the speakers adventures give a solid start on how
to reference library files, check for types and trace lifetimes.

### Improving web app with Rust and WASM

An introduction to plugging in Rust compiled down to WASM for web developers and
a case study in which it was immediately useful to a client. The speaker used
wasm to program a real time renderer for dentists to showcase teeth
modifications based on parameters provided from the JS layer.

The talk could have been improved a bit by providing absolute reference times
when comparing performance across JS and WASM and by providing a debugging
methodology.

### Cargo meets Autotools

I'm interested in build systems and package managers and I'm opinionated about
them. I use portage as a daily driver and I'm familiar with CMake and autotools.
Having autotools is a godsend for deployments that don't need cargo installed
(not distributed by default by distributions amongst other issues) and so is
CMake. There are folks who are about to distribute libraries with deb and rpm.

Some specifics about LTO, and differences mentioned in the talk between the
final product with autotools vs CMake was not very clear to me though.

### Distributed Actor System in Rust

A very well done talk with a real world use case that I'm tackling as well. A
distributed actor system that's not Akka and is build in-house by Alibaba for
their use case. Preserving types when sending messages and ensuring consistency
when compiling and rolling out were some of the topics that was talked
about. The speaker was eloquent (even to a non Chinese speaker) and the talk was
understandable even to a newbie. Awesome!

### Be Fearless Using Rust in Production

A talk about using Rust to replace an in house image stitcher from NodeJS. The
slides gave a clear idea of the kind of workload that was implemented and it's
something that I myself will first prototype in ImageMagick and then dump on C++
(OpenCV3). Using Rust seemed to have paid off, with impressive QPS being handled
by the same set of servers.

The talk also spoke about how difficult it was to get the company to approve
using Rust (at all) but they came around after an engineer used his own time to
prototype the solution.

Addition stuff that I'd have liked from the talk

* Rust bindings for OpenCV
* Benchmarks with an implementation in C++

### Misc

I wasn't too impressed with the talk about _how to learn rust efficiently_
because it was more about learning than it was about learning Rust. I suppose
just a mismatch on expectations and style of learning.

## Workshops

I did not attend the workshops because I had to get an expensive visa to enter
China. The time was better spent exploring the wonderful city of Beijing and
its delicious food.
