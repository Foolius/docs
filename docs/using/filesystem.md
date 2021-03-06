---
navhome: '/docs'
next: True
sort: 6
title: Filesystem
---

# Filesystem (`%clay`)

<div class="row">

<div class="col-md-8">

Urbit has its own revision-controlled filesystem, `%clay`. `%clay` is a
typed, global, referentially transparent namespace. An easy way to think
about it is like typed `git` with continuous sync.

The most common way to use `%clay` is to mount a `%clay` node in a Unix
directory. The Urbit process will watch this directory and automatically
record edits as changes, Dropbox style. The mounted directory is always
at the root of your pier directory.

</div>

</div>

## Quickstart

When developing it's a good idea to use a separate desk.

Create a `%sandbox` desk based on the `%home` desk:

    |merge %sandbox ~your-urbit %home

Most of the time we want to use `%clay` from Unix.

Mount the entire contents of your `sandbox` desk to Unix:

    ~your-urbit:dojo> |mount /=sandbox=

To explore the filesystem from inside Urbit `+ls` and `+cat` are useful:

    ~your-urbit:dojo> +ls /=sandbox=/web

Let's see the contents of `/=sandbox=/web/md`:

    ~your-urbit:dojo> +cat /=sandbox=/web/md

Sync from your friend `~some-ship`'s `%experiment` desk to your
`%sandbox` desk:

    |sync %sandbox ~some-ship %experiment

## Manual

<h3 class="first child">
Paths
</h3>

A path in `%clay` is a list of URL-safe text, restricted to
`[a z]`,`[0 9]`, `.`, `-`, `_` and `~`. Formally this is a `(list knot)`
where each segment is a `@ta`.

Paths can be expressed as either `/foo/bar/baz` or `[%foo %bar %baz]`.
The first three segments of an urbit path are `/plot/desk/case`, for
example `/~dozbud/home/1`.

#### Relative paths

Relative paths are expressed using `%` which is roughly equivalent to
Unix `.`. `%%` is `..` and `%%%` is what you would expect `...` to be.
There are no local relative paths. `foo/bar` must be written
`%/foo/bar`.

#### Substitution

If the default path is `/foo/bar/baz`, `/=/moo` means `/foo/moo`, and
`/=/moo/=/goo` means `/foo/moo/baz/goo`. Also, instead of `/=/=/zoo` or
`/=/=/=/voo`, write `/==zoo` or `/===voo`.

Most commonly this is used to avoid having to know the current revision
number in the `dojo`. Instead of writing
`/~your-urbit/home/4/path/to/file` you can just write
`/=home=/path/to/file`.

### Revision-control

<h3 class="first child">
<code>|mount clay-path [unix-name]</code>
</h3>

Mount the `clay-path` at the unix mount point `unix-name`.

**Examples:**

    |mount %/web

Mounts `%/web` to `/web` inside your pier directory.

    |mount %/gen generators

Mounts `%/gen` to `/generators` inside your pier directory.

### `|unmount [clay-path || unix-name]`

Unmount the path or name from Unix.

**Examples:**

    |unmount %/web

Unmounts the path `%/web` from whatever name it was mounted as.

    |unmount %generators

Unmounts the Unix path `/generators`.

### `|merge desk beak[, =gem strategy]`

Merges a `beak` into a `desk` using an optional merge `strategy`.

A `beak` is a ship-desk-case triple, encoded as a
path(`/~dozbud/home/2`)

**Examples:**

    |merge %home-work /=home=, =gem %fine

Merge `/=home=` into `%home-work` using merge strategy `%fine`.

    |merge %examples ~wacbex-ribmex %examples

Merge the `%examples` desk from `~waxbex-ribmex`

### `|sync desk plot [plot-desk]`

Subscribe to continuous updates from remote `plot` on local `desk`.
`plot-desk` can specify the remote `desk` name. When omitted it is
defaulted to being the same as `desk`. Non-comet urbits have
`|sync %home ~parent %kids` automatically set up (where `~parent` is the
planet that issued a moon, the star that issued a planet, or the galaxy
that issued a star).

**Examples:**

    |sync %home-local ~dozbud %home

    |sync %home ~doznec

### `|unsync desk plot [plot-desk]`

Unsubscribe from updates from remote `plot` on local `desk` with
optional `plot-desk`. Arguments must match original `|sync` command.

Example:

    |unsync %home-local ~dozbud %home

### `|label desk name`

Label the current version of `desk` `name`.

Example:

    |label %home release

### Merge strategies

`%init` - Used if it's the first commit to a desk. Also can be used to
"reinitialize" a desk -- revision numbers keep going up, but the new
revision isn't necessarily a descendent of the previously-numbered
version, allowing merges to be rerun.

`%this` - Keep what's in Bob's desk, but join the ancestry.

`%that` - take what's in Alice's desk, but join the ancestry. This is
the reverse of `%this`. This is different from `%init` because the new
commit has both sides in its ancestry.

`%fine` - "fast-forward" merge. This succeeds iff one head is in the
ancestry of the other.

`%meet`, `%mate`, and `%meld` - find the most recent common ancestor to
use as our merge base.

A `%meet` merge only succeeds if the changes from the merge base to
Alice's head (hereafter, "Alice's changes") are in different files than
Bob's changes. In this case, the parents are both Alice's and Bob's
heads, and the data is the merge base plus Alice's changed files plus
Bob's changed files.

A `%mate` merge attempts to merge changes to the same file when both
Alice and Bob change it. If the merge is clean, we use it; otherwise, we
fail. A merge between different types of changes -- for example,
deleting a file vs changing it -- is always a conflict. If we succeed,
the parents are both Alice's and Bob's heads, and the data is the merge
base plus Alice's changed files plus Bob's changed files plus the merged
files.

A `%meld` merge will succeed even if there are conflicts. If there are
conflicts in a file, then we use the merge base's version of that file,
and we produce a set of files with conflicts. The parents are both
Alice's and Bob's heads, and the data is the merge base plus Alice's
changed files plus Bob's changed files plus the successfully merged
files plus the merge base's version of the conflicting files.

`%auto` - meta-strategy. Check to see if Bob's desk exists, and if it
doesn't we use an `%init` merge. Otherwise, we progressively try
`%fine`, `%meet`, and `%mate` until one succeeds. If none succeed, we
merge Bob's desk into a scratch desk. Then, we merge Alice's desk into
the scratch desk with the `%meld` option to force the merge. Finally, we
annotate any conflicts, if we know how.

### Manipulation

<h3 class="first child">
<code>|rm path</code>
</h3>

Remove the data at `path`. `path` must be a path to actual node, not a
'directory'

### `|cp to from`

Copy the file at `from` into the path `to`.

#### `|mv to from`

Move the file at `from` into the path `to`.

In `%clay`, `|mv` is just a shorthand for `|cp` then `|rm`. The `|rm`
doesn't happen unless the `|cp` succeeds.

### Generators

<h3 class="first child">
<code>+cat path [path ...]</code>
</h3>

Similar to Unix `cat`. `+cat` takes one or more `path`s, and prints
their contents.

### `+ls path`

Similar to Unix `ls`. `+ls` takes a single `path`.

Produces a list of names at the `path`.
