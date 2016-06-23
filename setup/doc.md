## Documentation

Pony comes with a documentation generator. The documentation is embed into the
code as docstrings. Let's add a bit of package documentation.

`raft/main.pony`

```pony
"""
Package: pony-raft

A pure pony Raft implementation. This was created for educational purpose, as
an example for https://lisael.gitbooks.io/first-ride-with-pony/content/.
"""
actor Main
  """
  Our Main stub. Does nothing interesting yet.
  """
  new create(env: Env) =>
    """
    Realy nothing interesting, but it's documented, at least.
    """
    env.out.write("Hello, Pony\n")
```

To generate the docs:

```
$ ponyc --docs -o build --pass=docs raft
Building builtin -> /usr/local/lib/pony/0.2.1-955-g4eb42e2/packages/builtin
Building raft -> /home/bdupuis/projects/perso/pony-raft/raft
Writing docs to build/raft-docs/
$ ls build/raft-docs/
docs  mkdocs.yml
$ ls build/raft-docs/docs/
builtin-AlignCenter.md             builtin-FormatUTF32.md
[...]
builtin-FormatSettingsHolder.md    builtin-USize.md
builtin-FormatSettingsInt.md       index.md
builtin-FormatSettings.md          raft--index.md
builtin-FormatSpec.md              raft-Main.md
```

So raft generated [mkdocs](http://www.mkdocs.org/) documentation.

TODO: add an example of HTML generation and a make target.

```Makefile
doc: $(PONY_SRC) 
	$(PONYC) -o $(BUILD_DIR) --docs --path . --pass=docs $(PKG)
```
