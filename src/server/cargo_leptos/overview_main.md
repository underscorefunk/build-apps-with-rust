# Cargo Leptos `main.rs`

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

### Async server only main function

Then we have another curious macro:

```rust
#[tokio::main]
```

Tokio is an asynchronous (async) runtime for Rust. This macro initializes tokio as the software thats will handle the implementation of functions using the async keyword.

To make sense of this we'll need to go over the difference between synchronous and asynchronous code. Code that is synchronous runs sequentially. One piece of code runs after the next, in order. This is very predictable and easy to reason about because there is a linear timeline. There are problems with synchronous code though. It can only do one thing at a time. If we had a web server we would only be able to handle one response at a time. What we would like to do is handle responses as they come in, whenever they come in. Responses should be handled out of sync, or async. ^.^

Thinking about synchronous and asynchronous code involves a bit of a paradigm shift. A change in the way you think about code.

Here's an example of a synchronous process written as simple instructions:

```
Count to five and make a sandwich
Eat the sandwich when it's ready
```

In synchronous code the result of these instructions would look like this:

```
1,2,3,4,5
(started making a sandwich)
(finished making a sandwich)
(consumed the sandich)
```

Asynchronous code would yield something more like this:

```
1,
(started making a sandwich)
2,
3,
4,
(finished making a sandwich)
(consumed the sandwich)
5,
```

This result occurs because these tasks are running in parallel. We're counting AND making the sandwich, asynchronously.

In many languages we would call these **promises.** We don't have a sandwich, but we have a stand in which is a promisory value that it will become a sandwich. In rust, we call these *Futures.* The result of calling an async function is a struct that implements the [Futures trait](https://docs.rs/futures/0.1.31/futures/future/trait.Future.html). Which is to say, a struct that has a set of methods we are guaranteed to be able to call. 

Two approaches are commonly used when dealing with asynchronous code.

1) **Callbacks:** We don't know when an async function will conclude. In order to use the data that results from it, or to do something upon completion, we provide a callback. A function to be run on completion/failure/resolution of the async function. This is done through the [`then()`](https://docs.rs/futures/0.1.31/futures/future/trait.Future.html#method.then) method, which is part of the Futures trait. 

```rust
// Get a copy of me
let hungry_me = get_the_author();
// imagine that make_sandwich was an async function
make_sandwich() 
	.then(      
		move |sandwich| hungry_me.eat(sandwich) 
	);	
// This line will run while the sandwich is being made
```

2) **Awaiting:** We can wait for the async function to complete, pausing this part of the program and letting it resume when it's complete. This allows async code to read as synchronous code. Awaiting happens inside other async tasks. Now `om_nom_nom` can run and take the time it needs without holding up the rest of the application.

```rust
async fn om_nom_nom() {
	let hungry_me = get_the_author();
	let sandwich = make_sandwich().await();
	hungry_me.eat(sandwich);
}
```

#### Why do we need tokio?

There is an async runtime built into the browser which can be taken for granted when working on the front end in Javascript. The needs of how async tasks are handled for a systems language are varied. If the Rust language developers got it wrong, we'd be stuck with it. To safe this they've said, "Hey, we're going to tell you which syntax to use when you write async code. We'll tell you which methods you can call on Futures with the Trait specification, but we're not going to tell you how it'll work. You'll need to bring your own implementation." This gives us huge freedom because we can decide how async code actually runs! Tokio is handling the implementations of those traits for us. Very rad.

### Inside the server main function

We'll see a few main components inside the main function
1) Initialization of a logger that gives nicely formatted and timestamped console log output.
2) Setup of the server's config
3) Creation of our application
4) Injection of our application into the axum server so that our app will be used to handle the processing of requests to generate responses. Axum handles the nitty gritty of preparing the requests and formatting the final parts of the response so that we can focus on our application specific components such as updating state and rendering ui.

#### 1) Simple Logger

Simple logger allows us to call nice log messages like this:

```rust
log!("listening on http://{}", &addr);
```

#### 2) Server config

We're calling `get_configuration()` with an argument of `None` which will load the configuration from settings from Cargo Leptos. We could also replace `None` with `Some("./cargo.toml")` to load the settings we've established in our `cargo.toml`. In the example you'll see them listed under the `[package.metadata.leptos]` heading.

`leptos_options` is a property on the `conf` that was setup using the `cargo.toml` settings. We need to pass the options to a few places so we've assigned it to it's own variable. We've also cloned the site address so that we can pass it around without it being a reference into a struct. If we didn't clone it, Rust would likely complain about leptos_options having moved but we have a reference to it which is going elsewhere (like in the log message and used to bind the server to the address).

We generate the routes list from a view which contains our App Leptos component. If you peek into `/src/app.rs` you'll see the declarative way routes are listed. We'll get into that later. Just know that this function is reading our declarative routes and turning them into routes that the web server (Axum) can use.

#### 3) Setup our app as a service for axum

Our app will be a service which axum calls upon to handle and provide responses to requests. Axum's `Router` is the starting block for building this out. We initialize a new router. Then we append a route on it which is specially designed to work with Leptos. We say that axum will have routes for `api/function_name` which uses the function name to connect with our Leptos server functions. This is achieved through the `leptos_axum` integration module's `handle_server_fns`.

The `leptos_axum` integration adds a method to the `axum::Router` struct which allows us to configure the axum::Router with leptos config. That's what's happening here:

```rust
.leptos_routes(leptos_options.clone(), routes, |cx| view! { cx, <App/> })
```

The `fallback` allows us to serve the `error_template` component from  `/src/error_template.rs` if a route couldn't be found to serve. The internals of fallback are also responsible for serving static files like WASM and JS, because those aren't routes either.

And finally, `layer` allows us to create an extension with an `arc` (Asynchronous Reference Counted) value. The extension allows us to create something that all responses can see. The `arc` allows us to make it safe to pass the data across threads (required for axum). In the `arc` we place the `leptos_options`. 

#### 4) Start the web server

In this last step we bind the address to the axum server and tell it to serve our app, which we convert into a service for axum to use. We await it so that it continues to run until the process shuts down. This prevents the main function from hitting the end of its scope and concluding the applications running.

### Client side main function

At the bottom of main.rs you'll see the following:

```rust
#[cfg(not(feature = "ssr"))]  
pub fn main() {  
    // no client-side main function    
    // unless we want this to work with 
    // e.g., Trunk for pure client-side testing    
	// see lib.rs for hydration function instead
}
```

The following macro tells Rust to include this function if we're not in ssr (server) mode, which is denoted by the ssr feature being disabled. That would occur if we're compiling for the client side. In this setup we will only be looking at server side handling/routing.

