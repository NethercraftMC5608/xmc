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
