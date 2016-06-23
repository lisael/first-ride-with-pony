## Hello World

I always start a project with the most simple program, just to be sure that
everything works as expected.

I also always `git init` a project even if I don't plan to distribute it. With
git, you're free to experiment and rollback to a known working version.

Note that the filename has no importance here. All files in a directory are
concatenated at compile time (well almost: package imports via the `use`
keyword are limited to a file)

A directory of pony source files is a package, and may be imported as a whole.
Private symbols (classes, fields, and methods) can be accessed only from within
the package.

Let's create the package and a program

```
$ mkdir pony-raft
$ cd pony-raft
$ git init
# create the package dir
$ mkdir raft
```

`commit 4b8f70ba`

`raft/main.pony`
``` pony
actor Main
  new create(env: Env) =>
    env.out.write("Hello, Pony\n")
```

I won't spend a lot of time on these lines of code, as you probably already
read [the excellent
tutorial](http://tutorial.ponylang.org/getting-started/how-it-works.html).

We can now compile and run the package:

```
$ ponyc raft
Building builtin -> /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/builtin
Building raft -> /home/lisael/pony-raft/raft
Generating
 Reachability
 Selector painting
 Data prototypes
 Data types
 Function prototypes
 Functions
 Descriptors
Optimising
Writing ./raft.o
Linking ./raft1
```

This create an executable file named after the enclosing package:

```
$ ./raft1
Hello, Pony
```

The `1` at the end of `raft1` was added by `ponyc` to resolve the name collision
with the package `raft/`. We need to build our program more cleanly
