# XMC

> Run modern Minecraft Java on platforms it was never designed for.

XMC is an experimental portable runtime, transformation toolchain, and
platform layer for running **modern Minecraft Java Edition** without relying
on a conventional desktop JVM.

The long-term goal is to execute the same modern Minecraft client across
radically different targets:

- Web browsers through WebAssembly and WebGPU
- Xbox 360
- PlayStation 3
- Modern consoles and other native targets
- Potentially any platform for which an XMC backend can be implemented

XMC is **not a reimplementation of Minecraft**.

Instead, it takes the real Minecraft Java bytecode and its dependencies,
transforms them into a form suitable for constrained targets, supplies the
Java runtime functionality they depend on, and executes the resulting
program through a platform-specific backend.

---

## Architecture

```text
                  Minecraft Java Edition
                           │
                           │ .class / bytecode
                           ▼
                  ┌──────────────────┐
                  │   XMC Toolchain  │
                  │                  │
                  │ Reachability     │
                  │ Bytecode lowering│
                  │ AOT preparation  │
                  │ Metadata packing │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │   XMC Runtime    │
                  │                  │
                  │ Java objects     │
                  │ GC               │
                  │ Exceptions       │
                  │ Reflection       │
                  │ Class metadata   │
                  │ Java classlib    │
                  │ Threads / sync   │
                  └────────┬─────────┘
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
          Browser       Xbox 360       Future
        WASM/WebGPU     PPC/Xenos      Backends
```

The browser is currently the primary validation target.

Once Minecraft can run correctly through the shared XMC environment in the
browser, most of the Java compatibility work can be reused by native console
backends.

---

## Why?

Minecraft Java normally assumes the presence of a large modern Java runtime.

That means far more than simply executing bytecode.

Minecraft and its libraries expect functionality such as:

- garbage collection
- exceptions
- reflection
- annotations
- class metadata
- collections
- streams
- concurrency
- `java.time`
- NIO
- XML
- resource loading
- service discovery
- logging
- networking
- native/platform APIs

WebAssembly, an Xbox 360, or a PlayStation 3 does not provide any of that
automatically.

XMC provides the missing execution environment while preserving as much of
the original Minecraft program as possible.

---

## Not a browser Minecraft rewrite

There are already browser Minecraft projects which reimplement large parts of
the client in JavaScript or adapt older Minecraft versions specifically for
the web.

XMC takes a different approach.

```text
Traditional browser client:

Minecraft protocol
      ↓
reimplemented client
      ↓
JavaScript renderer
      ↓
browser
```

XMC:

```text
real Minecraft Java classes
          ↓
XMC transformation/runtime
          ↓
WebAssembly
          ↓
WebGPU
```

This makes XMC considerably more complicated, but also makes the runtime
architecture portable beyond the browser.

---

## Current status

XMC is under active development and is **not yet a playable release**.

The current runtime is already capable of executing substantial portions of
the real modern Minecraft dependency graph.

Implemented or substantially working areas include:

- Java classfile parsing and transformation
- `invokedynamic` elimination
- lambda lowering
- closed-world reachability analysis
- packed program images
- objects and arrays
- garbage collection
- exceptions
- virtual/interface dispatch
- class initialization
- reflection
- annotations
- `Method.invoke`
- constructor reflection
- class metadata
- `ServiceLoader`
- core collections
- streams
- portions of `java.time`
- IO and NIO
- charset support
- XML
- resource URLs
- logging bootstrap
- browser networking infrastructure
- WebGPU rendering infrastructure

The real Minecraft bootstrap currently reaches deep into Log4j configuration
and plugin construction using XMC's reflection implementation.

Development is currently focused on completing the remaining Java runtime
surface and runtime mechanisms required to progress further into Minecraft's
startup.

---

## Java class library

One of the largest challenges is providing enough of the Java standard
library for modern Minecraft.

XMC uses several strategies:

