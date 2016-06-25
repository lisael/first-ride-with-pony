# First ride with pony

The missing step-by-step pony tutorial.

In this tutorial we will build a Raft cluster from the ground up. In each step
we'll introduce new concepts, techniques, pony idioms, caveats, tips and tricks.

The code of pony-raft project is [hosted on github](https://github.com/lisael/pony-raft)

> Note: I'm a pony noob, not a guru. The code, tips and tricks in this tutorial
> come from my readings, my experience in coding and my short experience
> in pony. Neither the code or the tutorial have been reviewed yet. THIS TEXT
> COMES AS-IS, WITHOUT ANY GUARANTEE, AT ALL.

## What is Pony, anyway?

Well... I guess, if you ask this question, that you have to shut down your computer
and take a walk. You already clicked too much, too far. How the hell did you find this
book??

Anyway, I don't judge, and I gladly welcome any reader, but I think that reading
this tutorial requires at least a partial understanding of [the official
tutorial](http://tutorial.ponylang.org/). If you didn't yet, read it and come back
here. Don't be afraid if you don't catch all the subtleties, especially on
capacities, Pony needs practice, and practicing is precisely what we're about to
do here.

## Why this book?

I'm new to Pony (who isn't?). At the moment, learning Pony requires a lot of
personal investment. Large parts of Pony are simply not documented. One has to
read a lot of code, even the compiler itself, and the few available examples to
figure out how some Pony constructs work (`delegate`, generics...). I'm not
sure I can fill the blanks in [the official
tutorial](http://tutorial.ponylang.org/) already. Building a mid-sized project and
explaining every line of code are two great ways to learn a language.

## Why Pony?

Short answer: because it's fun and fresh.

## What is Raft?

Citing [Wikipedia](https://en.wikipedia.org/wiki/Raft_%28computer_science%29):

> "Raft is a consensus algorithm designed as an alternative to Paxos. It was
> meant to be more understandable than Paxos by means of separation of logic, but
> it is also formally proven safe and offers some new features. Raft offers a
> generic way to distribute a state machine across a cluster of computing
> systems, ensuring that each node in the cluster agrees upon the same series of
> state transitions."

The main resource to Raft is the [reference
paper](https://ramcloud.stanford.edu/wiki/download/attachments/11370504/raft.pdf).
Throughout this tutorial we will base our implementation on this paper, and
hopefully achieve a full-spec implementation.

A lot of resources are available at https://raft.github.io/ and the reader
doesn't have to read the long reference paper to start the tutorial. This [10
minutes tour](http://thesecretlivesofdata.com/raft/) of Raft is enough to
understand where the tutorial goes.

## Why Raft?

Raft is, to me, the perfect project for this kind of exercise, especially for
Pony.

* It's large enough
* It's not too large
* It has a well defined and bounded scope
* It relies on asynchronous calls, where pony excels
* It relies on network
* I know the algorithm
* It's fun and useful (think etcd. It relies on Raft)

## Who are you?

Just a Free Software enthusiast, seeking The Perfect Programming Language.

During this quest I started dreaming about an new keyword in Python: `actor`
that defines an asynchronous class that run in a separate thread with their own
GIL. I still had to find a smart way to check if a variable can be sent to an
actor or GC'd. I read about capabilities, and eventually I found Pony which
does exactly what I imagined and many more (compile time type checking, pattern
matching, generics...)

I mostly work with Python and C and played a bit with hipsters stuff like Go,
Rust and Erlang/Elixir. I also know a bit of old Java 1.5.

## Can I help?

Yes! Fork the repo and send a metric ton of pull request goodness. You may also
fill an issue on [github](https://github.com/lisael/first-ride-with-pony) As I
hope you did not guess yet, I'm not a native English speaker. I won't get upset
at all if the PR/issue is about grammar, spelling, syntax or vocabulary.
