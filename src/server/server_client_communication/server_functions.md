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
	Ok(())
}
```

### Source function specifications

- The functions must be `async`.
- It must have *serializable* arguments
- It must return `Result<T, ServerFnError>`
- The Ok returned type `T` must be *serializable*. 
- `cx: Scope` is an optional first parameter but is a snapshot of the scope provided by the server. It does not grant access to the reactive system like it does in components.

### Server function naming

It's important to note that the name of the function `my_server_fn` and the name of the server function `MyServerFunction` are different. This allows us to address the prepared server function separately from its source function `my_server_fn`.

### Server function registration

We now need to register the server function with Leptos. We do this by calling the `register()` method on the server function, in the main function. Thankfully we have the name of the server function as a handle to do this.

```rust
fn main(){
	_ = MyServerFunction::register();
}
```
>The register method returns a `Result<(),ServerFnError>`. Rust will complain about the return value of the method call not being handled and requires us to specify that it exists and that we're doing something with it. We assign it to an underscore which tells rust, "We see this assignment and we're explicitly doing nothing with it... but we see it."                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       

### Server function dependencies

Server functions depend on the ability to serialize data sent from the client to the server function, and to deserialize the result of a server function on its return trip to the client. You will need to add `serde` to your `cargo.toml` as the serialization implementation.

```toml
serde = {version = "1.0.152", features = ["derive"] }
```

---

## Calling server function

We previously stated that server functions must be async. But our component functions are synchronous. As you can see, there's a problem here. You can not await an async function inside a synchronous function.

There are two bridges inside Leptos that allow the reactive synchronous system to interact with the asynchronous action system:

- **Resources:** Read values
- **Actions:** Dispatch side effects.

>Resources and Actions can be used to bridge anything into the reactive system. While they're necessary for server functions, they are not limited to server functions.



### Reading data with **resources**

>[Official documentation](https://docs.rs/leptos_reactive/0.1.3/leptos_reactive/struct.Resource.html)

>There are some things you will need to do to use data read from resources in your UI. Continue reading after this section for that information.

A resource is a signal, like any that you've made before, that has its value updated when its `source` data is updated. This allows the async task to run in the background and update the signal when it's ready, which our reactive system can react to without waiting for it.

A resource is created with the function `create_resource`, which takes three arguments:
- `cx` — A scope context in which the resource will exist
- `source` — A function that returns the arguments used for the fetcher.
- `fetcher` — An async function (Future) which yields the resource's data when done.

#### Unary (one parameter) fetcher example

The following example exemplifies setting up a resource for a server function with one parameter. In this case, it is a function that accepts a single  8 bit integer (`u8`) called `some_number`.

```rust
#[server(MyServerFunction)]  
async fn my_server_fn(some_number: u8) -> Result<u8, ServerFnError> {
	// stuff happens here
	Ok(42)
}
```
>Note that we need to return `Result` types from server functions, so we have to wrap our return value in `Ok()`.

The first step is to create the signal which contains the arguments used to call the server function.

```rust
let default_server_fn_args : u8 = 6;

let (
	my_server_fn_args, 
	set_my_server_fn_args
) = create_signal(
	cx, 
	default_server_fn_args
);

let server_fn_result = create_resource(  
    cx,    
    my_server_fn_args,    // the signal listened to for changes
    my_server_fn          // the fn run with the server args signal value
);
```

#### Resources for server functions with no parameters or more than one parameter.

Resource `fetcher`s  allow only one parameter. Signals can only contain one value. These constraints require us to wrap aspects of the resource config in closures so that the signatures (data types) of the arguments matches the expected signatures of the `create_resources()` function parameters.

##### No parameters

A signal for a server function that has no parameters can be provided closures for their source and the fetcher can be wrapped in a closure with an unused parameter.

```rust
#[server(MyServerFunction)]  
async fn my_server_fn() -> Result<u8, ServerFnError> {
	Ok(1) 
}
```

```rust
let resource_that_takes_5_secs = create_resource(  
    cx,  
    ||{},    
    |_| my_server_fn()  
);
```

##### More than one parameter

More than one parameter can be achieved by using a tuple which will match the arguments provided, in order, to the server function.

```rust
#[server(MyServerFunction)]  
async fn my_server_fn(x: u8, y: u8) -> Result<u8, ServerFnError> {
	Ok( x + y ) 
}
```

```rust
let default_server_fn_args : (u8, u8) = (46, 2);

