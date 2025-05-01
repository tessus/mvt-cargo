# Overview

`cargo install --git` does not retrieve tags or branch info.

My command is: `cargo install --git https://github.com/tessus/mvt-cargo.git --branch xyz`

So, cargo install fetched the correct branch but renamed it to master and did not fetch any tags either. :thinking:

P.S.: Please note that this example is a bit exaggerated. In most cases the most recent tag should match the info in `Cargo.toml`. I only used it to even more underscore the issue I am seeing.

# Details

## cargo install

```bash
$ cargo install --git https://github.com/tessus/mvt-cargo.git --branch xyz
$ mvt-cargo
Version: 0.1.0-0123b0f4
```

## manual checkout and cargo build/install

```bash
$ git clone -b xyz https://github.com/tessus/mvt-cargo.git
$ cd mvt-cargo
$ cargo build --release
$ ./target/release/mvt-cargo
Version: 0.1.1-0123b0f4 (xyz)
```

## Investigation

A `cargo install --git https://github.com/tessus/mvt-cargo.git --branch xyz` creates a somewhat messed-up git directory in `.cargo/git/checkouts/`. e.g. `repo-7f25bc5356a562d5`. I call it messed-up because part of the info is missing (tags) and incorrect (branch).

The binary is then compiled from within this directory. However, in this directory there are no tags from the `https://github.com/tessus/mvt-cargo.git` repository, nor is the branch name correct.
This is why my `build.rs` cannot determine a tag nor the correct branch.

In the `repo-7f25bc5356a562d5` dir, the following happens:

```bash
$ git describe --abbrev=0 --tags
fatal: No names found, cannot describe anything.

$ git rev-parse --abbrev-ref HEAD
master
```

However, it should show the following (which is also shown when I run these commands in the directory cloned with `git clone -b xyz https://github.com/tessus/mvt-cargo.git`)

```bash
$ git describe --abbrev=0 --tags
0.1.1

$ git rev-parse --abbrev-ref HEAD
xyz
```

# Problem

As you can see, the output is different depending on how the binary was built.
It is currently impossible to use a `build.rs` script that generates correct information about the git repo, when using `cargo install --git ....`

## Workaround

- clone the repo
- use `cargo install --path ./path/to/repo` or `cargo build` (from within the repo)

# Feature request

Preserve tags and branch info, when using `cargo install --git ....`
