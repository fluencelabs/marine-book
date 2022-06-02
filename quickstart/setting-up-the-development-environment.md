# Setting up the development environment

### Setting up

To build Marine modules you need to install a CLI tool called `marine` that uses the Rust `wasm32-wasi` target and Marine environment to compile Wasm modules.

First, install Rust and supplementary tools:

```shell
# install the Rust compiler and tools to `~/.cargo/bin`
 $ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
info: downloading installer
...
Rust is installed now. Great!
To configure your current shell run source $HOME/.cargo/env

# add Rust tools to the current PATH
~ $ source $HOME/.cargo/env
<no output>

# install the nightly toolchain
~ $ rustup install nightly
```

To be able to compile Rust in Wasm, install the `wasm32-wasi` compilation target:

```shell
# install wasm32-wasi target for WebAssembly
~ $ rustup target add wasm32-wasi
info: downloading component 'rust-std' for 'wasm32-wasi'
info: installing component 'rust-std' for 'wasm32-wasi'
```

To be able to use `generate` subcommand of marine, install the `cargo-generate` tool:

```shell
# install cargo-generate target for the marine tool
~ $ cargo install cargo-generate
...
Installed package `cargo-generate v...` (executable `cargo-generate`) 
```

Then, install `marine` and `mrepl`:

```shell
# install marine
~ $ cargo install marine

# install mrepl, it requires nightly toolchain
~ $ cargo +nightly install mrepl
```
