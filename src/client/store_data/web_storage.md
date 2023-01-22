# Web Storage / Local Storage

>For more information visit the [MDN Web Storage API documentation](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)

## What we know

- How to setup basic event handlers in Leptos

## What we'll learn

- How to store and retrieve data from a domain's local storage in the client

## What's missing

- Type safety guarantees for non-string types
- Session storage and local persistant storage

## Caveats

- Local storage can be modified by users and other applications running on the same domain. As with pretty much everything happening on the client, you can't trust it.

## The lesson

Web storage allows us to store data in the browser that will live for the duration that the browser is open (session storage) or will persist until the browser's data is cleared (local storage). 

How these differ from cookies:
- Larger amounts of data can be stored
- The api to interact with them is easier to use. The web storage api is much newer than cookies.
- They're not sent to the server when making new requests.

In this lesson we're going to initialize a local storage value, apply some modifications to it, and read the value. Our example will be a counter.

Let's start with a basic client side leptos app that has a button to initialize our local storage value. Currently we just have a log message that will print a message to the browser's console when the button is clicked. This way we can confirm the handler is working.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <App />  
        }
	})
}  
  
#[component]  
fn App(cx: Scope) -> Element {  
    let initialize_value = |_|{  
      leptos::log!("Initialize a value in local storage");  
    };  
    view! {  
        cx,  
        <div>  
            <button on:click=initialize_value>
	            "Initialize value"
			</button>  
        </div>  
    }
}
```

### Accessing the Web Storage API

We will need to find a way to call out to the Web Storage API through web_sys. The first step is finding out where web storage exists in JavaScript in the browser. The [MDN Web Storage API documentation](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API) state, "These mechanisms are available via the [`Window.sessionStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage) and [`Window.localStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) properties."

Leptos provides us with a `window()` function which will efficiently return a `web_sys::Window`, allowing us to communicate to the browser's `window`. 

If we look at the `web_sys::Window` [documentation](https://rustwasm.github.io/wasm-bindgen/api/web_sys/struct.Window.html#method.local_storage), we'll see that there is a `local_storage` method!

Calling `local_storage` returns a `Result<Option<web_sys::Storage>, web_sys::JsValue>`.  We'll need to get the `Ok(Some(storage))` from it (`Ok` because of the result which contains `Some` because of the option). Once we do, we'll get that `web_sys::Storage`, which we can work with. The [`web_sys::Storage` struct's documentation](https://rustwasm.github.io/wasm-bindgen/api/web_sys/struct.Storage.html) enumerates all of the methods we can call, including `get()` and  `set()`!

#### Chaining Unwraps

You can use unwarps to extract the `Ok` and `Some` like this:

```rust
let storage = window().local_storage().unwrap().unwrap();
```

The problem is that `upwrap()` will throw a panic if it's the `Err` or `None` variants of `Result` and `Option` respectively. We don't want our application to panic! 

#### Nested Matches

We can use pattern matching as a potential solution:

```rust
match window().local_storage() {
	Ok( maybe_some_storage ) => match maybe_some_storage {
		Some( storage ) => {
			// Do your stuff wth storage
		},
		None => {}
	},
	Err() => {}
}
```

This solution doesn't panic, which is great. But it does put our code in deeply nested scopes which makes it hard to read.

#### Assigning the value of a match

What we're looking to do is assign the value to `storage` if it can be retrieved from the nested Result/Option, otherwise we'll do nothing. Do keep in mind that most applications will want to do something in the event that expected behaviour can't be followed. 

A match statemetn will evaluate to the value of its matched expression. We can assign that value to a variable!

Currently that value is wrapped in an Option, which is then wrapped in a Result. We are able to use the same nesting of our return types to create a patterns to extract the value we're looking for. By combining these we can say, "If local_storage() returns an `Ok` that has `Some` `storage`, let the value of the match statement be `storage`." The other pattern marked with an underscore (`_`)  indicates a catch-all. Anything that doesn't match what we want will return, breaking out of our closure!

```rust
let storage : web_sys::Storage = match window().local_storage() {  
    Ok(Some(storage)) => storage,  
    _ => return   
};
// We will only run code here if storage was able to be unwrapped by the match
```
> I added the `web_sys::Storage` type to make this more clear, but Rust will infer the type. You do not need to write it.

### Working with the web storage API

#### Setting a value

We can now call the `Storage` api through our `storage` variable (note the lowercase 's'. `Storage` is the struct/type, `storage` is our value). 

Here we are assigning (setting) the key "my-counter" with a value of `0`. 

```rust
storage.set("my-counter", &0.to_string());
```

It's important to note that the web storage api stores strings. We can represent numbers and complex data in string form, being as the numerical character or as serialized data respectively. Rust requires that we convert our integer `0` to a string with the `to_string()` method. This provides us with an owned string. As per the documentation set is looking for a reference to a string, a string slice (`&str)`. We can meet these requirements by prefixing the whole thing with an ampersand `&`.

### Retrieving a value

We can use the get method to retrieve a value from a given key.

```rust
let my_counter_value = storage.get("my-counter");
```

The return type of `get` is `Result<Option<String>,JsValue>`. We can use unwrap_or_default to safely unwarp or fail. 

```rust
let my_counter_value : String = storage  
    .get("my-counter")   // at this point we have a Result<Option<String>,JsValue>
    .unwrap_or_default() // Gives us Option<string> or the default value
    .unwrap_or_default(); // Gives us string or the default value, an empty string
