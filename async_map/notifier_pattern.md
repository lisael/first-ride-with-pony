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

In the pattern book, the notifier is a constructor argument of the actor. It's
a common pattern in the standard lib, especially in networking stuff. The
stdlib exposes an actor that does all the gory network details, and the user
has only to write a notifier that reacts to events on the connection. Then,
the user passes an instance of their notifier in the constructor of a
connection. This instance is called for each event.

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

Second, we added an [`iso`](# "I need the globally unique readable and
writeable reference") capability to the `RaftNotifier` argument of the
behaviours. Behaviours only allow sendable references as argument. A sendable
reference is one that can be safely shared between actors. If you own a
[`ref`](# "I need to the permission to read and write"), you can't share it
as-is with another actor. Two [`ref`](# "I need to the permission to read and
write") aliases can't live at the same time in separate actors. Imagine this
code:

```pony
actor SchroedingerBox
  be kitty_kitten(cat: Cat ref) =>
    cat.eat()

  fun open() =>
    cat.is_fed()

class Lab
  let sbox: SchroedingerBox
  fun cute_exeperiment() =>
    let cat: Cat ref = Cat.alive()
    sbox.kitty_kitten(cat)
    cat.die()
```    

Will the cat eat something? In our `cute_experiment` we first tell the cat to
eat, which it will eventually do. Eventually, I say, because `eat` is a
behaviour. It's asynchronous, and the call to `eat` returns immediately without
doing anything. Then, in (finally-not-so-)`cute_experiment` we kill the cat. Be
`die` a function or a behaviour is not important, we can't predict which method
(`eat` or `die`) will be executed first. Mr Shrödinger tells that as long as we
don't `open` the box, we can tell that the cat is fed __and__ not fed with
probabilities depending on a wave function. Modern computing doesn't support
the superposition of variables states, yet (Is it the point of quantum
computers? I should read a bit, I guess). Whatever, Mr Schödinger is not our
friend, as programmers. Pony won't let him do his –pretty silly, if you ask me–
experiments with your code and won't compile.

[`iso`](# "I need the globally unique readable and writeable reference") is
safe, because it guarantees the global uniqueness of the reference. When you
pass an [`iso`](# "I need the globally unique readable and writeable
reference") to a method call you have to
[`consume`](http://tutorial.ponylang.org/capabilities/consume-and-destructive-read.html)
it. It's no longer available in the caller's code. Because it enforces this
rule, the compiler knows that it's always safe to read an write an [`iso`](# "I
need the globally unique readable and writeable reference"). We may need to
mutate the `RaftNotifier`, so we accept [`iso`](# "I need the globally unique
readable and writeable reference") here.

There are two more sendable reference capabilities: [`var`](# "Globally
immutable") and [`tag`](# "I don't need to read or write, only identity and
behaviours"). [`var`](# "Globally immutable") references are globally
immutable. As a consequence, it's always safe to read from as no one can write
to. [`tag`](# "I don't need to read or write, only identity and behaviours") is
opaque. Because the callee can't read or write it, the rest of the code can
safely do anything with the reference's aliases while the callee uses it. 
