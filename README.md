# cervus

A WebAssembly subsystem for Linux.

![Screenshot](https://i.imgur.com/QFvUibQ.png)

## What's it?

Cervus implements a WebAssembly "usermode" on top of the Linux kernel (which tries to follow the [CommonWA](https://github.com/CommonWA/cwa-spec) specification), enabling wasm applications to run directly in ring 0, while still ensuring safety and security.

## But why?

- Managed execution (making it possible to perform optimizations based on tracing/partial evaluation)
- Avoid performance overhead introduced by system calls & address space switches
- Zero-copy I/O is possible

## Things that are working and not working

**Working:**

- An interpreter based on [HexagonE](https://github.com/losfair/hexagon-e)
- Binary translation & loading based on [wasm-core](https://github.com/losfair/wasm-core)
- Logging

**Not working:**

- Floating point
- JIT
- Everything else

## Build

### Kernel module

Requirements:

- xargo
- latest nightly rust
- kernel headers
- gnu make & gcc

```
xargo build --target x86_64-unknown-none-gnu --release
cd glue
./build.sh
sudo insmod cervus.ko
```

### Loader

This installs the `cvload` binary:

```
cd loader
cargo install
```

### Applications

`usermode/examples` contains a few examples for WebAssembly applications running in Cervus.

For example, to build and load the `hello_world` example:

```
cd usermode
cargo build --example hello_world --target wasm32-unknown-unknown --release
sudo cvload target/wasm32-unknown-unknown/release/examples/hello_world.wasm
```

And then run `dmesg | tail`. If everything works well, you should see something like:

```
[132382.502249] cervus: (20277) [INFO] Hello, world!
```

## Contribute

I'm busy with my College Entrance Examination until ~June 10, 2018, before which I cannot actively maintain this project. However, there are a few things that can be relatively easily worked on:

- A JIT based on Cretonne

Since Cretonne supports `no_std`, this should be relatively easy compared to other JIT approaches.

Interface with the rest of the system by implementing the `Backend` trait, for which the interpreter-based backend located in `src/backend/hexagon_e` is a good example to start with.

- File system API

Blocking filesystem I/O APIs can be added as virtual system calls, which locate in `src/env.rs`.

## License

Cervus itself has to use the GPL 2.0 license because it links to the Linux kernel. However, user code that runs on Cervus is not limited by this.
