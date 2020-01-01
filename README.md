# markli-buildah - Literate container definitions

## What?

markli-buildah is a thin wrapper around
[markli](https://github.com/lichtzeichner/markli) and
[buildah](https://github.com/containers/buildah). It allows you to generate
container images from a Markdown file.

## Why?

Ever got annoyed by long unpleasant command chains in a single `Dockerfile`
`RUN` statement? Yes, me too. Buildah is more pleasant in this case, but
sometimes you still have to either have a long quoted code block inside a shell
script or you have to split it into multiple `buildah run` invocations. Why not
use a file format like Markdown, which has good support for mixing embedded code
in different languages?

## Example

Here is a simple example:

```sh
### FILE: 00_FROM.txt
ubuntu:bionic
```

```sh
### FILE: 10_RUN_setup.sh
set -euo pipefail
apt update
apt dist-upgrade -y
apt install -y --no-install-recommends golang
apt-get clean -y
rm -rf /var/lib/apt/lists/*
```

```sh
### FILE: 20_CONF.txt
--cmd '/bin/bash -li'
```

Then execute the containing Markdown with markli-buildah. In fact you can even
execute this `README.md` with markli-buildah:

```sh
markli-buildah --tmp-dir -i README.md --tag $TAG
```

This is equivalent to this shell script:

```sh
set -euo pipefail
CONTAINER=$(buildah from ubuntu:bionic)

buildah run $CONTAINER -- apt update
buildah run $CONTAINER -- apt dist-upgrade -y
buildah run $CONTAINER -- apt install -y --no-install-recommends golang
buildah run $CONTAINER -- apt-get clean -y
buildah run $CONTAINER -- rm -rf /var/lib/apt/lists/*

buildah config --cmd '/bin/bash -li' $CONTAINER

buildah commit "$CONTAINER" "$TAG"
```

More examples can be found inside the `examples/` directory.

## More details

If you want to make use of the default options you should put your code into a
file called `markli-buildah.md`, by default the containing directory name will
be used as the image tag.

Place your instructions into Markdown code blocks starting with a _markli_ FILE
pragma (see the markli documentation for more info). The pragma file path has to
be simple path, not containing any directories, and it has to match a certain
convention: `<SEQ>_<OPCODE>_<SUFFIX>`. The `<SEQ>` part is used for sorting, and
specifies the order in which markli-buildah will execute the blocks. The
`<OPCODE>` specifies how it will be executed by markli-buildah. `<SUFFIX>` is
arbitrary and will not be further interpreted.

### Opcodes

| Opcode   | Translated                                  | Meaning                                                                                                                         |
| -------- | -----------                                 | --------                                                                                                                        |
| _FROM_   | `buildah from <CONTENTS>`                   | Specifies the base image, should usually be the first block.                                                                    |
| _SHELL_  | -                                           | Set the internal SHELL variable, see _RUN_.                                                                                     |
| _RUN_    | `buildah run $CONTAINER -- $SHELL < <FILE>` | Pipes a code block into `buildah run`.                                                                                          |
| _CONF_   | `buildah config <LINE> $CONTAINER`          | For each non empty line, execute `buildah config`                                                                               |
| _BLD_    | `buildah <LINE>`                            | For each non empty line, execute `buildah`, you can use the `$CONTAINER` variable in `<LINE>`                                   |
| _EVAL_   | `eval <CONTENTS>`                           | Eval the file contents from inside the `markli-buildah` bash script, gives the most flexibility, useful to set shell variables. |

## Command synopsis

TODO

## Installation

First make sure you have `markli` and `buildah` installed, `markli` can be installed like this:
```sh
go get -v github.com/lichtzeichner/markli
```

Then just put the `markli-buildah` script somewhere in your `$PATH` and make it executable
```sh
curl -sSf "https://raw.githubusercontent.com/simonfxr/markli-buildah/master/markli-buildah" -o markli-buildah && chmod +x markli-buildah
```
