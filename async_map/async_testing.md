## Asynchronous testing

Testing asynchronous code is not easy. However, because we use the notifier
pattern we can write a dedicated testing notifier using the recipe in [pony
patterns
book](http://patterns.ponylang.org/testing/notifier-interactions.html). We will
change the recipe a bit. The pattern book relies on a notifier that is passed
to the constructor of the actor. Here we pass the notifier to each behaviour.

`commit c37567a`
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


class _TestNotifier is RaftNotifier
  let _h: TestHelper
  let _expected: RaftResult
  let _complete: Bool
  new iso create(h: TestHelper, expected: RaftResult, complete: Bool = false) =>
    _h = h
    _expected = expected
    _complete = complete

  fun ref apply(result: RaftResult) =>
    match result
    | None => match _expected
      | None => None
      | let v: I64 => _h.fail("got None, expected " + v.string())
      end
    | let i: I64 => match _expected
      | None => _h.fail("got " + i.string() + " expected None")
      | let j: I64 => _h.assert_eq[I64](i,j)
      end
    end
    if _complete then _h.complete(true) end


class iso _TestKVStoreSet is UnitTest
  fun name():String => "_KVStore.set"

  fun apply(h: TestHelper) =>
    let k = _KVStore
    // add a value
    k.set("test1", 1, _TestNotifier(h, None))
    // should be 1
    k.get("test1", _TestNotifier(h, 1))
    // change the value. set returns the old value
    k.set("test1", 2, _TestNotifier(h, 1))
    // should be 2
    k.get("test1", _TestNotifier(h, 2, true))
    h.long_test(500_000_000)
```
> I omitted `_TestKVStoreGet` and `_TestKVStoreDelete` classes for brevity.

Because the `RaftNotifier` is a public interface, we can easily implement a
_TestNotifier. As long as the implementation has an `apply` method, anything
fits.

Pony unittest framework provides methods to deal with asychronous testing
through the [`TestHelper`](http://www.ponylang.org/ponyc/ponytest-TestHelper/).

`long_test(timeout: U64 val)` tells the helper to wait for `timeout`
nanoseconds after the test method (`fun apply(h: TestHelper)`) exits. Without
this call: `h.long_test(500_000_000)`, the test would complete as soon as we
exit `apply`, regardless of the behaviours that we called in the function. They
may or may not have complete.

Because the helper was told to continue, we must tell it when to stop. Either
the tests fail (`_h.fail("got None, expected " + v.string())`) or we
explicitely call `_h.complete(success: Bool val)`. In our testing code this
adds a bit of complexity, as the notifier has to complete the tests only if
it's the last one. That's the point of the `_complete` boolean field.
