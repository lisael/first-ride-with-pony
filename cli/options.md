## Options

### Configurations in Pony

Pony paradigms impose that there are no global variables. We must explicitly
share the configuration values with the actors that need it. I choose to
create a Config class that stores all configuration values and to pass it
around as needed as a ``val`` reference. The class definition holds the 
default values. One other strategy is to explitely share the configuration
values but it's quickly cumbersome as you have to pass the values from one
actor to another, rewritting a whole chain of behaviour calls each time a
configuration variable is added or removed.

### Options

The [`options`](http://www.ponylang.org/ponyc/options--index/) package provides
a simple API to read options from the command line. The default constructor
of our `Config` class is in charge of the parsing of the command line.

`commit 750bbae`

`raft/main.pony`

``` pony 
use "options"

class Config
  let env: Env
  var host: String = "127.0.0.1"
  var port: String = "9876"

  new val create(env': Env) =>
    env = env'
    var options = Options(env) +
      ("host", "h", StringArgument) +
      ("port", "p", StringArgument)
    try 
      for opt in options do
        match opt
        | ("host", let arg: String) => host = arg
        | ("port", let arg: String) => port = arg
        | let err: ParseError => err.report(env.out) ; error
        end
      end
    end


actor Main
  new create(env: Env) =>
    let config = Config(env)
    env.out.write("Listening on " + config.host + ":" + config.port + "\n")
```

### Code review

Let's walk through the code.

#### Using packages

First we `use` the package `options`. It's a standard package, provided by ponyc.
Once `use`d, every public class, actor, type, and primitive defined in `options` are
available in the source file. To avoid name clashes, one can namespace the imported
names with the syntax:

```
use "options" opts

class Main
  let options: opts.Options
```

Conceptualy, it works a lot like C `#include` except that it works at package level, not at
file level and you can optionaly use the namespace trick.

TODO: explain PONYPATH

#### Class definition

In pony a class has a name, fields and methods.

The main particularity of a pony class is that its fields are never empty after the
constructor has returned. There is not even a symbol that means "empty". Thats why
constructor are special methods. They MUST assign a value to ALL fields before they
return, except if the field has a default value (`"127.0.0.1"` or `"9876"` here).

Fields, just like method-local aliases are declared with `let` or `var`. `let` allows
only one assignment to the field, whereas `var` allows multiple assignments. Here,
we could logically declare `host` and `port` with let, as we know that these values
won't change. However, the compiler won't accept `let` because the assignments happen
into a `for` loop. It has no way to be sure that we don't assign the field twice. By
the way it can't prove that we do assign the variables at one point either. Try to remove
the default values, it won't compile anymore because there is a possibility that
`host` and `port` remain empty at the end of the constructor. Indeed, it does happen
if we don't pass any argument to the program.

As a rule of thumb: always use `let` except if you know that you WILL re-assign the
field. First, the compiler can make assumptions and optimise the code. Second, using
`let` documents your code. Third point, if at one point you finally re-assign
the field or alias, the compiler yields an error so you can stop, think and either
change the code or turn the variable into a `var` (the important word here is
"think". Pony main strenght, IMHO is that it requires the user to think about
the correctness of their code flow).

#### Prime and class namespace

TODO: explain

```
env = env'
```

#### Operator overloading

```
    var options = Options(env) +
      ("host", "h", StringArgument) +
      ("port", "p", StringArgument)
```

#### Type inference

TODO:

```
    let config = Config(env)
```

 <=>

```
    let config: Config val = Config(env)
```

because the compiler can infer the type and the capabilities.
