# env-rs
**WIP:** Rust env parsing library inspired by https://github.com/caarlos0/env and https://github.com/clap-rs/clap

## Goals
* Parse environment variables into a configuration struct.
* Prefixing variables and tree like destructuring
* Optional/Required/Defaults

### Extras
* Integrate with configuration file/command line arguments to support mutliple sources
  * Arguments > Env > Config File?

### Example
```rust
#[derive(Env, PartialEq)]
#[env(prefix = "FOO")]
struct Config {

    /// Supports a variety of types
    /// And declaring their defaults
    #[env(name="TIMEOUT", default="5s")]
    timeout: std::time::Duration,

    /// Optional by default
    #[env(name="OPTIONAL")]
    optional: Option<i32>,

    /// Required by default
    #[env(name="REQUIRED")]
    required: String,

    /// If the type impls default::Default
    /// then no default needs to be explicitly specified
    #[env(name="DEFAULT", default)]
    default: i32,

    nested: NestedConfig,
}

#[derive(Env, PartialEq)]
#[env(prefix = "BAR")]
struct NestedConfig {
    #[env(name="BAZ")]
    baz: String,
}

fn main() {
    env::set_var("FOO_BAR_BAZ", "baz");
    env::set_var("FOO_TIMEOUT", "1 second");

    // Error because missing required parameter
    assert!(Config::parse_env().is_err());

    env::set_var("FOO_REQUIRED", "required");
    let config = Config::parse_env().unwrap();

    let expected_config = Config {
        timeout: std::time::Duration::from_secs(1),
        optional: None,
        required: String::from("required"),
        default: 0,
        nested: NestedConfig {
            baz: "baz",
        }
    };
    assert_eq!(config, expected_config);
}
```