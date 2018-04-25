# cabtags

[hasktags](http://hackage.haskell.org/package/hasktags) is great, especially
for larger codebases where being able to quickly and easily jump around the
source is critical to speedy debugging and development.

[git submodules](https://git-scm.com/docs/git-submodule) are pretty nice for
large codebases that need to coordinate development on several projects.

Unfortunately, with very nested, interconnected submodule structures, it is
fairly common to have the same codebase living at two different places in your
directory tree, for example as a direct dependency submodule *and* as some
submodule's dependency submodule. There may also be codebases that don't
contribute to the build products, such as optional dependencies of submodules
with the top-level project does not use. In such a case, running hasktags on
the top-level project directory will list each definition in the shared
codebases multiple times, once for each location; and will include irrelevant
tags from the optional dependencies. This clutters tag searches, and is
potentially confusing if you don't notice that you are modifying the codebase
in the wrong location for the changes to get picked up.

Fortunately, such large codebases typically use one of [cabal
projects](https://www.haskell.org/cabal/users-guide/nix-local-build-overview.html)
or [stack](https://docs.haskellstack.org/en/stable/README/), and these tools
explicitly identify which subdirectories contain code to be built for the
top-level project. The existing (human-written) configurations for these tools
can therefore be used to disambiguate which of multiple copies of a codebase
should be considered "canonical".

`cabtags` is a terrible hack for parsing cabal project files and cabal package
files in shell and using the information in them to build this explicit list of
subdirectories to pass to hasktags.

## Usage

Install hasktags, then run `cabtags` from a directory that contains either a
cabal project (that is, at least one of a `cabal.project` or
`cabal.project.local` file) or a cabal file (that is, a file that matches the
glob `*.cabal`). It will pass a vaguely appropriate list of directories to
`hasktags -R`.

Currently it doesn't accept any command line flags or arguments; it's 100%
tailored to the way Daniel Wagner likes to work. That's intended to be an
expression of YAGNI and not a long-term property of the program; if you find it
too inflexible for your needs, let's talk about how to add more features and
flexibility.

## Bugs

It does terrible things with whitespace. It probably won't work in a path that
contains spaces.

It doesn't understand cabal's syntax for exciting strings (namely, Haskell
string syntax), so quoted or escaped strings in your hs-source-dirs won't be
treated right.

It doesn't actually use cabal's parser, so it's probably super buggy about
finding the right boundaries around fields. It seems to sort of work most of
the time on the project/cabal files I tried.

All of this could be addressed by using a real language and real parsers;
that's definitely on the table if it turns out that the existing way fails
often enough.

It doesn't yet understand stack configuration files. I'd love to hear from a
stack expert about what the right way to do this would be (and about what the
"quick and effective but probably a bit wrong" way would be, too).
