# Release notes for the next version

Release Please generates the changelog from conventional commits. That record
is accurate but it does not say what a change means for someone who already has
the previous release installed. This file collects those consequences between
releases. Whoever publishes the draft release merges this file into the release
body and then resets the file to the template at the end.

## macOS: `DATA` object files are not interchangeable across this release

`DATA` statements are compiled into a descriptor whose layout was packed on
every target. On Mach-O that puts the descriptor's pointer at offset 2, and
`ld64` refuses a pointer relocation at an unaligned offset, fatally on arm64.
Darwin therefore uses the natural layout now, on the compiler and the runtime
side alike, and the pointer sits at offset 8.

The layout on Linux and Windows is unchanged. On macOS the descriptor is part
of the object-file interface between compiled code and the runtime, so an
object file or static library that carries a `DATA` statement and was produced
by an earlier `fbc` cannot be linked against this runtime, and the reverse
holds as well. Recompile such objects; there is no mixed configuration that
works. Programs compiled entirely with this release are unaffected, as is every
program without a `DATA` statement.

## macOS: shared libraries are named `.dylib`

`fbc -dylib foo.bas` produced `libfoo.so` on macOS and now produces
`libfoo.dylib`. Mach-O reserves `.so` for loadable bundles, which are built and
loaded differently, and `fb_DylibLoad()` looks for the `.dylib` name first.
Build scripts, installation rules and packaging manifests that name the `.so`
file have to follow. Code that loads the library through `DyLibLoad("foo")`
needs no change.

The library is linked with `-dynamiclib` and carries an install name, the
Mach-O equivalent of an ELF `SONAME`. Earlier releases passed the GNU spellings
`-shared` and `-h<soname>`, which the macOS toolchain rejects, so `-dylib` did
not link at all there.

## `-e` compiles where the C toolchain is clang

The C backend emits the jump to an error handler as a computed goto. Clang
rejects one in a function that takes no label address, which every procedure
holding an error check was without `-ex`. Programs built with `-e` therefore
failed to compile wherever clang translates the generated C: on macOS, and on
the `win32-aarch64` host, whose CLANGARM64 toolchain ships clang under the `gcc`
name. `-e` was not buildable on that host before this release.

A handler installed with `ON ERROR GOTO` and reached from another procedure
still reads its locals through the wrong frame pointer. That defect is
independent of this change and is tracked in
[issue #159](https://github.com/metaneutrons/freebasic-ng/issues/159).

---

Template for the next cycle:

```markdown
# Release notes for the next version

Release Please generates the changelog from conventional commits. That record
is accurate but it does not say what a change means for someone who already has
the previous release installed. This file collects those consequences between
releases. Whoever publishes the draft release merges this file into the release
body and then resets the file to the template at the end.

_No notes for the next release yet._
```
