# jsrc Overview

`jsrc` is the C implementation of the J engine and its host-side front ends. The tree is mostly flat, so the easiest way to read it is by file family rather than by directories.

## Top-Level Shape

- Core runtime headers define the object model, interpreter state, portability layer, and macros.
- The parser/evaluator pipeline turns text into tokens, tokens into executable fragments, and fragments into J values.
- Primitive implementation files provide the built-in verbs, adverbs, conjunctions, numerics, arrays, and foreigns.
- Front-end bridge files expose the engine to the console, sockets, Java/JNI, and other hosts.
- Performance files provide SIMD, BLAS/GEMM, hashing, crypto, UTF conversion, and CPU-specific dispatch.

## Key Headers

### [j.h](../dev/c/jsource/jsrc/j.h)

Main global definitions header. This is the umbrella include for most engine code.

It contains:

- Platform/compiler compatibility logic
- SIMD and ISA feature setup
- Core types and macros
- Inclusion of the rest of the shared runtime headers

### [jt.h](../dev/c/jsource/jsrc/jt.h)

Defines the per-instance/per-thread runtime state (`jt` / `JST` / `JTTstruct`).

This is where the engine keeps:

- Parser state
- Symbol tables and locale state
- Memory pools and allocation bookkeeping
- Error and debug state
- RNG state
- Threadpool and task state
- Execution flags and recursion state

If you want to understand "what exists at runtime", this is the most important structural file.

### Other important headers

- [ja.h](../dev/c/jsource/jsrc/ja.h), [je.h](../dev/c/jsource/jsrc/je.h), [jtype.h](../dev/c/jsource/jsrc/jtype.h), [jerr.h](../dev/c/jsource/jsrc/jerr.h), [js.h](../dev/c/jsource/jsrc/js.h), [p.h](../dev/c/jsource/jsrc/p.h), [w.h](../dev/c/jsource/jsrc/w.h), [d.h](../dev/c/jsource/jsrc/d.h):
  supporting definitions for arrays, execution, types, errors, parsing, word formation, and diagnostics.
- [jlib.h](../dev/c/jsource/jsrc/jlib.h):
  public embedding API for front ends.
- [jeload.h](../dev/c/jsource/jsrc/jeload.h), [jfex.h](../dev/c/jsource/jsrc/jfex.h):
  support for loading/hosting the engine.

## Execution Pipeline

The main execution path is spread across a few families.

### `w*.c` and word formation

- [w.c](../dev/c/jsource/jsrc/w.c): lexical analysis and token formation.
- Converts a line of J text into internal words, handling names, numbers, strings, primitives, comments, and assignment forms.

This is the tokenizer layer.

### `p*.c` and parsing

- [p.c](../dev/c/jsource/jsrc/p.c): sentence parsing and execution.
- Uses the J parse table and reduction rules to recognize executable fragments and run them.

This is the parser/evaluator core.

### [cx.c](../dev/c/jsource/jsrc/cx.c) and explicit definitions

- Compiles explicit definitions (`:` forms) and executes sentences from them.
- Handles the tokenization and control-word encoding of the definition body, then drives line-by-line execution inside a defined verb, adverb, or conjunction.

### `d*.c` and diagnostics/debugging

- [d.c](../dev/c/jsource/jsrc/d.c): error formatting and stack display.
- [dc.c](../dev/c/jsource/jsrc/dc.c), [dss.c](../dev/c/jsource/jsrc/dss.c), [dstop.c](../dev/c/jsource/jsrc/dstop.c), [dsusp.c](../dev/c/jsource/jsrc/dsusp.c):
  debugger, stop/suspend, and diagnostic support.

This family makes failures intelligible and supports the debugger/runtime inspection model.

## Core Runtime Families

### [j.c](../dev/c/jsource/jsrc/j.c)

Global engine constants and permanent objects.

Examples:

- Small boxed and numeric constants
- Shared permanent arrays
- Version/build metadata
- Global capability flags

### `m*.c`

Memory management.

- [m.c](../dev/c/jsource/jsrc/m.c): allocator, virtual memory reservation/commit, pool management
- [mt.c](../dev/c/jsource/jsrc/mt.c): memory-related support utilities

This is a custom allocator-heavy runtime, not a malloc-only interpreter.

### `s*.c`

Symbol table, names, scripts, and related runtime services.

Representative files:

