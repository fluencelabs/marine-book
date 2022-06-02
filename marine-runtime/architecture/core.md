# Core

This section deep dives into the architecture of the Marine runtime component.&#x20;

### Interface-types

Marine uses Wasmer under the hood and this [crate](https://github.com/fluencelabs/interface-types) for interface types V1 implementation. By V1 we mean the interface-types proposal before November 2020:

![](<../../.gitbook/assets/Screenshot 2022-05-27 at 20.45.45.png>)

Each export and import function in a Marine Wasm module is wrapped by a corresponding import or export adapter. The purpose of an **import adapter** is to lift Wasm types to corresponding IT types, while the purpose of an **export adapter** is to lower IT types to corresponding Wasm types.

Adapters consist of IT instructions that are executed by a special interpreter. The full list of all supported instructions could be found [here](interface-types-instructions.md).

### Module linking

Let's consider a passing scheme between three modules in the [curl service](../../quickstart/develop-a-multi-modules-service.md):

![](<../../.gitbook/assets/Screenshot 2022-05-29 at 10.57.17.png>)

Here the `gen_n_save` function from the `facade` module calls the `download` function from the `curl` module and `put` from the `local_storage` module (more info about how these modules are structured and used could be found [here](../../quickstart/develop-a-multi-modules-service.md)).

For each module `Marine` creates corresponding Wasmer instance and for each import and export function, it creates a special interface-types adapter with an interpreter. In the picture below you could see the scheme with Wasmer instances for each module and adapters for each import and export linked together.

![](<../../.gitbook/assets/Screenshot 2022-05-27 at 20.46.35.png>)

### How do multi-module calls work

In this section, we're going to discuss what it costs to call a function from another module. Let's consider how it looks step-by-step when the `facade` module calls `download(url: String) -> String` exported from the `curl` module.

#### Step 1

This interop scheme starts with calling an adapter of the imported function `download`.

![](<../../.gitbook/assets/multi-module call, step 1.png>)

This part aims to lift a string (remember that `download` function receives one `String` as an argument) meaning that it constructs it from a pointer and a size provided by the Wasm part.

In the picture above the first two instructions push a pointer and a size to the operand stack, then `string.lift_memory` consumes them creating a resulted string. Then `call-core 1` calls `release_objects` that frees the memory occupied by the lifted string in a Wasm module. And, finally, the `curl.download` export adapter from the curl module is called.

Also note that a list of instructions each adapter consists on could be obtained with the Marine CLI, you could find the guide [here](../../marine-tooling-reference/marine-cli.md#it-shows-interface-types-of-the-wasm-binary).

#### Step 2

Then a prolog of the export adapter is being called. Its goal is vise-versa to the prolog of the import adapter - it should lower the passed string to make it possible to call the raw curl Wasm module with a pointer and a size.&#x20;

![](<../../.gitbook/assets/multi-module call, step 2.png>)

In this picture, the prolog starts with obtaining a string size by pushing it to the operand stack and calling `string.size`. Then it calls `allocate` exported from the `curl` module. Finally, it calls `string.lower_memory` that takes a string with a pointer to the allocated memory region and writes a string to this region.

#### Step 3

In the next step the "raw" `curl.download` is called with the pointer and the size produced on the previous step.&#x20;

![a](<../../.gitbook/assets/multi-module call, step 3.png>)

#### Step 4

The goal of 4th step is to lift the string produced by the `curl.download` function.

![](<../../.gitbook/assets/multi-module call, step 4.png>)

The code snippet above starts with retrieving a pointer and a size of the `download` function. It's done via two functions: [get\_result\_ptr](../../marine-rust-sdk/module-abi.md#get\_result\_ptr) and [get\_result\_size](../../marine-rust-sdk/module-abi.md#get\_result\_size). They are used because at the moment our interface-type implementation doesn't support multi-value. The last instruction is used to clean up this string from the `curl` module memory.

#### Step 5

The last step aims to lower string back to the `facade` module.

![](<../../.gitbook/assets/multi-module call, step 5.png>)

The lowering is done the similar to step 2 way: obtaining string size, calling `allocate`, and then `string.lower_memory`. The last two instructions call [set\_result\_size](../../marine-rust-sdk/module-abi.md#set\_result\_size) and [set\_result\_ptr](../../marine-rust-sdk/module-abi.md#set\_result\_ptr), which are needed for the current multi-value passing limitation.

And, finally, the control flow returns back to the `facade` Wasm module with the resulted string from the `curl.download` function.

The complete list of all IT instructions could be found in this [section](interface-types-instructions.md). Also, you could find additional info about miscellaneous functions in module ABI in this [section](../../marine-rust-sdk/module-abi.md).

### Performance

Despite the complex passing scheme between modules, Marine is a fast runtime with a lot of optimizations on different levels. The Marine REPL measures each operation and playing with Redis [compiled](https://medium.com/fluence-network/porting-redis-to-webassembly-with-clang-wasi-af99b264ca8) to Wasm one could see the following values:

![](<../../.gitbook/assets/marine performance.png>)
