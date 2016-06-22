## Testing

I won't emphasis here about why [you should
test](https://www.google.fr/search?q=why+should+i+unittest). I found that if I
don't create a good test framework at the very startup of my project, it seems
too hard and dull after. Let's just add tests, run them and go on.

Here, again, a lot has been said in [the most precious
tutorial](http://tutorial.ponylang.org/testing/ponytest.html). Another great
resource for tests is the [ponytest package
documentation](http://www.ponylang.org/ponyc/ponytest--index/). Some testing
strategies are explained in [the great book of pony
patterns](http://patterns.ponylang.org/testing/). We will explore a few other
patterns as we add more tests.

`commit 1f42962`

`test/main.pony`

```  pony
use "ponytest"

actor Main is TestList
  new create(env: Env) =>
    PonyTest(env, this)

  new make() =>
    None

  fun tag tests(test: PonyTest) =>
    test(_TestAdd)

class iso _TestAdd is UnitTest
  fun name():String => "addition"

  fun apply(h: TestHelper) =>
    h.assert_eq[U32](4, 2 + 2)
```

`Makefile`

``` Makefile
BUILD_DIR=build
PONYC=ponyc
PONY_SRC=$(wildcard **/*.pony) $(wildcard *.pony)
BIN=$(BUILD_DIR)/$(shell basename `pwd` )
TEST_BIN=$(BUILD_DIR)/test

all: $(BUILD_DIR) test $(BIN)

test: $(TEST_BIN) runtest

$(TEST_BIN): $(PONY_SRC)
	$(PONYC) -o $(BUILD_DIR) test

runtest:
	./$(TEST_BIN)

$(BUILD_DIR):
	mkdir -p $(BUILD_DIR)

$(BIN): $(PONY_SRC) 
	$(PONYC) -o $(BUILD_DIR)

clean:
	-rm -rf $(BUILD_DIR)

# debug
print-%  :
	@echo $* = $($*)
```

Our Makefile is now usable and useful. It detects changes and re-build only if
.pony files have been modified. Let's play a little with it.

```
$ make clean
rm -rf build
$ make
mkdir -p build
ponyc -o build test
Building builtin -> /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/builtin
Building test -> /home/lisael/pony-raft/test
Building ponytest -> /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/ponytest
Building time -> /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/time
Building collections -> /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/collections
Generating
 Reachability
 Selector painting
 Data prototypes
 Data types
 Function prototypes
 Functions
 Descriptors
Optimising
Writing build/test.o
Linking build/test
./build/test
1 test started, 0 complete: addition started
1 test started, 1 complete: addition complete
---- Passed: addition
----
---- 1 test ran.
---- Passed: 1
ponyc -o build
Building builtin -> /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/builtin
Building . -> /home/lisael/pony-raft
Generating
 Reachability
 Selector painting
 Data prototypes
 Data types
 Function prototypes
 Functions
 Descriptors
Optimising
Writing build/pony-raft.o
Linking build/pony-raft
$ make
./build/test
1 test started, 0 complete: addition started
1 test started, 1 complete: addition complete
---- Passed: addition
----
---- 1 test ran.
---- Passed: 1
$ make clean
```