- [s.c](../dev/c/jsource/jsrc/s.c)
- [sl.c](../dev/c/jsource/jsrc/sl.c)
- [sn.c](../dev/c/jsource/jsrc/sn.c)
- [sc.c](../dev/c/jsource/jsrc/sc.c)

These files participate in name lookup, locale handling, script execution, and symbol management.

### [`t.c`](../dev/c/jsource/jsrc/t.c)
Dispatching table for all the primitives

## Language Primitive Families

The terse one-letter filenames mostly correspond to J language implementation areas.

### `v*.c`: verbs and array operations

This is one of the biggest families.

- [v.c](../dev/c/jsource/jsrc/v.c): core verbs
- [v0.c](../dev/c/jsource/jsrc/v0.c), [v1.c](../dev/c/jsource/jsrc/v1.c), [v2.c](../dev/c/jsource/jsrc/v2.c): verb support by valence/pattern
- [va1.c](../dev/c/jsource/jsrc/va1.c), [va2.c](../dev/c/jsource/jsrc/va2.c), [va1ss.c](../dev/c/jsource/jsrc/va1ss.c), [va2s.c](../dev/c/jsource/jsrc/va2s.c), [va2ss.c](../dev/c/jsource/jsrc/va2ss.c): atomic/vectorized numeric kernels
- [vb.c](../dev/c/jsource/jsrc/vb.c), [vd.c](../dev/c/jsource/jsrc/vd.c), [ve.c](../dev/c/jsource/jsrc/ve.c), [vf.c](../dev/c/jsource/jsrc/vf.c), [vi.c](../dev/c/jsource/jsrc/vi.c), [vm.c](../dev/c/jsource/jsrc/vm.c), [vo.c](../dev/c/jsource/jsrc/vo.c), [vp.c](../dev/c/jsource/jsrc/vp.c), [vq.c](../dev/c/jsource/jsrc/vq.c), [vs.c](../dev/c/jsource/jsrc/vs.c), [vt.c](../dev/c/jsource/jsrc/vt.c), [vu.c](../dev/c/jsource/jsrc/vu.c), [vx.c](../dev/c/jsource/jsrc/vx.c), [vz.c](../dev/c/jsource/jsrc/vz.c): specialized verb families
- [vcat.c](../dev/c/jsource/jsrc/vcat.c), [vcatsp.c](../dev/c/jsource/jsrc/vcatsp.c), [vfrom.c](../dev/c/jsource/jsrc/vfrom.c), [vfromsp.c](../dev/c/jsource/jsrc/vfromsp.c), [vrep.c](../dev/c/jsource/jsrc/vrep.c), [vrand.c](../dev/c/jsource/jsrc/vrand.c), [vfft.c](../dev/c/jsource/jsrc/vfft.c), [vgauss.c](../dev/c/jsource/jsrc/vgauss.c), [vgranking.c](../dev/c/jsource/jsrc/vgranking.c): concrete array/numeric operations

In practice, this family covers a huge amount of the built-in language.

### `a*.c`: adverbs and adjacent facilities

- [a.c](../dev/c/jsource/jsrc/a.c): adverbs
- [ab.c](../dev/c/jsource/jsrc/ab.c), [af.c](../dev/c/jsource/jsrc/af.c), [ai.c](../dev/c/jsource/jsrc/ai.c), [am.c](../dev/c/jsource/jsrc/am.c), [am1.c](../dev/c/jsource/jsrc/am1.c), [amn.c](../dev/c/jsource/jsrc/amn.c), [ao.c](../dev/c/jsource/jsrc/ao.c), [ap.c](../dev/c/jsource/jsrc/ap.c), [ar.c](../dev/c/jsource/jsrc/ar.c), [as.c](../dev/c/jsource/jsrc/as.c), [au.c](../dev/c/jsource/jsrc/au.c)

These they implement modifier behavior and related execution support.

### `c*.c`: conjunctions, foreign-call support, and core execution helpers

- [c.c](../dev/c/jsource/jsrc/c.c): conjunctions
- [ca.c](../dev/c/jsource/jsrc/ca.c), [cc.c](../dev/c/jsource/jsrc/cc.c), [cf.c](../dev/c/jsource/jsrc/cf.c), [cg.c](../dev/c/jsource/jsrc/cg.c), [ch.c](../dev/c/jsource/jsrc/ch.c), [cl.c](../dev/c/jsource/jsrc/cl.c), [cp.c](../dev/c/jsource/jsrc/cp.c), [ct.c](../dev/c/jsource/jsrc/ct.c), [cu.c](../dev/c/jsource/jsrc/cu.c), [cv.c](../dev/c/jsource/jsrc/cv.c), [cx.c](../dev/c/jsource/jsrc/cx.c)
- [cd.c](../dev/c/jsource/jsrc/cd.c): foreign-function/call interface support
- [cip.c](../dev/c/jsource/jsrc/cip.c), [cip_t.h](../dev/c/jsource/jsrc/cip_t.h), [cipfloatmm_t.h](../dev/c/jsource/jsrc/cipfloatmm_t.h): optimized comparison/indexing support structures