```

In our case, we're using a number, so we'll need to parse it.

```rust
let my_counter_value = storage  
    .get("my-counter")  
    .unwrap_or_default()  
    .unwrap_or_default()  
    .parse::<i8>()  // attempt to parse the string as an 8 bit integer
    .unwrap_or_default(); 
    // ^ Return the Ok result, which is an 8 bit integer 
	//   or if there was a parse error, return the default value for an 8 bit integer
```

We can also write this as follows.

```rust
let my_counter_value : i8 = storage  
    .get("my-counter")  
    .unwrap_or_default()  
    .unwrap_or_default()  
    .parse()  
    .unwrap_or_default();
```

We could rewrite this as a a match statement. Here's an example with a little twist. We're specifying what the fallback value should be in a more visible way:

```rust
let my_counter_value: i8 = match storage.get("my-counter") {  
    Ok(Some(value)) => value.parse().unwrap_or(0),  
    _ => 0  
};
```

### Making a module

We can wrap these two bits of functionality into a nice little module for reuse:

```rust
mod local_storage {  
  
    use leptos::*;  
  
    pub fn set(key : &str , val : &str ) {  
        let storage = match window().local_storage() {  
            Ok(Some(storage)) => storage,  
            _ => return  
        };  
        storage.set(key, val);  
    }  
  
    pub fn get(key : &str ) -> String {  
        let storage = match window().local_storage() {  
            Ok(Some(storage)) => storage,  
            _ => return "".to_string()  
        };  
  
        match storage.get(key) {  
            Ok(Some(val)) => val,  
            _ => "".to_string()  
        }
	}
	
}
```
>Again it is important to note that we are not handling any errors here. 

Calls to our local storage module are all nicely cleaned up:

```rust
// set a value
local_storage::set( "my-counter", &22.to_string() );  

