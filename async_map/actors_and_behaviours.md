## Actors and behaviours

[`actor`](http://tutorial.ponylang.org/types/actors.html) in pony, is a keyword
to define a kind of class that accept `behaviour`s.

Behaviours are like methods, but they are executed asynchronously.  By
asynchronously I mean "eventually, in a given thread".

Behaviours are always executed in a single thread, one after another. As a
consequence, there is no need to worry about data races between behaviours,
because the access to the data is serialized.

Let's change our class to an actor and our functions to behaviours, then!

> `get` and `delete` omitted for brevity

`raft/server/datastore.pony`
```pony
use "collections"

actor _KVStore
  var _data: Map[String, I64] = Map[String, I64]

  be ref set(key: String, value: I64): (I64 | None) =>
    _data.update(key, value)
```

```
make test
[...]
Error:
/home/lisael/pony-raft/raft/server/datastore.pony:6:6: actor behaviour cannot specify receiver capability
  be ref set(key: String, value: I64): (I64 | None) =>
     ^
Error:
/home/lisael/pony-raft/raft/server/datastore.pony:6:45: actor behaviour cannot specify return type
  be ref set(key: String, value: I64): (I64 | None) =>
                                            ^
[...]
```

These two errors show the fundamental usage differences between classes
and actors.

### Behaviours receiver capability

We can't specify receiver capability on behaviours because behaviours are
allways called on [`tag`](# "I don't need to read or write, only identity and
behaviours") capability. [`tag`](# "I don't need to read or write, only
identity and behaviours") means "I need no read/write permission on this
object".

What's the point of owning a reference of something we can't read or write?
First, we can still compare identity. Second, we can still call behaviours.
Capabilities are not a fancy stuff that annoys you, their only purpose is to
guard against data races. As behaviours already guarantee that there's no data
race, it's always safe to call them, whichever capability your reference has,
even the most restrictive.

### Behaviours return type

Because behaviours are asynchronous, there is no point defining a return
type, as the caller function will never see the result. Behaviour calls
are not blocking (remember? No blocking call. Never) and they return
immediately the receiver actor to allow us to chain the behaviours
calls.

What if I still __really need__ the result of a behaviour in the caller
function? Well, you can write a blocking object, that you sync with locks,
and... Wait... __NO, YOU CAN'T__.

We will first fix the code, to make it compile, and then we will find the
way to fetch the results and to test the code of our actor.

> `get` and `delete` omitted for brevity

`raft/server/datastore.pony`
```pony
use "collections"

actor _KVStore
  var _data: Map[String, I64] = Map[String, I64]

  be set(key: String, value: I64) =>
    _data.update(key, value)
```

As expected, the _KVStore actor now compile, but the result checks in the test fail:

```
make test
[...]
Error:
/home/lisael/pony-raft/raft/server/tests.pony:25:7: this pattern can never match
    | None => None
      ^
    Info:
    /home/lisael/pony-raft/raft/server/datastore.pony:6:3: match type: _KVStore tag
      be set(key: String, value: I64) =>
      ^
    /home/lisael/pony-raft/raft/server/tests.pony:25:7: pattern type: None val
        | None => None
          ^
[...]
```

We can remove the pattern matchings in our tests and just check that the behaviours
are called and that we don't have a sneaky type error.

`raft/server/tests.pony`
```pony
use "ponytest"

class Tests is TestList 
  new create() =>
    None

  new make() =>
    None

  fun tag tests(test: PonyTest) =>
    this._test_vkstore(test)

  fun tag _test_vkstore(test: PonyTest) =>
    test(_TestKVStoreSet)
    test(_TestKVStoreGet)
    test(_TestKVStoreDelete)

class iso _TestKVStoreSet is UnitTest
  fun name():String => "_KVStore.set"

  fun apply(h: TestHelper) =>
    let k = _KVStore
    var result = k.set("test1", 1)

class iso _TestKVStoreGet is UnitTest
  fun name():String => "_KVStore.get"

  fun apply(h: TestHelper) =>
    let k = _KVStore
    var result = k.get("test1")

class iso _TestKVStoreDelete is UnitTest
  fun name():String => "_KVStore.delete"

  fun apply(h: TestHelper) =>
    let k = _KVStore
    var result = k.delete("test1")
```

The code at this point is `commit 9bd627f`. It's a good starting point to
play a bit with actors and behaviours.