This family mixes conjunction semantics with other core engine facilities due to the age and evolution of the codebase.

### `f*.c`: formatting for display

- [f.c](../dev/c/jsource/jsrc/f.c)
- [f2.c](../dev/c/jsource/jsrc/f2.c)
- [fbu.c](../dev/c/jsource/jsrc/fbu.c)

These cover higher-level function behavior, derived-function logic, and some execution mechanics.

### `r*.c`: Primitives to produce displayed representations of entities. Representative files:

- [r.c](../dev/c/jsource/jsrc/r.c)
- [rl.c](../dev/c/jsource/jsrc/rl.c)
- [rt.c](../dev/c/jsource/jsrc/rt.c)

These are supporting engine components rather than one isolated feature area.

### `x*.c`: foreigns, external interfaces, and "extra" functionality

- [x.c](../dev/c/jsource/jsrc/x.c): the foreign subsystem
- [xa.c](../dev/c/jsource/jsrc/xa.c), [xb.c](../dev/c/jsource/jsrc/xb.c), [xc.c](../dev/c/jsource/jsrc/xc.c), [xd.c](../dev/c/jsource/jsrc/xd.c), [xf.c](../dev/c/jsource/jsrc/xf.c), [xh.c](../dev/c/jsource/jsrc/xh.c), [xi.c](../dev/c/jsource/jsrc/xi.c), [xl.c](../dev/c/jsource/jsrc/xl.c), [xlp.c](../dev/c/jsource/jsrc/xlp.c), [xo.c](../dev/c/jsource/jsrc/xo.c), [xs.c](../dev/c/jsource/jsrc/xs.c), [xt.c](../dev/c/jsource/jsrc/xt.c), [xu.c](../dev/c/jsource/jsrc/xu.c)
- [xcrc.c](../dev/c/jsource/jsrc/xcrc.c), [xsha.c](../dev/c/jsource/jsrc/xsha.c), [xaes.c](../dev/c/jsource/jsrc/xaes.c), [xdic.c](../dev/c/jsource/jsrc/xdic.c), [xfmt.c](../dev/c/jsource/jsrc/xfmt.c)

This family covers:

- File and OS services
- Locale/environment features
- Hashing/crypto helpers
- Dictionary/trie-like extras
- Formatting and conversion helpers
- Foreign verbs such as `m!:n` 

If something feels "outside the pure core language", it often lands here.

### DLL Support
[x15.c](../dev/c/jsource/jsrc/x15.c) - the DLL call driver. It assembles the argument stack frame for each foreign function call and dispatches execution into loaded shared libraries via switchcall.


## Front Ends And Hosting

### [io.c](../dev/c/jsource/jsrc/io.c)

The engine/front-end boundary.

This file documents the callback protocol very clearly:

- Front end provides input/output callbacks
- Front end calls `jdo()`
- Engine calls back for output, debug input, script lines, and host services

This is the main place to understand how JE and a host process interact.

### Console and examples

- [jconsole.c](../dev/c/jsource/jsrc/jconsole.c): interactive console front end
- [jfex.c](../dev/c/jsource/jsrc/jfex.c): minimal front-end example
- [jep.c](../dev/c/jsource/jsrc/jep.c): socket-based front end/server

### Dynamic loading and embedding

- [jeload.c](../dev/c/jsource/jsrc/jeload.c): load or initialize JE and wire callbacks
- [jlib.h](../dev/c/jsource/jsrc/jlib.h): exported C API

### Java/Android

- [andjnative.c](../dev/c/jsource/jsrc/andjnative.c): JNI bridge
- [com_jsoftware_j_JInterface.h](../dev/c/jsource/jsrc/com_jsoftware_j_JInterface.h), [jx_utils_jnative.h](../dev/c/jsource/jsrc/jx_utils_jnative.h): generated/support JNI headers

## Performance, SIMD, And CPU-Specific Code

