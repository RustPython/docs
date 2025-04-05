# Calling Rust from Python
## Structure
```rust, ignore
use rustpython::vm::pymodule;
#[pymodule]
mod test_module {
    #[pyattr]
    pub const THE_ANSWER: i32 = 42;

    #[pyfunction]
    pub fn add(a: i32, b: i32) -> i32 {
        a + b
    }

    #[pyattr]
    #[pyclass]
    pub struct TestClass {
        pub value: i32,
    }

    #[pyclass]
    impl TestClass {
        #[pygetset]
        pub fn value(&self) -> i32 {
            self.value
        }

        #[pymethod]
        pub fn get_info(&self) -> i32 {
            self.value * 2
        }
    }
}
```
This code defines a Python module named `test_module` with two items:
a constant named `THE_ANSWER` and a function named `add`.
The `#[pymodule]` attribute is used to mark the module,
and the `#[pyattr]` and `#[pyfunction]` attributes are used to mark the constant and function, respectively.

RustPython allows for 3 types of items in a module:
- Variables: Defined using the `#[pyattr]` attribute.
- Functions: Defined using the `#[pyfunction]` attribute.
- Classes: Defined using the `#[pyclass]` attribute.

## General Configuration
Most attributes have a `name` parameter that can be used to specify the name of the item in Python.
If the `name` parameter is not provided, the Rust identifier is used as the name in Python.

## Variables
Variables are defined using the `#[pyattr]` attribute.
A variable can either be a constant or a function.
Note that classes are treated as attributes in RustPython
and are annotated with `#[pyattr]` as well, but that can be done away with if needed.
```rust, no_run
#[pyattr]
const THE_ANSWER: i32 = 42;
// ... or
#[pyattr]
fn the_answer() -> i32 {
    42
}
// ... or
// this will cache the result of the function
// and return the same value every time it is called
#[pyattr(once)]
fn cached_answer() -> i32 {
    42
}
```

## Valid Arguments/Return Types
Every input and return value must be convertible to `PyResult`. This is defined as `IntoPyResult` trait. So any return value of them must implement `IntoPyResult`. It will be `PyResult<PyObjectRef>`, `PyObjectRef` and any `PyResult<T>` when T implements `IntoPyObject`. Practically we can list them like:
- Any `T` when `PyResult<T>` is possible
- `PyObjectRef`
- `PyResult<()>` and `()` as `None`
- `PyRef<T: PyValue>` like `PyIntRef`, `PyStrRef`
- `T: PyValue` like `PyInt`, `PyStr`
- Numbers like `usize` or `f64` for `PyInt` and `PyFloat`
- `String` for `PyStr`
- And more types implementing `IntoPyObject`.

The `vm` paramter is optional. We add it as the last parameter unless we don't use `vm` at all - very rare case. It takes an object `obj` as `PyObjectRef`, which is a general python object. It returns `PyResult<String>`, which will turn into `PyResult<PyObjectRef>` the same representation of `PyResult`. The `vm` parameter does not need to be passed in by the python code.

If needed a seperate struct can be used for arguments using the `FromArgs` trait like so:

```rust
#[derive(FromArgs)]
struct BisectArgs {
    a: PyObjectRef,
    x: PyObjectRef
    #[pyarg(any, optional)]
    lo: OptionalArg<ArgIndex>,
    #[pyarg(any, optional)]
    hi: OptionalArg<ArgIndex>,
    #[pyarg(named, default)]
    key: Option<PyObjectRef>,
}

#[pyfunction]
fn bisect_left(
    BisectArgs { a, x, lo, hi, key }: BisectArgs,
    vm: &VirtualMachine,
) -> PyResult<usize> {
    // ...
}

// or ...

#[pyfunction]
fn bisect_left(
    args: BisectArgs,
    vm: &VirtualMachine,
) -> PyResult<usize> {
    // ...
}
```

## Errors

Returning a PyResult is the supported error handling strategy. Builtin python errors are created with `vm.new_xxx_error` methods.

### Custom Errors

```
#[pyattr(once)]
fn error(vm: &VirtualMachine) -> PyTypeRef {
    vm.ctx.new_exception_type(
        "<module_name>",
        "<error_name>",
        Some(vec![vm.ctx.exceptions.exception_type.to_owned()]),
    )
}

// convenience function
fn new_error(message: &str, vm: &VirtualMachine) -> PyBaseExceptionRef {
    vm.new_exception_msg(vm.class("<module_name>", "<error_name>"), message.to_owned())
}
```

## Functions
Functions are defined using the `#[pyfunction]` attribute.
```rust, no_run
#[pyfunction]
fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

## Classes
Classes are defined using the `#[pyclass]` attribute.
```rust, no_run
#[pyclass]
pub struct TestClass {
    pub value: i32,
}
#[pyclass]
impl TestClass {
}
```
### Associated Data
TODO.
### Methods
TODO.
### Getters and Setters
TODO.
### Class Methods
TODO.
### Static Methods
TODO.
### Inheritance
TODO.
