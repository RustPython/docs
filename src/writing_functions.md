# Writing Python functions in Rust

In RustPython, it's possible to write functions in Rust and import them into and call them from
Python. Here's an example:

```rust
fn rustmod_funct(
    obj: PyObjectRef,
    s: PyStringRef,
    maybe_thing: OptionalArg<i32>,
    vm: &VirtualMachine,
) -> PyResult<(String, Vec<u8>)> { ... }
```

## Parameters

You can use any type that implements [`FromArgs`] as a parameter to your function, which includes
types that implements [`TryFromObject`]. In our example, we use a standard `PyObjectRef`, a
`PyStringRef` (which is really just a `PyRef<PyString>`, and `PyRef` implements `TryFromObject`),
and an `OptionalArg<i32>`, where [`OptionalArg<T>`] gets an optional positional argument. In
addition, [`TryFromObject`] is implemented for every primitive number type, so you can use those as
args too.
[Here is a list of all of the types that implement `TryFromObject`](https://docs.rs/rustpython-vm/0.1.1/rustpython_vm/pyobject/trait.TryFromObject.html#foreign-impls).

## VirtualMachine parameter

You can optionally put a `vm: &VirtualMachine` parameter at the end of the parameter list in order to
get access to the Python VM. If you're doing anything more than a very simple function, you'll likely
want this, as it is necessary for creating exception objects; but, if it turns out you didn't use it
in the function, always remember that you can just remove it.

## Return type

You can put any type that implements [`IntoPyObject`] as the return type of your function, which includes
many simple Rust types that you'd expect to map cleanly to Python, e.g. `String` -> `str`, `Vec<u8>` -> `bytes`,
integral primitives -> `int`, `f32,f64` -> float. If you need to return an error from the function, you can
put any [`IntoPyObject`] type as the `T` in [`PyResult<T>`], and return an `Err(exc)`. (You can create the
exception using one of the `new_*_error` methods on [`VirtualMachine`])

[`FromArgs`]: https://docs.rs/rustpython-vm/*/rustpython_vm/function/trait.FromArgs.html
[`TryFromObject`]: https://docs.rs/rustpython-vm/*/rustpython_vm/pyobject/trait.TryFromObject.html
[`OptionalArg<T>`]: https://docs.rs/rustpython-vm/*/rustpython_vm/function/enum.OptionalArg.html
[`IntoPyObject`]: https://docs.rs/rustpython-vm/*/rustpython_vm/pyobject/trait.IntoPyObject.html
[`PyResult<T>`]: https://docs.rs/rustpython-vm/*/rustpython_vm/pyobject/type.PyResult.html
[`VirtualMachine`]: https://docs.rs/rustpython-vm/*/rustpython_vm/struct.VirtualMachine.html
