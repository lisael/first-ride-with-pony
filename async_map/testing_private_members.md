## Testing private members

Let's add a simple test for our _KVStore:

`test/main.pony`
```pony
use "ponytest"
use server = "raft/server"

actor Main is TestList
  new create(env: Env) =>
    PonyTest(env, this)

  new make() =>
    None

  fun tag tests(test: PonyTest) =>
    test(TestKVStore)

class iso TestKVStore is UnitTest
  fun name():String => "_KVStore"

  fun apply(h: TestHelper) =>
    let k = sever._KVStore
    var result = k.set("test1", 1)
    match result
    | None => None
    | let v: I64 => h.fail("The map should be empty")
    end
```

Run this:

```
make test
[...]
Error:
/home/lisael/pony-raft/test/main.pony:21:19: can't access a private type from another package
    let k = server._KVStore
[...]
```

Because `_KVStore` starts with a `_` it's private. We can use it only from
inside its package. The solution here is to move the test inside `raft/server`
package, make the test public and call it from `test` package.

`raft/server/test.pony`
```pony
use "ponytest"

class iso TestKVStoreSet is UnitTest
  fun name():String => "_KVStore"

  fun apply(h: TestHelper) =>
    let k = _KVStore
    var result = k.set("test1", 1)
    match result
    | None => None
    | let v: I64 => h.fail("The map should be empty")
    end
```

`test/main.pony`
```pony
  fun tag tests(test: PonyTest) =>
    test(server.TestKVStoreSet)
```

Correct. The tests compile and pass. But what is the point of making a private
member if the test package is still heavily coupled with this member tests ? If we
refactor `raft/server` package, we will need to change `test` to add/remove
tests. My solution is to provide a `TestList` in all packages, named `Tests`,
and call its tests from our `test.Main` `TestList`.

`raft/server/test.pony`
```pony
use "ponytest"

class Tests is TestList 
  new create() =>
    None

  new make() =>
    None

  fun tag tests(test: PonyTest) =>
    test(_TestKVStoreSet)

class iso _TestKVStoreSet is UnitTest
  fun name():String => "_KVStore.set"

  fun apply(h: TestHelper) =>
    let k = _KVStore
    var result = k.set("test1", 1)
    match result
    | None => None
    | let v: I64 => h.fail("The map should be empty")
    end
```

`test/main.pony`
```pony
  fun tag tests(test: PonyTest) =>
    server.Tests.tests(test)
```

Note that we made the tests themselves private. The only entry point is the class
tests.
