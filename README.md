# First ride with pony

The missing step-by-step pony tutorial.

In this tutorial we will build a Raft cluster from the ground up. In each step
we'll introduce new concepts, techniques, pony idioms, caveats, tips and tricks.

## Why this book?

I'm new to Pony (who isn't?). At the moment, learning Pony requires a lot of
personal investment. Large parts of Pony are simply not documented. One has to
read a lot of code, even the compiler itself, and the few available examples to
figure out how some Pony constructs work (`delegate`, generics...). I'm not
sure I can fill the blanks in [the official
tutorial](http://tutorial.ponylang.org/) yet. Building a mid-sized project and
explaining every line are two great ways to learn a language.

## Why Pony?

Short answer: because it's fun and fresh.

## What is Raft?

Citing [Wikipedia](https://en.wikipedia.org/wiki/Raft_(computer_science)):
"Raft is a consensus algorithm designed as an alternative to Paxos. It was
meant to be more understandable than Paxos by means of separation of logic, but
it is also formally proven safe and offers some new features.[1] Raft offers a
generic way to distribute a state machine across a cluster of computing
systems, ensuring that each node in the cluster agrees upon the same series of
state transitions."

The main resource to Raft is the [reference
paper](https://ramcloud.stanford.edu/wiki/download/attachments/11370504/raft.pdf)
and throughout this tutorial we will base our implementation on this paper, and
hopefully achieve a full-spec implementation.

A lot of resources are available at https://raft.github.io/ and the reader
doesn't have to read the long reference paper to start the tutorial. This [10
minutes tour](http://thesecretlivesofdata.com/raft/) greatly enough to
understand where the tutorial goes.

## Why Raft?

Raft is, to me, the perfect project for this kind of exercise, especially for
Pony.

* It's large enough
* It's not too large
* It has a well defined and bounded scope
* It relies on asynchronous calls
* It relies on network
* I know the algorithm
* It's fun and useful

## Who are you?

Just a Free Software enthusiast, seeking The Perfect Programming Language.

I mostly work with Python and C and played a bit with hipsters stuff like Go
and Rust.

## Can I help?

Yes! Fork the repo and send a metric ton of pull request goodness. You may also
fill an issue on [github](https://github.com/lisael/first-ride-with-pony) As I
hope you did not guess yet, I'm not a native English speaker, I won't get upset
at all if the PR/issue is about grammar, spelling or vocabulary.