// get a value
let v: i8 = local_storage::get("my-counter").parse().unwrap_or_default();
```

It would be great to avoid this whole parse and unwrap business as well. Let's see if we can't clean that up even more.

We'll need a generic type here. I'm going to use `Val` because it'll connect with `val` (the actual value). Many people use `T`. The letter doesn't matter. We'll provide it as a type argument by adding `<Val>` after `get` and before the parameter lsit. Then we'll set the return type to be of type `Val` as well, with `-> Val` after the parameter list. I'd like to be able to explicitly set the default value, so I'll add that as a parameter called `default` of type `Val`. The only thing left to do is add the `default` in where we had empty strings before. 

```rust
pub fn get<Val>(key : &str, default: Val ) -> Val {  
    let storage: web_sys::Storage = match window().local_storage() {  
        Ok(Some(storage)) => storage,  
        _ => return default  
    };  
  
    match storage.get(key) {  
        Ok(Some(val)) => val.parse().unwrap_or( default ),  
        _ => default  
    }  
}
```
>You might think that we'd need to provide `Val` in a turbofish for `val.parse::<Val>()` but we don't. Rust's compiler is smart enough (so darn smart). It knows that the final match statement doesn't end in a semicolon, so it must be the final expression. The result of the match will be our return value. This must be a `Val` type. It knows that if it's going to parse it *has* to parse to a `Val` value type!

This isn't quite there yet though. We need to add some type bounds for Val. We can't just accept anything. We want to only acccept thing that are of type `Val` if they can actually be parsed from a string. We can do this by adding the trait bound to the generic `<Val: std::str::FromStr>`. This means that whatever type `Val` is, it must implement the `std::str::FromStr` trait.

```rust
pub fn get<Val: std::str::FromStr>(key : &str, default: Val ) -> Val {  
    let storage: web_sys::Storage = match window().local_storage() {  
        Ok(Some(storage)) => storage,  
        _ => return default  
    };  
  
    match storage.get(key) {  
        Ok(Some(val)) => val.parse::<Val>().unwrap_or( default ),  
        _ => default  
    }  
}
```

Actually, while we're in here, let's make the local storage setter more flexible too. By adding a generic `Val` type we can call to_string() and turn it into a reference within this function. We do need to add constraints to the function. `Val: std::fmt::Display` guarantees that we can call to_string() on whatever type `Val` is.

```rust
// The function definition's type constraint can also be written with a 
// where keyword after the parameter list as follows.
// pub fn set<Val>(key : &str , val : Val ) where Val: std::fmt::Display {  
pub fn set<Val: std::fmt::Display>(key : &str , val : Val ) {  
    let storage: web_sys::Storage = match window().local_storage() {  
        Ok(Some(storage)) => storage,  
        _ => return  
    };  
    storage.set(key, &val.to_string());  
}
```

Now, as you look at this, you should be thinking, "But wouldn't I want to know if I failed to read a value? Won't this module make it look like local storage is working correctly even if it's not?!" Use the knowledge you've gained in this lesson to refactor the module so that your code expresses the behaviour of your application. Think critically about where failures are important to note and handle, and where they're not.

## The final code

Here's the code all wrapped up and rolled together.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <App />  
        }
	})
}  
  
  
mod local_storage {  
    use leptos::*;  
  
    pub fn set<Val: std::fmt::Display>(key: &str, val: Val) {  
        let storage: web_sys::Storage = match window().local_storage() {  
            Ok(Some(storage)) => storage,  
            _ => return  
        };  
        storage.set(key, &val.to_string());  
    }  
  
    pub fn get<Val: std::str::FromStr>(key: &str, default: Val) -> Val {  
        let storage: web_sys::Storage = match window().local_storage() {  
            Ok(Some(storage)) => storage,  
            _ => return default  
        };  
  
        match storage.get(key) {  
            Ok(Some(val)) => val.parse().unwrap_or(default),  
            _ => default  
        }  
    }}  
  
#[component]  
fn App(cx: Scope) -> Element {  
    let initialize_value = |_| {  
        local_storage::set("my-counter", 0);  
        leptos::log!("Init counter to {}", local_storage::get("my-counter", 0));  
    };  
  
    let increment_value = |_| {  
        let value: i32 = local_storage::get("my-counter", 0);  
        local_storage::set("my-counter", value.saturating_add(1));  
        leptos::log!("Increment counter to {}", local_storage::get("my-counter", 0));  
    };  
  
    let decrement_value = |_| {  
        let value: i32 = local_storage::get("my-counter", 0);  
        local_storage::set("my-counter", value.saturating_sub(1));  
        leptos::log!("Decrement counter to {}", local_storage::get("my-counter", 0));  
    };  
  
    view! {  
        cx,  
        <div>  
            <button on:click=initialize_value>"Initialize value"</button>  
            <button on:click=increment_value>"+"</button>  
            <button on:click=decrement_value>"-"</button>  
        </div>  
    }
}
```