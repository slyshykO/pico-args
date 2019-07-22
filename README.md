## pico-args
[![Build Status](https://travis-ci.org/RazrFalcon/pico-args.svg?branch=master)](https://travis-ci.org/RazrFalcon/pico-args)
[![Crates.io](https://img.shields.io/crates/v/pico-args.svg)](https://crates.io/crates/pico-args)
[![Documentation](https://docs.rs/pico-args/badge.svg)](https://docs.rs/pico-args)

An ultra simple CLI arguments parser.

- Only flags, options and free arguments are supported.
- Arguments can be separated by a space or `=`.
- No help generation.
- No combined flags (like `-vvv` or `-abc`).

### Example

```rust
use pico_args::Arguments;

struct Args {
    help: bool,
    version: bool,
    number: u32,
    opt_number: Option<u32>,
    width: u32,
    free: Vec<String>,
}

fn parse_width(s: &str) -> Result<u32, String> {
    s.parse().map_err(|_| "not a number".to_string())
}

fn main() -> Result<(), Box<std::error::Error>> {
    let mut args = Arguments::from_env();
    // Arguments can be parsed in any order.
    let args = Args {
        // You can use a slice for multiple commands
        help: args.contains(["-h", "--help"]),
        // or just a string for a single one.
        version: args.contains("-V"),
        // Parses a value that implements `FromStr`.
        number: args.value_from_str("--number")?.unwrap_or(5),
        // Parses an optional value that implements `FromStr`.
        opt_number: args.value_from_str("--opt-number")?,
        // Parses a value using a specified function.
        width: args.value_from_fn("--width", parse_width)?.unwrap_or(10),
        // Will return all free arguments or an error if any flags are left.
        free: args.free()?,
    };

    Ok(())
}
```

### Alternatives

The core idea of `pico-args` is to provide some "sugar" for arguments parsing without
a lot of overhead (binary or compilation time wise).
There are no point in comparing parsing features, because `pico-args` supports
only the bare minimum. So we will compare only the size overhead and compilation time.

There are a lot of arguments parsing implementations, but we will use only these one:

- [clap](https://crates.io/crates/clap) - is the most popular and complete one
- [gumdrop](https://crates.io/crates/gumdrop) - a simple parser that uses procedural macros
- [structopt](https://crates.io/crates/structopt) - a two above combined

| | `pico-args` | `clap` | `gumdrop` | `structopt` |
---|---|---|---|---
| Binary overhead | 18.9KiB | 435.1KiB | 23KiB | 436.8KiB |
| Build time | 0.9s | 15s | 31s | 27s |
| Tested version | 0.1.0 | 2.33.0 | 0.6.0 | 0.2.18 |

- Binary size overhead was measured by subtracting the `.text` section size of an app with
  arguments parsing and a hello world app.
- Build time was measured using `hyperfine 'cargo clean; cargo build --release'`.
- Test projects can be found in `examples/`.

### License

MIT