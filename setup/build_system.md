## Build system

The build chain in Pony is minimal. `ponyc` compiles a package and all `use`d
if they are located it the directories in `PONYPATH` environment var.  It's the
only magic that pony performs.

I'm not found of ad-hoc package managers like `gem`, `pip`, `npm`, `cargo` and
al, for a number of reasons I won't ~~troll~~ discus here. So I like this
minimal build chain. You can wrap ponyc in a well known build tool of yours.
Mine is `make`. 

Here's the Makefile stub that will grow with the project:

`commit 02fbd64`

`Makefile`

``` Makefile
BIN_NAME=$(shell basename `pwd` )

all: clean $(BIN_NAME)

$(BIN_NAME):
	ponyc

clean:
	-rm $(BIN_NAME)
	-rm $(BIN_NAME).o

# debug
print-%  :
	@echo $* = $($*)
```

Note that even if we could improve this Makefile you can use it as-is in all of
your simple Pony projects. I'll try to keep this property all along the
tutorial.
