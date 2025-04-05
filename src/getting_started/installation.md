# Installation
## Requirements
RustPython requires Rust latest stable version to be installed

## Stable
The latest stable version of the library can be installed using the following command:
```bash
cargo add rustpython
```

or by adding the following to your `Cargo.toml`:
```toml
[dependencies]
rustpython = "0.4"
```

## Nightly
Nightly releases are built weekly and are released on git.
```toml
[dependencies]
rustpython = { git = "https://github.com/RustPython/RustPython", tag = "2025-02-24-main-13" }
```

The tag should be pointed to the latest tag found at https://github.com/RustPython/RustPython/tags.

## Features
By default `threading`, `stdlib`, and `importlib` are enabled.
### `bz2`
If you'd like to use the `bz2` module, you can enable the `bz2` feature.
### `stdlib`
`stdlib` is the default feature that enables the standard library.
### `sqlite`
If you'd like to use the `sqlite3` module, you can enable the `sqlite` feature.
### `ssl`
If you'd like to make https requests, you can enable the ssl feature,
which also lets you install the pip package manager.
Note that on Windows, you may need to install OpenSSL, or you can enable the ssl-vendor feature instead,
which compiles OpenSSL for you but requires a C compiler, perl, and make.
OpenSSL version 3 is expected and tested in CI. Older versions may not work.

