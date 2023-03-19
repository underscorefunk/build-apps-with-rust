
# Documenting Applications

Documentation with rust is easy and amazing. Documentation comments support markdown.

Docs can be created 2 ways:

1. rustdoc commands
	- Manually trigger documentation authoring, destination, etc
2. `cargo doc`
	- Docs are created using the src/lib.rs and placed in the /target folder
	- You probably want to build docs with `cargo doc` :)


There are two types of doc line starts:
1. `///` (three slashes)
	- Standard documentation comments
2. `//!` (double slash bang) 
	 - Must come before any element in its scope
	 - Used for documenting a crate or jus tinside a struct, fn etc
	 - Think of it as a block preample that describes a scope

Good documentation includes:
1. A short sentence explaining what is happening
2. A code example that users can copy/paste to try it
3. Advanced explanations if necessary

*Fun fact: Documentation lines are syntactic sugar for compiler directives*

Read - https://doc.rust-lang.org/rustdoc/what-is-rustdoc.html

## Takeaways
- Create docs with `rustdoc src/lib.rs --crate-name docFolderNameHere`
	- src/lib.rs is general the entry point, so we create docs for the sub components by starting here (as the entry point) listed as the first argument in the `rustdoc` command
- Certain documentation comments have destrictions for where they can be used:
```rust
//! Inner documentation comment can go here

/// This first line will be used as a sumamry of the 
/// function in the doc index file
pub fn foo() -> i32 {
	//! Inner documentation can go here
	let n = 10;
	{
		//! Inner documentation can go here too
		let n = 15;
	}
	// Double slash is a full comment (ignored by the compiler)
	// We can not place an inner doc comment here because it is 
	// not the first comment in the block scope
}
/// Inner standard documentation comments can go here 
```
- You can link to documentation pages with [....] like in standard markdown. For example ` [foo()]` or `/// [crate::foo()]`
- Documentation can contain test that can be run via `rustdoc src/lib.rs --test`
	- You can hide lines in test from being printed to the documentation by adding the pound sign `/// # this documentation comment is hidden from output but will be compiled`
	- Codeblocks for compiled rust in documentation both start and end with `/// ``` ` (three slashes to denote the documentation target and then three back ticks to denote the opening of a code block)
	- Modifiers can be added to opening code block lines to express compilation intent or to change how they're run
		- `/// ```ignore ` - doesn't get compiled
		- `/// ```should_panic`
		- `/// ```no_run` - compiled but not run. Useful when documenting api calls
		- `/// ```compile_fail`
	- The `?` will return an error if found. Wrap the test with a function that terminates the error. We can hide the error terminator wrapper with the # symbols.
```rust
/// ```
/// use std::io;
/// # fn main() -> io::Result<()>{
/// let mut input = String::new();
/// io::stdin().read_line( &mut input)?;
/// # Ok(())
/// # }
/// ```
```
- Compiler annotations can be used in this context to specify playform specific features too, but that's not too important right now. :)