This codebase spends a lot of effort on vectorization and tuned kernels.

### Vector and ISA-specific families

- [viavx.c](../dev/c/jsource/jsrc/viavx.c), [viavx1-2.c](../dev/c/jsource/jsrc/viavx1-2.c), [viavx1-4.c](../dev/c/jsource/jsrc/viavx1-4.c), [viavx2.c](../dev/c/jsource/jsrc/viavx2.c), [viavx3.c](../dev/c/jsource/jsrc/viavx3.c), [viavx4.c](../dev/c/jsource/jsrc/viavx4.c), [viavx5.c](../dev/c/jsource/jsrc/viavx5.c), [viavx6.c](../dev/c/jsource/jsrc/viavx6.c)
- [viix.c](../dev/c/jsource/jsrc/viix.c), [visp.c](../dev/c/jsource/jsrc/visp.c)

These provide AVX/AVX2/AVX-512 and related optimized paths.

### Sorting/merge/ranking helpers

- [vgsort.c](../dev/c/jsource/jsrc/vgsort.c), [vgsortiqavx512.c](../dev/c/jsource/jsrc/vgsortiqavx512.c)
- [vgsort.h](../dev/c/jsource/jsrc/vgsort.h), [vgsortq.h](../dev/c/jsource/jsrc/vgsortq.h), [vgsortavx512.h](../dev/c/jsource/jsrc/vgsortavx512.h), [vgsortinavx512.h](../dev/c/jsource/jsrc/vgsortinavx512.h), [vgsortiqavx512.h](../dev/c/jsource/jsrc/vgsortiqavx512.h), [vgmerge.h](../dev/c/jsource/jsrc/vgmerge.h), [vgmergemincomp.h](../dev/c/jsource/jsrc/vgmergemincomp.h), [vgcomp.c](../dev/c/jsource/jsrc/vgcomp.c)

### Linear algebra and GEMM

- [gemm.c](../dev/c/jsource/jsrc/gemm.c), [gemm.h](../dev/c/jsource/jsrc/gemm.h)
- [cblas.c](../dev/c/jsource/jsrc/cblas.c), [cblas.h](../dev/c/jsource/jsrc/cblas.h)
- `blis/` subtree

This is the matrix/BLAS performance layer.

### CPU feature detection

- [cpuinfo.c](../dev/c/jsource/jsrc/cpuinfo.c), [cpuinfo.h](../dev/c/jsource/jsrc/cpuinfo.h)

Used to select optimized kernels and tune behavior by architecture.

## Data Conversion, Unicode, Crypto

### Unicode/UTF

- `utf/` subtree

Contains optimized UTF-8 / UTF-16 conversion code, including AVX-512 implementations.

### Crypto and hashing

- [aes-arm.c](../dev/c/jsource/jsrc/aes-arm.c), [aes-c.c](../dev/c/jsource/jsrc/aes-c.c), [aes-sse2.c](../dev/c/jsource/jsrc/aes-sse2.c), [crc32c.c](../dev/c/jsource/jsrc/crc32c.c), [xsha.c](../dev/c/jsource/jsrc/xsha.c), [xcrc.c](../dev/c/jsource/jsrc/xcrc.c), [xaes.c](../dev/c/jsource/jsrc/xaes.c)
- `openssl/sha/` subtree

These support built-in hashing, checksums, and crypto-oriented verbs/utilities.

### Number/string conversion

- [dtoa.c](../dev/c/jsource/jsrc/dtoa.c), [dtoa.h](../dev/c/jsource/jsrc/dtoa.h)
- [str.c](../dev/c/jsource/jsrc/str.c), [strptime.c](../dev/c/jsource/jsrc/strptime.c)
- [xfmt.c](../dev/c/jsource/jsrc/xfmt.c)

These support formatting and text/numeric conversion work across the engine.

## Build And Deliverable Files

The makefiles show the major products.

- `Jconsole.mk`: console executable
- `Jamalgam.mk`: amalgamated/self-contained executable build
- `Jnative.mk`: JNI/shared-library bridge
- `Android.mk`, `Android-static.mk`: Android build variants
- `Tsdll.mk`: build test DLL for validating foreign-call behavior

## Small But Useful Special Files

- [tsdll.c](../dev/c/jsource/jsrc/tsdll.c):
  test shared library used to validate `cd`/foreign call behavior.
- [thread.c](../dev/c/jsource/jsrc/thread.c):
  portability shims for thread affinity on Apple platforms.
