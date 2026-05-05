# jsrc Overview

`jsrc` is the C implementation of the J engine and its host-side front ends. The tree is mostly flat, so the easiest way to read it is by file family rather than by directories.

## Top-Level Shape

- Core runtime headers define the object model, interpreter state, portability layer, and macros.
- The parser/evaluator pipeline turns text into tokens, tokens into executable fragments, and fragments into J values.
- Primitive implementation files provide the built-in verbs, adverbs, conjunctions, numerics, arrays, and foreigns.
- Front-end bridge files expose the engine to the console, sockets, Java/JNI, and other hosts.
- Performance files provide SIMD, BLAS/GEMM, hashing, crypto, UTF conversion, and CPU-specific dispatch.

## Key Headers

### [j.h](https://github.com/jsoftware/jsource/blob/master/jsrc/j.h)

Main global definitions header. This is the umbrella include for most engine code.

It contains:

- Platform/compiler compatibility logic
- SIMD and ISA feature setup
- Core types and macros
- Inclusion of the rest of the shared runtime headers

### [jt.h](https://github.com/jsoftware/jsource/blob/master/jsrc/jt.h)

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

- [ja.h](https://github.com/jsoftware/jsource/blob/master/jsrc/ja.h), [je.h](https://github.com/jsoftware/jsource/blob/master/jsrc/je.h), [jtype.h](https://github.com/jsoftware/jsource/blob/master/jsrc/jtype.h), [jerr.h](https://github.com/jsoftware/jsource/blob/master/jsrc/jerr.h), [js.h](https://github.com/jsoftware/jsource/blob/master/jsrc/js.h), [p.h](https://github.com/jsoftware/jsource/blob/master/jsrc/p.h), [w.h](https://github.com/jsoftware/jsource/blob/master/jsrc/w.h), [d.h](https://github.com/jsoftware/jsource/blob/master/jsrc/d.h):
  supporting definitions for arrays, execution, types, errors, parsing, word formation, and diagnostics.
- [jlib.h](https://github.com/jsoftware/jsource/blob/master/jsrc/jlib.h):
  public embedding API for front ends.
- [jeload.h](https://github.com/jsoftware/jsource/blob/master/jsrc/jeload.h), [jfex.h](https://github.com/jsoftware/jsource/blob/master/jsrc/jfex.h):
  support for loading/hosting the engine.

## Execution Pipeline

The main execution path is spread across a few families.

### `w*.c` and word formation

- [w.c](https://github.com/jsoftware/jsource/blob/master/jsrc/w.c): lexical analysis and token formation.
- Converts a line of J text into internal words, handling names, numbers, strings, primitives, comments, and assignment forms.

This is the tokenizer layer.

### `p*.c` and parsing

- [p.c](https://github.com/jsoftware/jsource/blob/master/jsrc/p.c): sentence parsing and execution.
- Uses the J parse table and reduction rules to recognize executable fragments and run them.

This is the parser/evaluator core.

### [cx.c](https://github.com/jsoftware/jsource/blob/master/jsrc/cx.c) and explicit definitions

- Compiles explicit definitions (`:` forms) and executes sentences from them.
- Handles the tokenization and control-word encoding of the definition body, then drives line-by-line execution inside a defined verb, adverb, or conjunction.

### `d*.c` and diagnostics/debugging

- [d.c](https://github.com/jsoftware/jsource/blob/master/jsrc/d.c): error formatting and stack display.
- [dc.c](https://github.com/jsoftware/jsource/blob/master/jsrc/dc.c), [dss.c](https://github.com/jsoftware/jsource/blob/master/jsrc/dss.c), [dstop.c](https://github.com/jsoftware/jsource/blob/master/jsrc/dstop.c), [dsusp.c](https://github.com/jsoftware/jsource/blob/master/jsrc/dsusp.c):
  debugger, stop/suspend, and diagnostic support.

This family makes failures intelligible and supports the debugger/runtime inspection model.

## Core Runtime Families

### [j.c](https://github.com/jsoftware/jsource/blob/master/jsrc/j.c)

Global engine constants and permanent objects.

Examples:

- Small boxed and numeric constants
- Shared permanent arrays
- Version/build metadata
- Global capability flags

### `m*.c`

Memory management.

- [m.c](https://github.com/jsoftware/jsource/blob/master/jsrc/m.c): allocator, virtual memory reservation/commit, pool management
- [mt.c](https://github.com/jsoftware/jsource/blob/master/jsrc/mt.c): memory-related support utilities

This is a custom allocator-heavy runtime, not a malloc-only interpreter.

### `s*.c`

Symbol table, names, scripts, and related runtime services.

Representative files:

- [s.c](https://github.com/jsoftware/jsource/blob/master/jsrc/s.c)
- [sl.c](https://github.com/jsoftware/jsource/blob/master/jsrc/sl.c)
- [sn.c](https://github.com/jsoftware/jsource/blob/master/jsrc/sn.c)
- [sc.c](https://github.com/jsoftware/jsource/blob/master/jsrc/sc.c)

These files participate in name lookup, locale handling, script execution, and symbol management.

### [`t.c`](https://github.com/jsoftware/jsource/blob/master/jsrc/t.c)
Dispatching table for all the primitives

## Language Primitive Families

The terse one-letter filenames mostly correspond to J language implementation areas.

### `v*.c`: verbs and array operations

This is one of the biggest families.

- [v.c](https://github.com/jsoftware/jsource/blob/master/jsrc/v.c): core verbs
- [v0.c](https://github.com/jsoftware/jsource/blob/master/jsrc/v0.c), [v1.c](https://github.com/jsoftware/jsource/blob/master/jsrc/v1.c), [v2.c](https://github.com/jsoftware/jsource/blob/master/jsrc/v2.c): verb support by valence/pattern
- [va1.c](https://github.com/jsoftware/jsource/blob/master/jsrc/va1.c), [va2.c](https://github.com/jsoftware/jsource/blob/master/jsrc/va2.c), [va1ss.c](https://github.com/jsoftware/jsource/blob/master/jsrc/va1ss.c), [va2s.c](https://github.com/jsoftware/jsource/blob/master/jsrc/va2s.c), [va2ss.c](https://github.com/jsoftware/jsource/blob/master/jsrc/va2ss.c): atomic/vectorized numeric kernels
- [vb.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vb.c), [vd.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vd.c), [ve.c](https://github.com/jsoftware/jsource/blob/master/jsrc/ve.c), [vf.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vf.c), [vi.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vi.c), [vm.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vm.c), [vo.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vo.c), [vp.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vp.c), [vq.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vq.c), [vs.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vs.c), [vt.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vt.c), [vu.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vu.c), [vx.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vx.c), [vz.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vz.c): specialized verb families
- [vcat.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vcat.c), [vcatsp.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vcatsp.c), [vfrom.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vfrom.c), [vfromsp.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vfromsp.c), [vrep.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vrep.c), [vrand.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vrand.c), [vfft.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vfft.c), [vgauss.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vgauss.c), [vgranking.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vgranking.c): concrete array/numeric operations

In practice, this family covers a huge amount of the built-in language.

### `a*.c`: adverbs and adjacent facilities

- [a.c](https://github.com/jsoftware/jsource/blob/master/jsrc/a.c): adverbs
- [ab.c](https://github.com/jsoftware/jsource/blob/master/jsrc/ab.c), [af.c](https://github.com/jsoftware/jsource/blob/master/jsrc/af.c), [ai.c](https://github.com/jsoftware/jsource/blob/master/jsrc/ai.c), [am.c](https://github.com/jsoftware/jsource/blob/master/jsrc/am.c), [am1.c](https://github.com/jsoftware/jsource/blob/master/jsrc/am1.c), [amn.c](https://github.com/jsoftware/jsource/blob/master/jsrc/amn.c), [ao.c](https://github.com/jsoftware/jsource/blob/master/jsrc/ao.c), [ap.c](https://github.com/jsoftware/jsource/blob/master/jsrc/ap.c), [ar.c](https://github.com/jsoftware/jsource/blob/master/jsrc/ar.c), [as.c](https://github.com/jsoftware/jsource/blob/master/jsrc/as.c), [au.c](https://github.com/jsoftware/jsource/blob/master/jsrc/au.c)

These they implement modifier behavior and related execution support.

### `c*.c`: conjunctions, foreign-call support, and core execution helpers

- [c.c](https://github.com/jsoftware/jsource/blob/master/jsrc/c.c): conjunctions
- [ca.c](https://github.com/jsoftware/jsource/blob/master/jsrc/ca.c), [cc.c](https://github.com/jsoftware/jsource/blob/master/jsrc/cc.c), [cf.c](https://github.com/jsoftware/jsource/blob/master/jsrc/cf.c), [cg.c](https://github.com/jsoftware/jsource/blob/master/jsrc/cg.c), [ch.c](https://github.com/jsoftware/jsource/blob/master/jsrc/ch.c), [cl.c](https://github.com/jsoftware/jsource/blob/master/jsrc/cl.c), [cp.c](https://github.com/jsoftware/jsource/blob/master/jsrc/cp.c), [ct.c](https://github.com/jsoftware/jsource/blob/master/jsrc/ct.c), [cu.c](https://github.com/jsoftware/jsource/blob/master/jsrc/cu.c), [cv.c](https://github.com/jsoftware/jsource/blob/master/jsrc/cv.c), [cx.c](https://github.com/jsoftware/jsource/blob/master/jsrc/cx.c)
- [cd.c](https://github.com/jsoftware/jsource/blob/master/jsrc/cd.c): foreign-function/call interface support
- [cip.c](https://github.com/jsoftware/jsource/blob/master/jsrc/cip.c), [cip_t.h](https://github.com/jsoftware/jsource/blob/master/jsrc/cip_t.h), [cipfloatmm_t.h](https://github.com/jsoftware/jsource/blob/master/jsrc/cipfloatmm_t.h): optimized comparison/indexing support structures

This family mixes conjunction semantics with other core engine facilities due to the age and evolution of the codebase.

### `f*.c`: formatting for display

- [f.c](https://github.com/jsoftware/jsource/blob/master/jsrc/f.c)
- [f2.c](https://github.com/jsoftware/jsource/blob/master/jsrc/f2.c)
- [fbu.c](https://github.com/jsoftware/jsource/blob/master/jsrc/fbu.c)

These cover higher-level function behavior, derived-function logic, and some execution mechanics.

### `r*.c`: Primitives to produce displayed representations of entities. Representative files:

- [r.c](https://github.com/jsoftware/jsource/blob/master/jsrc/r.c)
- [rl.c](https://github.com/jsoftware/jsource/blob/master/jsrc/rl.c)
- [rt.c](https://github.com/jsoftware/jsource/blob/master/jsrc/rt.c)

These are supporting engine components rather than one isolated feature area.

### `x*.c`: foreigns, external interfaces, and "extra" functionality

- [x.c](https://github.com/jsoftware/jsource/blob/master/jsrc/x.c): the foreign subsystem
- [xa.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xa.c), [xb.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xb.c), [xc.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xc.c), [xd.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xd.c), [xf.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xf.c), [xh.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xh.c), [xi.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xi.c), [xl.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xl.c), [xlp.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xlp.c), [xo.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xo.c), [xs.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xs.c), [xt.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xt.c), [xu.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xu.c)
- [xcrc.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xcrc.c), [xsha.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xsha.c), [xaes.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xaes.c), [xdic.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xdic.c), [xfmt.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xfmt.c)

This family covers:

- File and OS services
- Locale/environment features
- Hashing/crypto helpers
- Dictionary/trie-like extras
- Formatting and conversion helpers
- Foreign verbs such as `m!:n` 

If something feels "outside the pure core language", it often lands here.

### DLL Support
[x15.c](https://github.com/jsoftware/jsource/blob/master/jsrc/x15.c) - the DLL call driver. It assembles the argument stack frame for each foreign function call and dispatches execution into loaded shared libraries via switchcall.


## Front Ends And Hosting

### [io.c](https://github.com/jsoftware/jsource/blob/master/jsrc/io.c)

The engine/front-end boundary.

This file documents the callback protocol very clearly:

- Front end provides input/output callbacks
- Front end calls `jdo()`
- Engine calls back for output, debug input, script lines, and host services

This is the main place to understand how JE and a host process interact.

### Console and examples

- [jconsole.c](https://github.com/jsoftware/jsource/blob/master/jsrc/jconsole.c): interactive console front end
- [jfex.c](https://github.com/jsoftware/jsource/blob/master/jsrc/jfex.c): minimal front-end example
- [jep.c](https://github.com/jsoftware/jsource/blob/master/jsrc/jep.c): socket-based front end/server

### Dynamic loading and embedding

- [jeload.c](https://github.com/jsoftware/jsource/blob/master/jsrc/jeload.c): load or initialize JE and wire callbacks
- [jlib.h](https://github.com/jsoftware/jsource/blob/master/jsrc/jlib.h): exported C API

### Java/Android

- [andjnative.c](https://github.com/jsoftware/jsource/blob/master/jsrc/andjnative.c): JNI bridge
- [com_jsoftware_j_JInterface.h](https://github.com/jsoftware/jsource/blob/master/jsrc/com_jsoftware_j_JInterface.h), [jx_utils_jnative.h](https://github.com/jsoftware/jsource/blob/master/jsrc/jx_utils_jnative.h): generated/support JNI headers

## Performance, SIMD, And CPU-Specific Code

This codebase spends a lot of effort on vectorization and tuned kernels.

### Vector and ISA-specific families

- [viavx.c](https://github.com/jsoftware/jsource/blob/master/jsrc/viavx.c), [viavx1-2.c](https://github.com/jsoftware/jsource/blob/master/jsrc/viavx1-2.c), [viavx1-4.c](https://github.com/jsoftware/jsource/blob/master/jsrc/viavx1-4.c), [viavx2.c](https://github.com/jsoftware/jsource/blob/master/jsrc/viavx2.c), [viavx3.c](https://github.com/jsoftware/jsource/blob/master/jsrc/viavx3.c), [viavx4.c](https://github.com/jsoftware/jsource/blob/master/jsrc/viavx4.c), [viavx5.c](https://github.com/jsoftware/jsource/blob/master/jsrc/viavx5.c), [viavx6.c](https://github.com/jsoftware/jsource/blob/master/jsrc/viavx6.c)
- [viix.c](https://github.com/jsoftware/jsource/blob/master/jsrc/viix.c), [visp.c](https://github.com/jsoftware/jsource/blob/master/jsrc/visp.c)

These provide AVX/AVX2/AVX-512 and related optimized paths.

### Sorting/merge/ranking helpers

- [vgsort.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vgsort.c), [vgsortiqavx512.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vgsortiqavx512.c)
- [vgsort.h](https://github.com/jsoftware/jsource/blob/master/jsrc/vgsort.h), [vgsortq.h](https://github.com/jsoftware/jsource/blob/master/jsrc/vgsortq.h), [vgsortavx512.h](https://github.com/jsoftware/jsource/blob/master/jsrc/vgsortavx512.h), [vgsortinavx512.h](https://github.com/jsoftware/jsource/blob/master/jsrc/vgsortinavx512.h), [vgsortiqavx512.h](https://github.com/jsoftware/jsource/blob/master/jsrc/vgsortiqavx512.h), [vgmerge.h](https://github.com/jsoftware/jsource/blob/master/jsrc/vgmerge.h), [vgmergemincomp.h](https://github.com/jsoftware/jsource/blob/master/jsrc/vgmergemincomp.h), [vgcomp.c](https://github.com/jsoftware/jsource/blob/master/jsrc/vgcomp.c)

### Linear algebra and GEMM

- [gemm.c](https://github.com/jsoftware/jsource/blob/master/jsrc/gemm.c), [gemm.h](https://github.com/jsoftware/jsource/blob/master/jsrc/gemm.h)
- [cblas.c](https://github.com/jsoftware/jsource/blob/master/jsrc/cblas.c), [cblas.h](https://github.com/jsoftware/jsource/blob/master/jsrc/cblas.h)
- `blis/` subtree

This is the matrix/BLAS performance layer.

### CPU feature detection

- [cpuinfo.c](https://github.com/jsoftware/jsource/blob/master/jsrc/cpuinfo.c), [cpuinfo.h](https://github.com/jsoftware/jsource/blob/master/jsrc/cpuinfo.h)

Used to select optimized kernels and tune behavior by architecture.

## Data Conversion, Unicode, Crypto

### Unicode/UTF

- `utf/` subtree

Contains optimized UTF-8 / UTF-16 conversion code, including AVX-512 implementations.

### Crypto and hashing

- [aes-arm.c](https://github.com/jsoftware/jsource/blob/master/jsrc/aes-arm.c), [aes-c.c](https://github.com/jsoftware/jsource/blob/master/jsrc/aes-c.c), [aes-sse2.c](https://github.com/jsoftware/jsource/blob/master/jsrc/aes-sse2.c), [crc32c.c](https://github.com/jsoftware/jsource/blob/master/jsrc/crc32c.c), [xsha.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xsha.c), [xcrc.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xcrc.c), [xaes.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xaes.c)
- `openssl/sha/` subtree

These support built-in hashing, checksums, and crypto-oriented verbs/utilities.

### Number/string conversion

- [dtoa.c](https://github.com/jsoftware/jsource/blob/master/jsrc/dtoa.c), [dtoa.h](https://github.com/jsoftware/jsource/blob/master/jsrc/dtoa.h)
- [str.c](https://github.com/jsoftware/jsource/blob/master/jsrc/str.c), [strptime.c](https://github.com/jsoftware/jsource/blob/master/jsrc/strptime.c)
- [xfmt.c](https://github.com/jsoftware/jsource/blob/master/jsrc/xfmt.c)

These support formatting and text/numeric conversion work across the engine.

## Build And Deliverable Files

The makefiles show the major products.

- `Jconsole.mk`: console executable
- `Jamalgam.mk`: amalgamated/self-contained executable build
- `Jnative.mk`: JNI/shared-library bridge
- `Android.mk`, `Android-static.mk`: Android build variants
- `Tsdll.mk`: build test DLL for validating foreign-call behavior

## Small But Useful Special Files

- [tsdll.c](https://github.com/jsoftware/jsource/blob/master/jsrc/tsdll.c):
  test shared library used to validate `cd`/foreign call behavior.
- [thread.c](https://github.com/jsoftware/jsource/blob/master/jsrc/thread.c):
  portability shims for thread affinity on Apple platforms.
- [linenoise.c](https://github.com/jsoftware/jsource/blob/master/jsrc/linenoise.c), [linenoise.h](https://github.com/jsoftware/jsource/blob/master/jsrc/linenoise.h):
  command-line editing/history support for the console.
- `codingrules.txt`:
  local implementation conventions.

## Reading Order Recommendation

If you are new to this tree, a good order is:

1. [jlib.h](https://github.com/jsoftware/jsource/blob/master/jsrc/jlib.h)
2. [jconsole.c](https://github.com/jsoftware/jsource/blob/master/jsrc/jconsole.c)
3. [jeload.c](https://github.com/jsoftware/jsource/blob/master/jsrc/jeload.c)
4. [io.c](https://github.com/jsoftware/jsource/blob/master/jsrc/io.c)
5. [j.h](https://github.com/jsoftware/jsource/blob/master/jsrc/j.h)
6. [jt.h](https://github.com/jsoftware/jsource/blob/master/jsrc/jt.h)
7. [w.c](https://github.com/jsoftware/jsource/blob/master/jsrc/w.c)
8. [p.c](https://github.com/jsoftware/jsource/blob/master/jsrc/p.c)
9. [d.c](https://github.com/jsoftware/jsource/blob/master/jsrc/d.c)
10. [v.c](https://github.com/jsoftware/jsource/blob/master/jsrc/v.c), [a.c](https://github.com/jsoftware/jsource/blob/master/jsrc/a.c), [c.c](https://github.com/jsoftware/jsource/blob/master/jsrc/c.c), [x.c](https://github.com/jsoftware/jsource/blob/master/jsrc/x.c)

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

Blocks are created by the `GA`/`GATV`/`GAT0` family of macros, which call the allocator in [m.c](https://github.com/jsoftware/jsource/blob/master/jsrc/m.c). Small blocks are served from per-thread free lists keyed by power-of-2 size class (`jt->mempool[]`). Large blocks go directly to the OS via `mmap` (POSIX) or `VirtualAlloc` (Windows). The engine never uses `malloc` for J values.

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
