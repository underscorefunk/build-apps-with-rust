# Cargo Leptos Overview

The following is an overview of the example project setup in the [Cargo Leptos Setup](cargo_leptos_setup.md) lesson. The example project looks like a lot. We'll step through the important parts so that you can find you way around. 

## Front end and back end together

One important think to keep in mind is that both front end and back end code exists in the same code base. Config macros are used to include or exclude code from the final binaries or WASM depending on the target. This allows us to take a whole code base and compile only the server code for the server application, while being able to compile only the front end code for the WASM application.

## Point of entry —`/src/main.rs`

The server is run from  `src/main.rs` .  In this file we'll see macros like `#[cfg(feature = "ssr")]`. These macros tell Rust to only include the following function if the "ssr" feature is enabled. SSR stands for "Server Side Rendering". This is what our web server is doing. You may have noticed that the ready message for Cargo Leptos has "--feature=ssr" in it. 

### Server only use statements

At the very top we'll see a set of use statements wrapped in a macro that roughly reads as "if ssr is enabled, do the following."

```rust
cfg_if::cfg_if! { if #[cfg(feature = "ssr")] {
	//...
}
```

Here we're including the external crates required for the Axum web server. 

### Server only main function

Next we'll see our main function. Above it we have a feature macro which only allows the compiler to see this function if the ssr feature flag is set.

```rust
#[cfg(feature = "ssr")]
```

### Async main function

Then we have another curious macro:

```rust
#[tokio::main]
```



Tokio is an async runtime. Async means that the code can run simultsneously. Synchronous code will run in order.