- [linenoise.c](../dev/c/jsource/jsrc/linenoise.c), [linenoise.h](../dev/c/jsource/jsrc/linenoise.h):
  command-line editing/history support for the console.
- `codingrules.txt`:
  local implementation conventions.

## Reading Order Recommendation

If you are new to this tree, a good order is:

1. [jlib.h](../dev/c/jsource/jsrc/jlib.h)
2. [jconsole.c](../dev/c/jsource/jsrc/jconsole.c)
3. [jeload.c](../dev/c/jsource/jsrc/jeload.c)
4. [io.c](../dev/c/jsource/jsrc/io.c)
5. [j.h](../dev/c/jsource/jsrc/j.h)
6. [jt.h](../dev/c/jsource/jsrc/jt.h)
7. [w.c](../dev/c/jsource/jsrc/w.c)
8. [p.c](../dev/c/jsource/jsrc/p.c)
9. [d.c](../dev/c/jsource/jsrc/d.c)
10. [v.c](../dev/c/jsource/jsrc/v.c), [a.c](../dev/c/jsource/jsrc/a.c), [c.c](../dev/c/jsource/jsrc/c.c), [x.c](../dev/c/jsource/jsrc/x.c)

That path gets you from host API to runtime state to tokenization/parsing to the language primitives.

## Bottom Line

`jsrc` is best understood as five overlapping subsystems:

- The engine core: object model, runtime state, memory, parser, evaluator
- The language implementation: verbs, adverbs, conjunctions, foreigns
- The host interface: console, embedding API, sockets, Java/JNI
- The performance layer: SIMD, BLAS/GEMM, CPU dispatch, tuned kernels
- The utility layer: Unicode, crypto, formatting, diagnostics, OS services

The tree looks flat, but it is a mature interpreter/runtime with a fairly clean conceptual architecture once you group files by family.

## Memory Management

Every J value — array, verb, noun, box — lives in an `A` block: a header (type, rank, shape, atom count, use-count, flags, pool index) followed immediately by the data. There is no separate heap object; the header and data are one contiguous allocation.

### Allocation

Blocks are created by the `GA`/`GATV`/`GAT0` family of macros, which call the allocator in [m.c](../dev/c/jsource/jsrc/m.c). Small blocks are served from per-thread free lists keyed by power-of-2 size class (`jt->mempool[]`). Large blocks go directly to the OS via `mmap` (POSIX) or `VirtualAlloc` (Windows). The engine never uses `malloc` for J values.

### Reference counting

Each block carries a use-count (`AC`). When a value is stored under a name or shared across a boundary, `ra(x)` increments the count. When the reference is dropped, `fa(x)` decrements it; if the count reaches zero, `jtfamf()` frees the block and recursively decrements any child blocks it contains (boxes, sparse arrays, verb bodies, etc.).

Permanent blocks — built-in primitives, small constants, shared empty arrays — have a special `ACISPERM` marker and are never freed.

### The tstack and PROLOG/EPILOG

Every engine function that allocates intermediate values opens a frame with `PROLOG`, which snapshots the current top of the per-thread tstack. Intermediate allocations are pushed onto the tstack as they are created. At the end of the function, `EPILOG(z)` calls `gc(z, _ttop)`, which protects the result block and frees every intermediate block pushed since the snapshot. This makes cleanup automatic and deterministic: there is no tracing GC and no stop-the-world pause.

### In-place execution

Blocks with the `ACINPLACE` flag in their use-count are owned by exactly one execution frame and may be modified in place rather than copied. The parser and evaluator track this flag carefully so that, for example, the result of one step can become the input buffer for the next without an allocation.

### Virtual blocks

A virtual block shares its data with a backing block (`ABACK`). It is used to represent slices and reshaped views without copying. Freeing a virtual block releases its reference to the backer; the backer is freed only when its own use-count reaches zero.

### Thread repatriation

Each thread has its own pool. When a block allocated on one thread is freed on another, it is placed in a repatriation queue (`repatq`) and returned to the owning thread's pool at the next opportunity, keeping the per-thread pools balanced without locking.

### Pool coalescing

The allocator tracks freed bytes per size class in `memballo[]`. When cumulative frees in a class exceed a threshold (`SBFREEB`, ~16 KB), the free list for that class is scanned and excess OS memory is released. This is the closest thing to a GC sweep in the system, but it operates per size-class on already-freed memory, not on live objects.
