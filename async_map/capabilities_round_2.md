## Capabilities - Round 2

So. Now we have written and tested our shared data store, let's use it!

Shared? Are you sure? We had fixed the code to make it usable on a `ref` variable.
`ref` means "I need to be able to read and write the object", but it also means
"I deny read and write on this object to all the other references to this
object". That's how capabilities are used by the compiler to detect data races.
If, at the same time, two or more `ref` reference to the same object exist in
separate threads, there will be data races. The compiler won't let us do that.

Capabilities in pony are more about denying permissions to the rest of the code
than adding permissions.
