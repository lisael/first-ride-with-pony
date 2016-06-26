## Capabilities - Round 2

So. Now we have written and tested our shared data store, let's use it!

Shared? Are you sure? We had fixed the code to make it usable on a `ref`
variable.  `ref` means "I need to be able to read and write the object", but it
also means "I deny read and write on this object to all the other references to
this object". That's how capabilities are used by the compiler to detect data
races.  If, at the same time, two or more `ref` reference to the same object
exist in separate threads, there will be data races. The compiler won't let us
do that.

__Capabilities in pony are more about denying permissions to the rest of the code
than adding permissions.__

The class we coded so far is useless in our use case. We need a way to be sure
that all operations on the map are serialized in a single thread. Pony provides
no locking primitive. Synchornisation in Pony is achieved using actors.

The full code of the class, including tests, is at `commit a2333b6`. It contains
refinements I did not discussed here (TODO), on the `delete` method.
