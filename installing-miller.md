# Installing Miller

Miller is one static binary named
`mlr`. There is nothing to configure
afterwards and nothing that runs in the
background; installing Miller means
putting that one file somewhere on your
`PATH` and proving it answers.

## What the lab did

The lab used the project's own release.
From the Miller releases page on GitHub
(`github.com/johnkerl/miller`), it took
the archive for its platform —
`miller-6.22.0-linux-amd64.tar.gz` — and
unpacked it into a scratch folder:

```bash
tar xzf miller-6.22.0-linux-amd64.tar.gz
./miller-6.22.0-linux-amd64/mlr --version
```

```text
mlr 6.22.0
```

That is the version every transcript in
this guide was made with. Moving the
`mlr` file into a directory on your
`PATH` (a `~/bin` or `/usr/local/bin`)
finishes the install; the archive holds
nothing else the program needs.

Check: `mlr --version` answers with a
version and nothing else, and
`mlr --csv cat` on any small CSV you own
prints the file back to you. If the
shell answers "command not found", the
binary is not yet on your `PATH` — the
install is a file placement, so the fix
is a path, not a reinstall.

## The other routes

The same binary is packaged widely, and
any of these is the real Miller — this
chapter lists them from the project's
documentation; the lab verified only the
release-tarball route above.

- **Package managers.** Homebrew
  (`brew install miller`), MacPorts, and
  the package repositories of most Linux
  distributions carry Miller, as do
  Windows routes such as Chocolatey and
  winget. A distribution's package may
  lag the project's releases by a version
  or two; for this guide's commands,
  anything in the Miller 6 line speaks
  the same language.
- **Building from source.** Miller is
  written in Go, and the project's
  repository builds with the standard Go
  toolchain for anyone who needs an
  unusual platform. The release list is
  wide — Linux, macOS, FreeBSD and
  Windows builds, on several processor
  architectures — so source is the last
  resort, not the usual one.

## A note on the name

The program is called Miller and the
command is `mlr`; both are named for
the project's author, John Kerl, by way
of the miller's trade — the person who
grinds grain into flour, which is a fair
description of what the tool does to
data. In this guide "Miller" is the
tool and `mlr` is what you type.
