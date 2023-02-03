# Server Functions

>[Official `leptos_server` documentation](https://docs.rs/leptos_server/0.1.3/leptos_server/)

A server function is a function that runs on the server. It can perform tasks or return values.

>**Design Pattern Aside:** It's often a good idea to separate functions that do something, and functions that return values. This is called a "Command/Query" design pattern.

## Isomorphic

When a server function is _called in the context of the **server**_, it will run as expected. 

When a server function is _called in the context or from the **client**,_ it will dispatch a request to the server, which will in turn run the server function on the server. Any returned data will be transparently returned by the original function call on the client, as if it had done the work. The ability to call the same function with these different implementations is why Leptos is considered an _Isomorphic_ framework.

## Why though?

You may ask, why do we care if we run code on the server or the client? It might feel like e should push as much code to the client as possible so that we don't have a lot of server costs and to cut down on the response time of requesting new data from the server. The answer to this question is always, "it depends." 

There are some things we don't want to do on the client. A lot of these decisions come from the reality that the client is a multipurpose device with variable computational ability (slow) and we can't guarantee that users aren't doing anything sketchy on their clients (untrusted/unreliable).

By using server functions we can move computationally heavy tasks to the server, where they can also be cached. We can also move tasks like requests for data to a place where credentials to access that data are out of reach from the client.

An interesting side effect is that by doing more on the server we can often do the work once, cache it, and then serve it to multiple clients. This allows us to use less total power across our application (server and client) foot print. It may allow us to ship smaller client side applications which also reduces waste. It also allows us to create a quality and inclusive experience for lower power or older devices, encouraging people to keep those devices for longer.

## Setup

Server functions are declared by prefixing a function with the `#[server()]` macro. The server macro's first argument should be the name you're giving to the server function. Here we've called it `MyServerFunction`

```rust
#[server(MyServerFunction)]
async fn my_server_fn(cx: Scope) ->Result<(),ServerFnError> {
	println!("You rang?");
}
```

### Source function specifications

- The functions must be `async`.
- It must have serializable arguments
- It must return `Result<T, ServerFnError>`
- The Ok returned type `T` must be serializable. 
- `cx: Scope` is an optional first parameter but is a snapshot of the scope provided by the server. It does not grant access to the reactive system like it does in components.

### Server function naming

It's important to note that the name of the function `my_server_fn` and the name of the server function `MyServerFunction` are different. This allows us to address the prepared server function separately from its source function `my_server_fn`.

### Server function registration

We now need to register the server function with Leptos. We do this by calling the `register()` method on it in the main function. Thankfully we have the name of the server function as a handle to do this.

```rust
fn main(){
	_ = MyServerFunction::register();
}
```
>The register method returns a `Result<(),ServerFnError>`. Rust will complain about the return value of the method call not being handled and requires us to specify that it exists and that we're doing something with it. We assign it to an underscore which tells rust, "We see this assignment and we're explicitly doing nothing with it... but we see it."                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       

>!!! Currently Paused !!! Serialize trait required for empty action prevents progress. Ask Greg about how this is supposed to work. :D

