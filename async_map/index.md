## Chapter 2: An Asynchronous Map

Raft purpose is to store data. Storing a single entry is not that useful. For
this example we need a key-value store. Because the clients will update and
read the map constantly, we need to share the map safely.

We're going to try a naive implementation that wraps a map into a classic
class. We will fail. BTW, failures are good things if we learn from them.

In Pony, actors and behaviours are the way to go when you need to share data
safely. We will fix our code using an actor.

Then we'll discuss about Pony async model, and what it implies in terms of
coding styles and design.

Testing this module is interesting and requires specific techniques precisely
because it's asynchronous.
