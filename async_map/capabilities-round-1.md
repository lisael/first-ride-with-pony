## Actors and behaviours

To implement our data store, let's first try a naive code using a `class`:

`commit d83dd62`

`raft/server/datastore.pony`
```pony
use "collections"

class _KVStore
  var _data: Map[String, I64] = Map[String, I64]

  fun set(key: String, value: I64) =>
    _data.update(key, value)
      
  fun delete(key: String) =>
    try _data.remove(key) else I64(0) end

  fun get(key: String) =>
    try _data(key) else 0 end
```

In most common languages this looks like a good starter:

1. We wrap a standard `collections.Map` as a private field
2. We add methods to `set`, `get` and `delete` values

However you certainly noticed that because we want to share the data store, we
must add locking or synchronize accesses to the underlying map to avoid data
race. We (apparently) can't do this, because there is no locking primitive in Pony!
Don't fill a bug report, it's a feature.

Let's try to compile this anyway:

```
$ ponyc raft/server
Building builtin -> /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/builtin
Building raft/server -> /home/lisael/pony-raft/raft/server
Building collections -> /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/collections
Building ponytest -> /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/ponytest
Building time -> /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/time
Error:
/home/lisael/pony-raft/raft/server/datastore.pony:7:17: receiver type is not a subtype of target type
    _data.update(key, value)
                ^
    Info:
    /home/lisael/pony-raft/raft/server/datastore.pony:7:5: receiver type: this->HashMap[String val, I64 val, HashEq[String val] val] ref
        _data.update(key, value)
        ^
    /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/collections/map.pony:58:3: target type: HashMap[String val, I64 val, HashEq[String val] val] ref
      fun ref update(key: K, value: V): (V^ | None) =>
      ^
    /home/lisael/pony-raft/raft/server/datastore.pony:4:14: HashMap[String val, I64 val, HashEq[String val] val] box is not a subtype of HashMap[String val, I64 val, HashEq[String val] val] ref: box is not a subtype of ref
      var _data: Map[String, I64] = Map[String, I64]
                 ^
Error:
/home/lisael/pony-raft/raft/server/datastore.pony:10:21: receiver type is not a subtype of target type
    try _data.remove(key) else I64(0) end
                    ^
    Info:
    /home/lisael/pony-raft/raft/server/datastore.pony:10:9: receiver type: this->HashMap[String val, I64 val, HashEq[String val] val] ref
        try _data.remove(key) else I64(0) end
            ^
    /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/collections/map.pony:103:3: target type: HashMap[String val, I64 val, HashEq[String val] val] ref
      fun ref remove(key: box->K!): (K^, V^) ? =>
      ^
    /home/lisael/pony-raft/raft/server/datastore.pony:4:14: HashMap[String val, I64 val, HashEq[String val] val] box is not a subtype of HashMap[String val, I64 val, HashEq[String val] val] ref: box is not a subtype of ref
      var _data: Map[String, I64] = Map[String, I64]
                 ^
```

Ouch! This code which looks correct at a first glance don't even compiles. The
compiler had the same observation we had: if we share an instance of this
class, we run into troubles.

It's the main feature of pony: you just cannot write code that can lead to data
races. Many compilers perform static type checks to ensure that an instance you
call a method on has this method. Pony also checks that you can safely (in
terms of data synchronisation) call a method or access a field. When it comes
to data race, the development cycle with [whatever language] is:

```
  >  code --> compile --> CI --> production --> data-race or dead lock found by the user
 |_____________________________________________________________________________________|
```

It's because data races and dead locks are hard to predict for the human brain. We're
not good at thinking asynchronous systems.

In pony the cycle is more like

```
  > code --> compile --> CI --> production 
 |                 |
 |___________  data race
```

Pony uses computers where they are good at, and let your brain do what it does
the best: write great software.

### How does the compiler know that we want to share instances of this class?

We did not tell pony that we don't! Pony comes with sane and handy defaults,
and without programmer's hints, it assumes that an object is shareable. The
hints a programmer can give to the compiler in pony are capabilities.

We will talk a lot about capabilities in this tutorial as they are the main
step on pony learning curve (I'm not sure I passed this step myself). Sure they
will annoy you as a pony starter, but think it's for your own good. This
feature will detect hidden data races before your users, and will always help
you to fix it right.

### How can we fix our code, then?

Pony doesn't expose locking primitives because locking is hard to make it right.
It's even too hard for the compiler. Pony compiler is able to performs capabilities
checks and to guarantee dead-lock-free binary because it enforces those two
limitations:

1. No lock. At all. Ever
2. No blocking call. At all. Ever

Locks and blocking calls are at the heart of most of asynchronous paradigms
(python and java asyncIO, Go channels, C/C++ hand-crafted async code, ...).
OTHO in pony these rules make it easy for the compiler to check sharing safety
and to trigger the GC (thus: the concurrent GC code is straightforward -- I
won't say it's trivial, a GC is never trivial --, as compared to Go concurrent
GC for example).

So let's try to fix the code adding explicit capabilities to our class:

```pony
use "collections"

class _KVStore
  var _data: Map[String, I64] iso = recover iso Map[String, I64] end

  fun iso set(key: String, value: I64) =>
    _data.update(key, value)
      
  fun iso delete(key: String) =>
    try _data.remove(key) else I64(0) end

  fun iso get(key: String) =>
    try _data(key) else 0 end
```

It does compile (except that there is no Main):
```
$ ponyc raft/server
Building builtin -> /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/builtin
Building raft/server -> /home/lisael/pony-raft/raft/server
Building collections -> /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/collections
Building ponytest -> /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/ponytest
Building time -> /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/time
Generating
Error:
no Main actor found in package 'server'
```

How does the fix work?

We added the `iso` capability to all methods. We tell the compiler to allow us
to call this method ONLY on `iso` references of _KVStore instances. `iso`
references are globally unique, and the compiler will check and enforce this
property in caller code. Because the compiler knows that whenever `set` method
is called there is only one reference to the instance, it decides that the code
is safe and compiles with no complains.

But, a globaly unique reference is not what we want, is it?

Good point. At first we plan to make a shared data store. It works but it's not
shareable anymore. What we need is a synchonisation mechanism, a way to be sure
the methods are called in the correct order. Good news: pony provides this kind
of objects: actors.