let (
	my_server_fn_args, 
	set_my_server_fn_args
) = create_signal(
	cx, 
	default_server_fn_args
);

let server_fn_result = create_resource(  
    cx,    
    my_server_fn_args,          // the signal listened to for changes
    |(x,y)| my_server_fn(x,y)  // Add parens to destructure
);
```


### Displaying async data in the UI with `<Suspense>`

> [Official documentation](https://docs.rs/leptos/latest/leptos/fn.Suspense.html)

The UI system won't know to wait for new UI data. Leptos has a special component called a `<Suspense>` component which will wrap use of our resources (signals tied into async values). Failure to do this results in a common error when working with resources through non-suspense wrapped UI. You can almost think of suspense as an automatically awaiting and resolving UI Future.

>**Common Error:** "You’re trying to update a `Signal<usize>` that has already been disposed of. This is probably either a logic error in a component that creates and disposes of scopes, or a Resource resolving after its scope has been dropped without having been cleaned up."

Suspense has two different states:

- **Fallback/Pending:** If any signals used in the children have _unresolved resources,_ a fallback view will be displayed via the `fallback` property on the `Suspense` component tag.
- **Resolved:** If _all resources are resolved,_ the children of the `Suspense` component will be displayed.

The syntax is as follows:

```rust
let resource_that_takes_5_secs = create_resource(  
	cx,                  // our scope/context   
    ||{},                // a closure that returns fetcher params
    |_| wait_5_seconds() // a closure for the fetcher, which as no params 
);

view!{cx,  
    <Suspense fallback=||"Loading...".to_string()>  
        { move || resource_that_takes_5_secs.read() }  
        "Loaded"  
    </Suspense>  
}
```

Above we created a `<Suspense>` container with a fallback closure. The closure must return an `impl IntoView`. Strings support IntoView, as do results of the `view!` macro. Then we move the resource into a block that can be re-run (a closure), calling `read()` on it. This hooks the resource into the suspense. The rule is, if a resource is used in the suspense, then the suspense must wait for the resources to all be available before it renders its children.

```rust
async fn wait_5_seconds() {  
    futures_timer::Delay::new(Duration::from_secs(5)).await;  
}
```
> We use the `futures_timer` crate (installed with `cargo add futures_timer`) so that we get proper async waiting. If we used `thread::sleep(Duration::from_secs(5))` we'd end up with errors and the whole application would be pausing synchronously.

### Writing data or dispatching side effects with **actions**

>[`Action` Official Documentation](https://docs.rs/leptos/latest/leptos/struct.Action.html)
>[`create_action` Official Documentation](https://docs.rs/leptos/latest/leptos/fn.create_action.html)

Actions allow us to make async calls in our synchronous reactive system. We can use actions to make server function calls by calling them an action's task.

Actions are created with the `create_action` function which accepts two arguments:
- `cx` — A scope context in which the action will be run
- `task` — A function to run asynchronously. Arguments should always be passed by reference

An actions tasks is triggered by calling the `dispatch` method on the `Action`.

```rust
#[server(MyServerFunction)]  
async fn my_server_fn(x: u8, y: u8) -> Result<u8, ServerFnError> {
	Ok( x + y ) 
}
```

```rust
let my_action = create_action(cx, |&(x: u8,y: u8)| {
  my_server_fn(x,y)
}

let my_action_task_args = (46,2);
my_action.dispatch( my_action_task_args );
```

Actions have a few handy methods we can call on them aside from dispatch:
- `input` — the argument currently running
- `pending` — whether the call is pending
- `value` — the most recent returned result
- `version` — how many times the action has run. Useful for reactively updating something else in response to a `dispatch` and response