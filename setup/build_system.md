## Build system

The build chain in Pony is minimal. `ponyc` compiles a package and all `use`d
if they are located it the directories in `PONYPATH` environment var.  It's the
only magic that pony performs.

I'm not found of ad-hoc package managers like `gem`, `pip`, `npm`, `cargo` and
al, for a number of reasons I won't ~~troll~~ discus here. So I like this
minimal build chain. You can wrap ponyc in a well known build tool of yours.
Mine is `make`. 

Here's the Makefile stub that will grow with the project:

`commit c7848fe`

`Makefile`

``` Makefile
BUILD_DIR=build
PONYC=ponyc
PONY_SRC=$(wildcard **/*.pony) $(wildcard *.pony)
BIN=$(BUILD_DIR)/$(shell basename `pwd` )

all: $(BUILD_DIR) $(BIN)

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

Note that even if we could improve this Makefile you can use it as-is in all of
your simple Pony projects. I'll try to keep this property all along the
tutorial.
