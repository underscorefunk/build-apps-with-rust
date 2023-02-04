# Structuring Applications

You have two options:
1) Make a new crate from inside /src of a crate and load it as a module in cargo.toml
2) Split up code into modules, load those modules into a folder that becomes the namespace through a mod.rs file in a folder of the aformentioned name

## Crates

Important Distinctions
- library crates expose functions that other crates can call and use 
- binary crates are meant to be run on their own.

Read The Cargo Book — https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/cargo/index.html

### Takeaways
- Cargo.toml
  - in a bin, put cargo.lock in git so that it rebuilds with the same deps versions
  - cargo upgrade // update dependencies
  - Versioning (semver)
    - Before you reach 1.0.0, anything goes, but if you make breaking changes, increment the minor version. In Rust, breaking changes include adding fields to structs or variants to enums.
    - After 1.0.0, only make breaking changes when you increment the major version. Don’t break the build.
    - After 1.0.0, don’t add any new public API (no new pub anything) in tiny versions. Always increment the minor version if you add any new pub structs, traits, fields, types, functions, methods or anything else.
  - Profiles can be created to determin how cargo run acts
  - You can create config dependent dependencies

- Create sub crates by running `cargo new cratename --lib`
  - Make a requirement in cargo.toml with `cratename = {path:cratename}`
  - Use the crate in your main project with `extern crate cratename;`

## Modules
Read — https://doc.rust-lang.org/rust-by-example/mod.html

### Take aways
- Make a mod with the syntax `mod modname{...}`
  - expose things to the public with `pub` keyword
  - use them via `modname::modfunctionname()`
- Modules can be nested
- Modules can have visibility for their fields too
- `use` binds the last component of a path as the accessible name 
  - `use foo::bar::baz`
  - `use foo::bar::{baz,bazsibling}`
  - `use foo::bar::baz as renamedbaz`
- `use` statements can be used in block scopes for more concise code
- Dynamic module roots
  - `self::` refers to current scope as root
  - `super::` refers to parent scope as root
  - `crate::` refers to crate root as scope
- A module can be loaded into a *.rs file with `mod modname`
  - rust will look for modname.rs and modname/mod.rs
  - public components will be exposed on the name space
  - A common pattern is
    - A module at `modfolder/somemod.rs` with public components
    - A module at `modfolder/mod.rs` with `pub mod somemod;` with brings somemod into the scope of `mod.rs`
    - Use the module in `main.rs` with `mod modfolder;` and calling `modfolder::somemod::somemodfn()`

## Workspaces
Create an environment with multiple crates which depend on eachother but keep them separate for faster compilation, separation, etc

Read - https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html

### Takeaways
- Workspaces allow for efficient code reuse because you can have multiple distinct crates in a single work space
- Create a folder and put a cargo.toml in it
 ```toml
	[workspace] 
	members = [ 
		"some-binary-crate", 
		"some-lib-crate", 
		"another-lib-crate"
	]
```
- Crate the crate AFTER specifiying its membership or you'll get warnings
```bash
cargo new some-lib-crate --lib
```
- Specify dependencies on other crates in the workspace with the following cargo.toml entry
```toml
some-lib-crate = { path = "../some-lib-crate" }
```
- Use ```bash cargo run -p some-binary-crate ``` in the workspace folder to run the binary crate. -p specifies package
- You can run tests with ```bash cargo test -p some-library-crate ``` as well