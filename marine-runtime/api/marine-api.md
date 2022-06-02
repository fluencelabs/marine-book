# Marine API

FaaS layer is based on the runtime Marine component significantly expanding it by providing a few ways of its instantiation as well as calling with JSON values.

#### instantiation

{% tabs %}
{% tab title="with_raw_config" %}
```rust
fn with_raw_config<C>(config: C) -> FaaSResult<Self>
    where
        C: TryInto<FaaSConfig>,
        FaaSError: From<C::Error>,
```
{% endtab %}

{% tab title="with_modules" %}
```rust
fn with_modules<C>(mut modules: HashMap<String, Vec<u8>>, config: C) -> FaaSResult<Self>
    where
        C: TryInto<FaaSConfig>,
        FaaSError: From<C::Error>,
```
{% endtab %}

{% tab title="with_modules_names" %}
```rust
fn with_module_names<C>(names: &HashMap<String, String>, config: C) -> FaaSResult<Self>
    where
        C: TryInto<FaaSConfig>,
        FaaSError: From<C::Error>,
```
{% endtab %}
{% endtabs %}

Creates a FaaS instance from the provided config and modules or just module names.

#### calling a module

{% tabs %}
{% tab title="call_with_ivalues" %}
```rust
fn call_with_ivalues(
        &mut self,
        module_name: impl AsRef<str>,
        func_name: impl AsRef<str>,
        args: &[IValue],
        call_parameters: marine_rs_sdk::CallParameters,
    ) -> FaaSResult<Vec<IValue>>
```
{% endtab %}

{% tab title="call_with_json" %}
```rust
fn call_with_json(
        &mut self,
        module_name: impl AsRef<str>,
        func_name: impl AsRef<str>,
        json_args: JValue,
        call_parameters: marine_rs_sdk::CallParameters,
    ) -> FaaSResult<JValue>
```
{% endtab %}
{% endtabs %}

Invokes a function of a module inside Marine by given function name with given arguments. For more info about `IValue` take a look to [this](../ivalue-and-itype.md) chapter.

#### getting a module interface

{% tabs %}
{% tab title="get_interface" %}
```rust
fn get_interface(&self) -> FaaSInterface<'_>
```
{% endtab %}

{% tab title="FaaSInterface" %}
```rust
struct FaaSInterface<'a> {
    pub modules: HashMap<&'a str, FaaSModuleInterface<'a>>,
}

struct FaaSInterface<'a> {
    pub modules: HashMap<&'a str, FaaSModuleInterface<'a>>,
}

struct MModuleInterface<'a> {
    pub record_types: &'a MRecordTypes,
    pub function_signatures: Vec<MFunctionSignature>,
}

type MRecordTypes = HashMap<u64, Rc<IRecordType>>;

struct MFunctionSignature {
    pub name: Rc<String>,
    pub arguments: Rc<Vec<IFunctionArg>>,
    pub outputs: Rc<Vec<IType>>,
}
```
{% endtab %}
{% endtabs %}

This method returns a public interface of a module, i.e. a set of all public functions and records.

#### getting module memory stats

{% tabs %}
{% tab title="module_memory_stats" %}
```rust
fn module_memory_stats(&self) -> MemoryStats<'_>
```
{% endtab %}

{% tab title="MemoryStats" %}
```rust
struct MemoryStats<'module_name>(pub Vec<ModuleMemoryStat<'module_name>>);

/// Contains module name and a size of its linear memory in bytes.
/// Please note that linear memory contains not only heap, but globals, shadow stack and so on.
/// Although it doesn't contain operand stack, additional runtime (Wasmer) structures,
/// and some other stuff, that should be count separately.
struct ModuleMemoryStat<'module_name> {
    pub name: &'module_name str,
    pub memory_size: usize,
    // None if memory maximum wasn't set
    pub max_memory_size: Option<usize>,
}
```
{% endtab %}
{% endtabs %}

Returns a statistics of memory usage for all loaded modules.
