## The notifier pattern

To fetch the results from the actor and do useful stuff with it, we're going to
use the notifier pattern. It's a common pattern in pony. So common that there's
a [dedicated
chapter](http://patterns.ponylang.org/testing/notifier-interactions.html) about
how to test it in the pony patterns book.

The underlying idea is simple: we pass an object to the actor, with a known
interface. When the behaviour has computed the data it calls a method on
the object. Yes, I know what you think... It's just a callback. Sort of.
But pony allows a lot of strategies here. I'd say it's a callback on steroids.
We will use a few of them in the tutorial.

Here are 2 strategies

In the pattern book, the notifier is a constructor argument of the actor.
It's a common pattern in the standard lib, especially in networking stuff.
The stdlib exposes an actor that does all the gory network details, and
the user has only to write a notifier that react to events on the connection.
Then, the user passes an instance of their notifier in the constructor of
a connection. This instance is called for each event.

Here, we will pass explicitly a notifier with each call to a behaviour.

`commit c37567a`
`raft/server/datastore.pony`
```pony
use "collections"

type RaftValue is I64
type RaftResult is ( I64 | None )

interface RaftNotifier
  fun ref apply(resp: RaftResult)

actor _KVStore
  var _data: Map[String, RaftValue] = Map[String, RaftValue]

  be set(key: String, value: RaftValue, notify: RaftNotifier iso) =>
    notify(_data.update(key, value))
      
  be get(key: String, notify: RaftNotifier iso) =>
    notify(try _data(key) else None end)

  be delete(key: String, notify: RaftNotifier iso) =>
    notify(try
      (_, let v: RaftValue) = _data.remove(key)
      v
    else
      None
    end)
```

> I added a bit more than the notifier, here. I made type aliases to abstract
> out the values, and a refined the delete method to make it return a proper
> `RaftResult`.

First thing to note is that we use an
[`interface`](http://tutorial.ponylang.org/types/traits-and-interfaces.html)
here. A hard coded class or actor would fit, but we want our actor to be
usable in some ways we don't imagine yet. At least it's useful for the tests,
as we'll see in the next section, to test the actor, we will define a custom
notifier.

Second, we added an `iso` capability to the `RaftNotifier` argument of the
behaviours. Behaviours only allow sendable references as argument. A sendable
reference is one that can be safely shared between actors. If you own a
[`ref`](# "I need to the permission to read and write"), you can't share it
as-is with another actor. Two `ref` aliases can't live at the same time in
separate actors. Imagine this code:

```pony
actor SchroedingerBox
  be kitty_kitten(cat: Cat ref) =>
    cat.eat()

class Lab
  let sbox: SchroedingerBox
  fun cute_exeperiment() =>
    let cat: Cat ref = cat.alive()
    sbox.kitty_kitten(cat)
    cat.kill()
```    

Will the cat eat something? Modern computing doesn't support the superposition
of variables state (Is it the point of quantum computers? I should read a bit).
Whatever, Schödinger is not our friend, as programmers. Pony won't let him do
his experiments with your code.

[`iso`](# "I need the only readable and writeable reference")
is safe, because it 
