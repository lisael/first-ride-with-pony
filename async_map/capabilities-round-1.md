## Capabilities - Round 1

To implement our data store, let's first try a naive code using a `class`:


`raft/server/datastore.pony`
```pony
use "collections"

class _KVStore
  var _data: Map[String, I64] = Map[String, I64]

  fun set(key: String, value: I64): (I64 | None) =>
    _data.update(key, value)
      
  fun get(key: String): (I64 | None) =>
    try _data(key) else None end

  fun delete(key: String): (String, I64) =>
    try _data.remove(key) else (key, I64(0)) end
```

### Complex types

The new notion we introduce here is complex types. The return types of our functions
are composed. `set` and `get` may return either an I64 or None. `delete` returns
a tuple composed of a String and an I64. It's not very consistent, at the moment, we
will soon make `delete` return `(I64 | None)` too.

### Type alias

Pony has the keyword `type` that creates type alias. Here we could write 

```
type RaftValue is I64
type RaftResult is (RaftValue | None)

class _KVStore
  var _data: Map[String, RaftValue] = Map[String, RaftValue]

  fun set(key: String, value: RaftValue): RaftResult =>
    _data.update(key, value)
```

Not only it saves typing but it also make the code more future-proof. If we want our
data store to accept other types than I64. 

> Note that type alias cannot be recursive. This limitation is due to the simple
> implementation of aliases in the compiler: it's just a text substitution. This may
> evolve in the future.

### None

