# Marine Tooling Reference

Fluence services are virtual constructs linking Wasm IT modules into addressable compute network services. We use the [Rust](https://www.rust-lang.org/) language to create code that compiles to the [wasm32-wasi](https://doc.rust-lang.org/stable/nightly-rustc/rustc\_target/spec/wasm32\_wasi/index.html) target with the help of Fluence's [Marine Rust SDK](https://github.com/fluencelabs/marine-rs-sdk/tree/master/crates/main).&#x20;

Since the output of our compiled rust code are `wasm` modules that eventually are distributed to one or more network peers, testing of the individual and linked _modules_ is critical before deployment. The Fluence Marine ecosystem equips developers with the:&#x20;

* [Marine REPL](https://github.com/fluencelabs/marine/tree/master/tools/repl), which wraps a local Marine runtime to allow developers to locally interact with Wasm module interfaces and business logic
* [Marine CLI](https://github.com/fluencelabs/marine/tree/master/tools/cli) allows developers to build Marine modules and examine various info from them.