```text
Existing XMC implementation
          │
          ├── use directly
          │
Pinned JDK implementation
          ├── import when safe
          │
TeaVM classlib
          ├── use as an implementation donor
          │
          ▼
XMC-specific implementation
```

TeaVM is used as a **source donor**, not as XMC's runtime.

All imported or adapted behavior is validated against a pinned JDK version.

```text
TeaVM = implementation source
JDK    = behavioral reference
XMC    = final runtime
```

This allows large groups of ordinary Java library functionality to be
implemented in batches instead of manually recreating every method.

---

## Closed-world analysis

XMC does not attempt to implement the entirety of Java SE.

Instead, it determines the Java surface actually reachable from Minecraft and
its dependencies.

The surface analysis tracks:

- unresolved symbols
- static call sites
- missing XMC members
- safe JDK imports
- TeaVM donor candidates
- reflection roots
- runtime-bound functionality
- subsystem ownership

This lets development focus on the subset of Java that Minecraft actually
requires.

---

## Multi-agent development

XMC is large enough that development is divided into independent subsystem
workstreams.

Examples include:

- collections
- general JCL compatibility
- streams
- time/date
- NIO
- concurrency
- reflection
- runtime internals
- JDK differential testing
- surface-analysis tooling

Each workstream uses isolated Git worktrees and produces independently tested
commits which are integrated into the authoritative runtime.

The authoritative Minecraft boot is used as an integration checkpoint rather
than as a scheduler for every individual missing method.

---

## Browser target

The browser target is intended to use:

- WebAssembly for execution
- WebGPU for rendering
- browser-native input
- browser networking bridges
- browser storage/platform services

The browser build also serves as the fastest environment for validating the
shared XMC runtime.

---

## Xbox 360 target

The Xbox 360 is one of XMC's primary long-term targets.

Target hardware:

```text
CPU:    Xenon PowerPC
GPU:    Xenos
Memory: 512 MiB unified
```

The Xbox backend requires:

- PowerPC AOT code generation
- Xenos rendering
- strict memory management
- console-specific input/audio/networking
- aggressive runtime and asset memory control

Running a modern Minecraft Java client on hardware from 2005 is intentionally
an extreme test of XMC's portability.

---

## Future targets

The architecture is intended to allow additional backends without rebuilding
the Java compatibility layer from scratch.

Potential targets include:

- PlayStation 3
- Nintendo Switch
- PlayStation 5
- native desktop targets
- other embedded or unconventional systems

These are future goals and are not currently supported releases.

---

## Design principles

### Preserve the original program

Prefer executing real Minecraft code over reimplementing game behavior.

### Share as much as possible

Java compatibility should live in the common runtime rather than being
rewritten for each platform.

### Fail loudly

Unsupported behavior should produce an explicit runtime capability failure,
not silently return fake values.

### Validate against the JDK

Passing compilation is not enough.

Where possible, XMC behavior is compared directly against a reference JDK.

### Optimize after correctness

The browser interpreter/runtime is useful for bringing the program up and
finding compatibility issues.

Performance-critical code can later move toward direct AOT compilation.

---

## Project philosophy

XMC started with a simple question:

> What would it take to run modern Minecraft Java on hardware and platforms
> that were never supposed to run it?

The answer turned out to involve considerably more than translating a game.

It requires building enough of the environment around Java itself to make a
large modern application portable.

XMC is an exploration of that problem.

---

## Status warning

XMC is experimental research software.

It is currently intended for development, reverse engineering, runtime
research, and experimentation.

Expect:

- incomplete Java APIs
- runtime assertions
- missing platform functionality
- performance issues
- breaking changes
- unfinished console backends

There is currently no stable end-user release.

---

## Legal

XMC does not distribute Minecraft game assets or proprietary Minecraft source
code.

Users are responsible for supplying any required game files through legally
obtained copies of Minecraft.

Minecraft is a trademark of Microsoft/Mojang Studios.

XMC is an independent project and is not affiliated with or endorsed by
Microsoft or Mojang Studios.