None is something! Pony has no NULL pointer (thus no `NullPointerException`...
Java readers start to sweat, just reading this word). Pythonists here may
recognize their good ol' None.  Well, actually... Because of static typing
Pony's None never causes runtime exceptions ( `NoneType has no attribute...`).
None is a [`primitive`](http://tutorial.ponylang.org/types/primitives.html)

### Compile

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
/home/lisael/pony-raft/raft/server/datastore.pony:13:21: receiver type is not a subtype of target type
    try _data.remove(key) else (key, I64(0)) end
                    ^
    Info:
    /home/lisael/pony-raft/raft/server/datastore.pony:13:9: receiver type: this->HashMap[String val, I64 val, HashEq[String val] val] ref
        try _data.remove(key) else (key, I64(0)) end
            ^
    /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/collections/map.pony:103:3: target type: HashMap[String val, I64 val, HashEq[String val] val] ref
      fun ref remove(key: box->K!): (K^, V^) ? =>
      ^
    /home/lisael/pony-raft/raft/server/datastore.pony:4:14: HashMap[String val, I64 val, HashEq[String val] val] box is not a subtype of HashMap[String val, I64 val, HashEq[String val] val] ref: box is not a subtype of ref
      var _data: Map[String, I64] = Map[String, I64]
                 ^
```

Ouch! This code, which looks correct at a first glance, don't even compiles. The
compiler made the same observation we did: if we share an instance of this
class, we risk data races and therefore, we run into troubles.

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
and without programmer's hints, it assumes that an object can be shared for
reading.  The hints a programmer can give to the compiler in pony are
capabilities.

We will talk a lot about capabilities in this tutorial as they are the main
step on pony learning curve (I'm not sure I passed this step myself). Sure they
will annoy you as a pony beginner, but think it's for your own good. This
feature will detect hidden data races before your users, and will always help
you to fix it right.

### How can we fix our code, then?

Pony doesn't expose locking primitives because locking is hard to make it right.
It's even too hard for a compiler. Pony compiler is able to performs capabilities
checks and to guarantee dead-lock-free binary because it enforces those two
limitations:

1. No lock. At all. Ever
2. No blocking call. At all. Ever

Locks and blocking calls are at the heart of most of asynchronous paradigms
(Python and Java asyncIO, Go channels, C/C++ hand-crafted async code, ...).
OTOH, in pony these rules make it easy for the compiler to check data safety.

> As a cool side effect, the capabilities allow a much simpler garbage collector,
> which is easy to make it run concurrently (relatively, huh, a GC is never
> trivial). Go, for example, with an army of google engineers have just released
> a partially concurrent GC.

So let's try to fix the code adding explicit capabilities to our class.

The default capabilities on functions is [`box`](# "I need to be able to read
from this, but I won't write to it") which means "I need to be able to read
from this, but I won't write to it". Capabilities on functions are hints to the
compiler to make it know the capabilities of the target in the calling code.

In other words we promise to the compiler that we will never need a more
permissive cap than [`box`](# "I need to be able to read from this, but I won't
write to it") on references of the targets we call these functions on.

The problem here is that we did break our promise, because we tried to mutate
our object, at least in `set` and `delete`.

#### How the compiler can tell ?

Good question. If you can't read English (say, you're a compiler), it's
not obvious that `set` and `delete` mutate the target while `get` doesn't.
Let me introduce viewpoint adaptation. It's a fairly complex name for a simple
concept: the capabilities of an instance's field is the least common denominator
of the permissions on the reference of the target and the permissions on the
field, as seen from within the target.

The default capability of a field is [`ref`](# "I need to the permission to
read and write"), which means "I need to be able to read and write to the
object". So, as seen from the function, `_data` should be writeable. But
because the function is only callable from a [`box`](# "I need to be able to
read from this, but I won't write to it") target, the capabilities of `_data`
become [`box`](# "I need to be able to read from this, but I won't write to
it"), the least common denominator of [`ref`](# "I need to the permission to
read and write") and [`box`](# "I need to be able to read from this, but I
won't write to it"). Then in `set` we try to call `Map.update`. `Map.update`
only accept [`ref`](# "I need to the permission to read and write") target. At
this point the type checker yields. Hopefully it yields useful info, that is
worth reading and understanding.

First, it let us know where the error happened and what is the kind of the error.

```
Error:
/home/lisael/pony-raft/raft/server/datastore.pony:7:17: receiver type is not a subtype of target type
    _data.update(key, value)
                ^
```

> As you maybe noticed it's a type error. In pony `MyType ref` __is not the same
> type__ as `MyType box`, a bit like `const uint32` is not the same type as `uint32`
> in C++. This also means that capability checks are only performed during the
> compilation. __The runtime doesn't know about capabilities__

Then the compiler gives us hints to understand the error.

> The readability is somewhat obfuscated by type alias and generics expansions.
> In the message, `HashMap[String val, I64 val, HashEq[String val] val]` is
> our `Map[String, I64]`.

The caller point of view:

```
    Info:
    /home/lisael/pony-raft/raft/server/datastore.pony:7:5: receiver type: this->HashMap[String val, I64 val, HashEq[String val] val] ref
        _data.update(key, value)
        ^
```

The callee perspective:

```
    /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/collections/map.pony:58:3: target type: HashMap[String val, I64 val, HashEq[String val] val] ref
      fun ref update(key: K, value: V): (V^ | None) =>
      ^
```

And finally why they clash:

```
    /home/lisael/pony-raft/raft/server/datastore.pony:4:14: HashMap[String val, I64 val, HashEq[String val] val] box is not a subtype of HashMap[String val, I64 val, HashEq[String val] val] ref: box is not a subtype of ref
      var _data: Map[String, I64] = Map[String, I64]
                 ^
```

### A fix, finally

Now the fix is obvious: allow [`ref`](# "I need to the permission to read and
write") target on offending functions:

`raft/server/datastore.pony`

```pony
  fun ref set(key: String, value: I64): (I64 | None) =>
    _data.update(key, value)

  fun ref delete(key: String): (String, I64) =>
    try _data.remove(key) else (key, I64(0)) end
```

Yay! It does compile...

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

... well, actually... `ponyc` only compiles running binaries
(over-simplification spotted! Read about `--library` in `ponyc -h`). It needs
an entry point that does actually something. We don't want to add a `Main` to
all of our packages, and we need to test the code. Let's add tests for our class. 
