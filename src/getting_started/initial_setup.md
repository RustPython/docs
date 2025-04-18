# Initial Setup

First `rustpython_vm` needs to be imported.
If `rustpython` is installed, it can be imported as a re-export:
```rust
use rustpython::vm;
```

if `rustpython_vm` is installed, it can be imported just like any other module.

```rust
use rustpython::vm;

fn main() -> vm::PyResult<()> {
    vm::Interpreter::without_stdlib(Default::default()).enter(|vm| {
        let scope = vm.new_scope_with_builtins();
        let source = r#"print("Hello World!")"#;
        let code_obj = vm
            .compile(source, vm::compiler::Mode::Exec, "<embedded>".to_owned())
            .map_err(|err| vm.new_syntax_error(&err, Some(source)))?;

        vm.run_code_obj(code_obj, scope)?;

        Ok(())
    })
}
```

This will print `Hello World!` to the console.

## Adding the standard library
If the `stdlib` feature is enabled,
the standard library can be added to the interpreter by calling `add_native_modules`
with the result of `rustpython_stdlib::get_module_inits()`.
```rust
use rustpython::vm as vm;
use std::process::ExitCode;
use vm::{Interpreter, builtins::PyStrRef};

fn py_main(interp: &Interpreter) -> vm::PyResult<PyStrRef> {
    interp.enter(|vm| {
        let scope = vm.new_scope_with_builtins();
        let source = r#"print("Hello World!")"#;
        let code_obj = vm
            .compile(source, vm::compiler::Mode::Exec, "<embedded>".to_owned())
            .map_err(|err| vm.new_syntax_error(&err, Some(source)))?;

        vm.run_code_obj(code_obj, scope)?;
    })
}

fn main() -> ExitCode {
    // Add standard library path
    let mut settings = vm::Settings::default();
    settings.path_list.push("Lib".to_owned());
    let interp = vm::Interpreter::with_init(settings, |vm| {
        vm.add_native_modules(rustpython_stdlib::get_module_inits());
    });
    let result = py_main(&interp);
    let result = result.map(|result| {
        println!("name: {result}");
    });
    ExitCode::from(interp.run(|_vm| result))
}
```

to import a module, the following code can be used:
```rust, no_run
// Add local library path
vm.insert_sys_path(vm.new_pyobj("<module_path>"))
    .expect("add examples to sys.path failed");
let module = vm.import("<module_name>", 0)?;
```
