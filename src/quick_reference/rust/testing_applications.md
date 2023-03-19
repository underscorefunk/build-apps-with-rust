# Testing Applications (Unit Tests)

Tests can be run in 2 ways:
1) In test modules (or as functions)
2) Documentation tests

Tests can:
1) Test assertions
2) Tests panics
3) Test result Errors

Important distinctions:
- Libraries (contains lib.rs) - Integration tests should be in the crateRoot/tests directory  
- Binaries without a Library (no lib.rs) - Binaries do not export and can be used with `extern crate` import syntax. This is why it's recommended to have a lib.rs which exports functionality so that it can be tests and is also consumed by the main.rs binary entry point.

## Unit Tests
- Reading Testing — https://doc.rust-lang.org/1.30.0/book/2018-edition/ch11-00-testing.html
- Reading Rustdoc tests - htt****ps://doc.rust-lang.org/rustdoc/documentation-tests.html

## Takeaways
- A basic test looks like this
  - A cfg annotation macro that says to only run if in test mode
  - A mod to namespace/group the tests, encapsulating them
  - A test macro that sets up the function as a test
  - The assertion to test
```rust
  #[cfg(test)]
  mod test {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
  }
```
  - Writing `use super:*` will bring the parent scope into the tests module for use
  - Useful macros include. Only testCase, a, and b require arguments. The others provide custom error messaging 
```rust 
  assert!(testCase,errorMessage, errorMsgTemplateValues...); 
  assert_eq!(a,b,errorMessage, errorMsgTemplateValues...); 
  assert_ne!(a,b,errorMessage, errorMsgTemplateValues...);
```
   - Panic states can be tested by adding the `#[should_panic]` attribute after `#[test]` and before the function definition 
   - A function that returns a result doesn't require an assertion. The returned result will be tested and the Err returned will make the test fail
```rust
  fn it_works() -> Result<(), String> {
      if 2 + 2 == 4 {
          Ok(())
      } else {
          Err(String::from("two plus two does not equal four"))
      } 
  } 
```
  - tests can be run in parallel by specifying the number of threads. The -- is required to set no value for the name of the test to run.  `cargo test -- --test-threads=2`
  - Examples included in rustdoc doc blocks are run as tests if you run ``` rustdoc --test src/filetotest.rs```
```rust
//  (two slashes is a regular comment, three is a documentation comment

/// ```
/// let x = 5;
/// ```

// or 

/// ```should_panic
/// assert!(false);
/// ```
```

