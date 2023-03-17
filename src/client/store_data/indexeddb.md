# IndexedDB

>__This lesson is in `notes` status and is an extremely rough daft.__
>***This lesson is not a complete notes draft***

## What we know

- Data can be stored in the browser

## What we'll learn

- Basic interaction with IndexedDB

## Caveat

The IndexedDB API is complicated. Like all other web storage APIs, they are modifiable by any other scripts running on the page, making them untrusted. Pushing data from WASM to JS and back is slower than performing more work in WASM and letting Leptos update the resulting data. For these reasons, it's probably a better idea to think about your applications and create purpose built data structurs that you can query and work with instead of using the indexedDB API and data store.

## The Lesson

IndexedDB is a NoSQL like database that lives in the browser. We are able to grab data by creating a cursor that allows us to jump around the field of data that is the database. IndexedDB does not use structured query language (SQL) to retrieve data like MySQL, Maria, Postgres, etc.

The IndexedDB API is notoriously unconfortable to use. This article will provide a cursory overview and exploration of it for those curious. 

Let's get things started with our Leptos [inital client side application](client_side_app_setup.md).

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
    view! {  
        cx,  
        <div>  
            "My App"  
        </div>  
    }
}
```

We'll jump over to the [documentation](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API) and scroll down to "Interfaces". This gives us some hints for where we need to go. 

The documentation reads:
>To get access to a database, call [`open()`](https://developer.mozilla.org/en-US/docs/Web/API/IDBFactory/open) on the [`indexedDB`](https://developer.mozilla.org/en-US/docs/Web/API/indexedDB) attribute of a [window](https://developer.mozilla.org/en-US/docs/Web/API/Window) object.

In my mind I think, "I should make a button that will connect to the database when I click it." I find it's easiest to build and learn if I can provide my own input and introspect the result. 

> **Reoccuring Pattern Alert:** It's interesting how this feedback loop is the same as a servers request->response, or how applications are built in general. Programs are often the way they are because they're expressions of how we think.

```rust
#[component]  
fn App(cx: Scope) -> Element {  
    let connect_to_database = |_|{  
        leptos::log!("Connect to database")  
    };  
    view! {  
        cx,  
        <div>  
            <button on:click=connect_to_database>  
                "Connect to DB"  
            </button>  
        </div>  
    }
}
```

The documentation states that indexedDB is an attribute of the `window` object. We'll use Leptos' `window()` function to grab it's cached reference to `window` as a `web_sys::Window`.

```rust
let connect_to_database = |_|{  
	leptos::log!("Connect to database")    
	let window = window();  
};
```

If we look at the [web_sys::Window documentation](https://rustwasm.github.io/wasm-bindgen/api/web_sys/struct.Window.html#method.indexed_db) we'll see that there is a web_sys version of the indexedDB javascript proprty called `indexed_db`. Note the difference in case. Rust is prescriptive about it's use of snake case for function/method names. We can see that the return type is `Result<Option<IdbFactory>, JsValue>`.

This `IdbFactory` looks interesting. If we check the IndexedDB documentation we'll see a good definition of what it is.

>[`IDBFactory`](https://developer.mozilla.org/en-US/docs/Web/API/IDBFactory): Provides access to a database. This is the interface implemented by the global object [`indexedDB`](https://developer.mozilla.org/en-US/docs/Web/API/indexedDB) and is therefore the entry point for the API.

This is perfect. We want an entry point for the API!

We can use the following match pattern to get the `IdbFactory` (note that this refers to the struct with its PascalCase) out of our `indexed_db()` call, or return (prematurely terminate the closure/click handler) if the pattern doesn't match.

```rust
let idb = match window().indexed_db() {  
    Ok(Some(idb_factory)) => idb_factory,  
    _ => return  
};
```

The `web_sys::indexed_db()` documentation also states:

>_This API requires the following crate features to be activated: `IdbFactory`, `Window`_

We can enable these feature by adding the following to our cargo.toml file

```toml
[dependencies.web-sys]  
features = [ "Window",  "IdbFactory" ]
```

`web_sys` has a lot of features and compiling all of them into the Leptos WASM application would make it needlessly large. For this reason, features are hidden behind feature flags like this so that we can pick and choose what gets added to our final application on a needs basis. Rust is very considerate.

Let's zip on over to the [`web_sys::IdbFactory` struct documentation](https://rustwasm.github.io/wasm-bindgen/api/web_sys/struct.IdbFactory.html) to see what's avaialble to us there. I see an [`open()`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/struct.IdbFactory.html#method.open) method which looks like what we want, so we'll try that. Note that we need to add some extra features to the web-sys crate. `open()` requires "_`IdbFactory`, `IdbOpenDbRequest`_", so we'll add the missing `IdbOpenDbRequest` 

```toml
[dependencies.web-sys]  
features = [ "Window",  "IdbFactory", "IdbOpenDbRequest" ]
```

I made a few changes to make the code more visible. This is where we're at:

```rust
let connect_to_database = |_|{  
  
    // Grabbed a reference to window  
    let window = window();  
  
    // Got a factory to be able to make an open connection  
    let idb = match window.indexed_db() {  
        Ok(Some(idb_factory)) => idb_factory,  
        _ => return  
    };  
  
    let idb_open_request = match idb.open("my-database") {  
         Ok(idb_open_request) => idb_open_request,  
         _ => return  
    };  
  
    // Do something with the connection  
};
```

The question that we now have is, how do we work with IndexedDB? We need some more information.

### Store type

The [MDN documentation](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Basic_Terminology) states that IndexedDB is a key-value store. The values can be complex objects, and keys can be properties of those objects. If we're thinking in terms of Rust, what they're saying is that we can store structs in IndexedDB, where properties like 'id' could be a key used to look up the struct.

### Transactions

All interactions with IndexedDB are done in the form of transactions. The mental model is such that we requst for a change in the database and the database will hold the request until it is safe to perform. This guarantees data integrity where we don't have two sources potentially modifying the same data, etc.

There are three transaction types. We'll only be looking at the first two:

1. readwrite
2. readonly
3. versionchange

### Data Retrieval

Data is not returned from the database as soon as you request it. Recall that we submit requests to the database as transactions for it to perform. The database decides when it is safe to perform those transactions. For this reason we'll need to provide indexedDB with callbacks to run when the transactions are preformed. We'll be reacting to the retrieval of data. In fact, IndexedDB uses DOM events to notifuy us when resuts are available. It's not dissimilar to how we worked with buttons and click events. 

### MDN to the rescue

The wonderful Mozilla Developer Network (MDN) has a reference on [Using IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB). In it, they outline the basic usage steps as follows:

1.  Open a database.
2.  Create an object store in the database.
3.  Start a transaction and make a request to do some database operation, like adding or retrieving data.
4.  Wait for the operation to complete by listening to the right kind of DOM event.
5.  Do something with the results (which can be found on the request object).

We'll follow along, but in Rust and Leptos.

Where we last left off, we created a `web_sys::IdbOpenDbRequest`

```rust
let idb_open_request = match idb.open("my-database") {  
     Ok(idb_open_request) => idb_open_request,  
     _ => return  
};  
```

This isn't a connection persey. It is part of the chain of processes to setup a connection.

In the MDN documentation, the author establishes the same type of object in Javascript and then attaches event handlers to `onerror` and `onsuccess`.

We'll add callback functions/handlers to our Rust version.

If we look at the definitions of [`set_onsuccess`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/struct.IdbOpenDbRequest.html#method.onsuccess) and [`set_onerror`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/struct.IdbOpenDbRequest.html#method.set_onerror) we'll see that these are the functions that allow us to set the values of the onsuccess and onerror propreties of the JavaScript object. Exactly what we're looking for. Their definitions also tell us that we need to add the `IdbRequest` feature to our `cargo.toml` file.

```toml
[dependencies.web-sys]  
features = [ "IdbFactory", "IdbOpenDbRequest", "IdbRequest"]
```

Intuitively I think, "JavaScript accepts functions as the values for these properties, so I should use closures for mine. Though it'll need to be wrapped in a `Some` because it's an option."

```rust
idb_open_request.set_onsuccess(
	Some(
		||{  
			// on success stuff here 
		}
	)
);  
```
>This is all spaced out so that you can easily see the syntax.

Unfortunately, my intuition is off. Even thought the word `Function` looks familiar, we have to remember that in Rust we have `Fn`, `FnOnce`, and `FnOnce` as function types. `Function` isn't a native function type. Looking closer I can see that `Function` is a special struct that is callable by WASM. It is a `js_sys::Function`.

### Creating JavaScript closures in Rust

What we need to do is create a [`wasm_bindgen::closure::Closure`](https://rustwasm.github.io/wasm-bindgen/api/wasm_bindgen/closure/struct.Closure.html) and then  [cast the closure to a js_sys::Function](https://rustwasm.github.io/wasm-bindgen/api/wasm_bindgen/closure/struct.Closure.html#casting-a-closure-to-a-js_sysfunction). The MDN documentation uses a struct with a `_closure` property. We'll do things slightly differently, stepping through each line of code and what it does.

The first step is making a closure. We'll use the [`Closure::wrap`](https://rustwasm.github.io/wasm-bindgen/api/wasm_bindgen/closure/struct.Closure.html#method.wrap) method. The documentation defines it as:

>A more direct version of `Closure::new` which creates a `Closure` from a `Box<dyn Fn>`/`Box<dyn FnMut>`, which is how it’s kept internally.

This sounds like exactly what we want,

```rust
// cb stands for callback
let cb = wasm_bindgen::closure::Closure::wrap(
	// We need something here.
);
```

We need to provide an argument which is a `Box` that contains a `dyn FnMut`. There's a lot here to unpack.

#### Box

In Rust, there are two types of memory allocation, heap and stack. The stack is fast but requires the size of what's being stored in it to be consistent and known. The heap allows us to store things that may change in size, but they need to be looked up in the stack to get their actual values. It's two steps instead of one. Also, the heap isn't as organized as the stack, so it's lookups will also be slower. 

A `Box` is a way for us to store data in the heap. The actual size of a `Box` is known, because it's a pointer to memory in the heap. 

#### dyn (dynamic)

Rust wants to know everything in advance to be able to optimize all code and make its security guarantees. If we're going to be skipping across contiquous zeros and ones in memory and interpreting them as data, we need to know the size of the data we're reading.

Unfortunately sometimes this isn't possible to do at compilation time. When we use traits as types in Rust, we're telling Rust that anything that impements the specified trait is fair game for use as an argument. This is a powerful technique because we're allowing any values to be used in the future provided someone writes an `impl` (implementation) for the given trait.  

For example:

```rust
struct RobotDuck{}  

impl RobotDuck {  
    fn assert_duckitude() {  
        println!(
	        "I'm totally not a robot. 
	        Look at me click on these 
	        images of bread floating 
	        in a pond."
		)  
    }
}  
    
struct RealDuck{}  
  
trait Quack {  
    fn quack(&self){  
        println!("QUACK");  
    }  
}  
  
impl Quack for RobotDuck{}  
impl Quack for RealDuck{}  
```

In the above, example, we have two structs with different functions. As a result, they'll look different in memory. Both of these ducks implement the `Quack` trait and can call the default trait implementation `quack()`.

Let's say we have this function:

```rust
fn this_thing_quacks<T>(quackable: T) where T: Quack {
	println!("This thing quacks!");  
	quackable.quack();  
}
```

We have a trait bound on the generic type `T` that requires implementation of `Quack`. When the compiler runs, it will actually create a version of this function for each type that impements `Quack`.

```rust
fn this_thing_quacks(quackable: RobotDuck){
	//...
}

fn this_thing_quacks(quackable: RealDuck){
	//...
}
```

Recall that functions are also data! Rust needs to have guarantees about the sizes of data as arguments for the function with the generic `T`.

We'll get an error if we try to change the signature to this though.

```rust
fn this_thing_quacks(quackable: Quack) {
	println!("This thing quacks!");  
	quackable.quack();  
}
```

The reason being is that we don't know what size `Quack` is when it is being called. There are multiple things that implement `Quack` and they aren't all the same size! Rust doesn't stamp out the different versions because it hasn't been pre-defined. When we use a trait bound (with the where clause or with `<T: Quack>`) , Rust's precompilation can prepare the versions for you. When we use the trait object as a type, we defer to runtime checks.

```rust
fn this_thing_quacks(quackable: dyn Quack) {
	println!("This thing quacks!");  
	quackable.quack();  
}
```

By adding [`dyn`](https://doc.rust-lang.org/std/keyword.dyn.html) we tell the compiler that it will need to look up the data, and an associated table of its functions. If we knew the type at compilation time, we wouldn't need to look up associated functions because they would be known.

### Fn / FnMut (Function Trait Objects)

Closures and things that are callable implement one (or more) of three function traits in Rust. They are FnOnce, FnMut, and Fn. Closures that have data moved into them are actually like structs, with properties for their closed over values. For this reason they implement FnOnce (they can only be called once). 

**The function traits cascade:**
- Fn can be used anywhere an FnMut and FnOnce can
- FnMut can be used anywhere an FnOnce can
- FnOnce can only be used where an FnOnce is specified

*The rules for what implements the traits are as follows:*
- Fn - Accepts values that are owned or references as arguments
- FnMut - Accepts values that have mutable references
- FnOnce - uses move smenatics

This should not be confused with `fn` which is a function pointer. Function pointers are used to refer to functions whoes identity (and as a result, size) are not known at compilation time. A pointer to a function needs to be used in this case, just like a Box<> gives us a pointer because the contents of a box might not be known.

With all that known, let's go back to the closure we're trying to make.

The parameter type of Closure::wrap is outlined as follows: 

>A more direct version of `Closure::new` which creates a `Closure` from a `Box<dyn Fn>`/`Box<dyn FnMut>`, which is how it’s kept internally.

To satisfy this the specification of `Closure:wrap` let's first add that `Box`. And in that box we'll put a closure.

```rust
let cb = wasm_bindgen::closure::Closure::wrap(
	Box::new(
		|| {  
		    leptos::log!("Connected ok");  
		}
	)
);
```

The Rust compiler will throw an error here, asking for us to be more specific:

>  the trait `WasmClosure` is not implemented for closure `[closure@src/main.rs]`

We can tell the Rust compiler to treat our box as a specific type (which is will check against) with the `as Type` statement after the `Box`'s initialization.

```rust
let cb = wasm_bindgen::closure::Closure::wrap(
	Box::new(
		|| {  
		    leptos::log!("Connected ok");  
		}
	) as Box<dyn Fn()->()>
);
```

My hope is that now you'll look at this and read it as follows:

We're creating a closure but it needs to be stored in the heap. The value of the `Box` which specifies heap storage is a closure which doesn't have any arguments and does't close over any values. We can cast the internal type of the `Box` as a `dyn Fn()->()` because it's actual type won't be known until runtime and it accepts no arguments and returns no values (or returns a unit type `()`).

```rust
let cb = wasm_bindgen::closure::Closure::wrap(
	Box::new(
		|| {  
		    leptos::log!("Connected ok");  
		}
	) as Box<dyn Fn()->()>
);
```

Note that in a lot of cases we can use the turbo fish to specify type arguments `Box::new` does not accept any generic as an argument so we need to specify it after the fact.

Let's keep climbing out of this hole back up to where we started, with trying to create a js_sys::Function that we can pass into the connection handler as a reference.

The result of this is whole thing is that we have a closure, but we don't have a reference to js_sys::Function. In fact, we need `Some(js_sys::Function)`.

Here we'll take our callback closure, well get it as a reference, then we'll cast that reference as a js_sys::Function with the turbo fish. And of course, we wrap it all in `Some()`. 

```rust
Some(cb.as_ref().unchecked_ref::<Function>())
```

One additional thing that we need to do here, for the sake of Leptos, is to add the following:

```rust
on_cleanup(cx, move || {  
    drop(cb);  
});
```

Leptos has a clean up routine that it runs when a context is closed. We need to move our callback closure, which is actually a handle, to the cleanup function's callback closure. 

`on_cleanup` is being told, "Hey, when cx is cleaned up, run this closure!" In that closure we've moved our callback and passed it into drop(). This means that it'll be cleaned up in WASM's memory, and JavaScript land will prune the closure on its side as well.

Our whole connection callback looks like this:

```rust
let connect_to_database = move |_|{  
  
    // Grabbed a reference to window  
    let window = window();  
  
    // Got a factory to be able to make an open connection  
    let idb = match window.indexed_db() {  
        Ok(Some(idb_factory)) => idb_factory,  
        _ => return  
    };  
  
    let idb_open_request = match idb.open("my-database") {  
         Ok(idb_open_request) => idb_open_request,  
         _ => return  
    };  
  
    let ok_cb = wasm_bindgen::closure::Closure::wrap(  
        Box::new(|| {  
            leptos::log!("Connected ok");  
        }) as Box::<dyn Fn()->() >  
    );  
  
    idb_open_request.set_onsuccess(  
        Some(ok_cb.as_ref().unchecked_ref::<js_sys::Function>())  
    );  
  
    on_cleanup(cx, move || {  
        drop(ok_cb);  
    });  
  
    let error_cb = wasm_bindgen::closure::Closure::wrap(  
        Box::new(|| {  
            leptos::log!("Connected error");  
        }) as Box::<dyn Fn()->() >  
    );  
  
    idb_open_request.set_onerror(  
        Some(error_cb.as_ref().unchecked_ref::<js_sys::Function>())  
    );  
  
    on_cleanup(cx, move || {  
        drop(error_cb);  
    });  
  
    // You are here.  
  
};
```

So, here's where things get interesting. Our database connection is stored in the `result` property of our `IdbOpenDbRequest` if our connection was successful. What we want to do is create a signal so that we can store the `IDBDatabase` on success. It looks like our open database request may need to be used in a few scopes too. We can use Leptos signals to store this data.

```rust
let ( 
	idb_open_db_request, 
	set_idb_open_db_request 
) = create_signal::<Option<web_sys::IdbOpenDbRequest>>( cx, None );  

let ( 
	idb, 
	set_idb 
) = create_signal::<Option<web_sys::IdbDatabase>>( cx, None );
```

It's important that we use `Option` types here so that we have the ability to set a default value of None. 

We'll update our click handler closure with the `move` keyword, so that these signals will be moved into it when they're used. Keep in mind that signals support the `Copy` trait, so they'll be copied into the closurs without being moved from the scope they were defined it.

We'll also update the names of some of these variables so that they reflect their types and disambiguate from the new signals. There is a lot of idb this and idb that. 

I present, the start of our on click connect to db handler closure/callback:

```rust
let connect_to_database = move |_|{  
  
    let window = window();  

	// Guard assignment
	// idb was renamed to idb_factory
    let idb_factory = match window.indexed_db() {  
        Ok(Some(idb_factory)) => idb_factory,  
        _ => return  
    };  

	// Guard assignment
	// updated to now set the reactive value
    match idb_factory.open("my-database") {  
         Ok(new_idb_open_request) => set_idb_open_db_request
	         .set(Some(new_idb_open_request)),  
         _ => return  
    };
```

We need to update our callbacks for the database connection lifecycle to use our signals as well. In the `onsuccess` callback, we'll also need to pull the database connection out of the db connection requests `result`. 

```rust
let ok_cb = wasm_bindgen::closure::Closure::wrap(  
    Box::new(move|| {  
        
        leptos::log!("Connected ok");  

		// We'll get the request's value from the reactive system
        match idb_open_db_request.get() {  

			// If it is set we'll use it, referring to it herein
			// as ok_idb_open_request
            Some(ok_idb_open_request) => {  

				// We'll grab the result which in this context will
				// be an idb database. 
                match ok_idb_open_request.result() {  
		            
		            // If the result() was accessible
		            // it'll be a new_idb_connection
                    Ok(new_idb_connection) => {  

						// But this is from JavaScript so we have 
						// to unchecked_into with the Rust type.
                        let new_idb = new_idb_connection
	                        .unchecked_into::<web_sys::IdbDatabase>();  
                        
                        // We'll store this new connection in
                        // Leptos' reactive syste
                        set_idb.set(Some(new_idb));  
                    
	                    // We'll log the result from Leptos'
	                    // reactive system to confirm that it
	                    // worke as planned.
                        leptos::log!("{:?}", idb.get());  
                    },  
                    Err(_) => {}  
                }  
            },  
            None => {}  
        };  
  
    }) as Box::<dyn Fn()->() >  
);
```
>The above code is spaced wide and in a verbose syntax so that it is clear.

The rest of the callback contains a our on error handler, and a new onupgrade needed.

```rust
	let error_cb = wasm_bindgen::closure::Closure::wrap(  
	    Box::new(move || {  
	        leptos::log!("Connected error");  
	    }) as Box::<dyn Fn()->() >  
	);  
	  
	let upgrade_cb = wasm_bindgen::closure::Closure::wrap(  
	    Box::new(move || {  
	        leptos::log!("Doing database upgrade or setup");  
	    }) as Box::<dyn Fn()->() >  
	);  
	  
	match idb_open_db_request.get() {  
	    Some(idb_odbr) => {  
	        idb_odbr.set_onsuccess(  
	            Some(ok_cb.as_ref().unchecked_ref::<Function>())  
	        );  
	        idb_odbr.set_onerror(  
	            Some(error_cb.as_ref().unchecked_ref::<Function>())  
	        );  
	        idb_odbr.set_onupgradeneeded(  
	            Some(upgrade_cb.as_ref().unchecked_ref::<Function>())  
	        );  
	    },  
	    None => {}  
	}  
	  
	on_cleanup(cx, move || {  
	    drop(ok_cb);  
	    drop(error_cb);  
	    drop(upgrade_cb);  
	});
}
```

The on upgrade needed will fire when the database needs to be initialized or if the format of the database changes. This is where we will add our initialization code for the type of data stored in the database.

The interesting thing is that `onsuccess` will happen after the `onupgradeneeded`. As per the MDN documentation it looks like `onupgradeneeded` gets passed an event which we can use to extract the `event.target.result` out of.  

```javascript
// This event handles the event whereby a new version of
// the database needs to be created Either one has not
// been created before, or a new version number has been
// submitted via the window.indexedDB.open line above
// it is only implemented in recent browsers
DBOpenRequest.onupgradeneeded = (event) => {
  const db = event.target.result;
```
>From [# IDBOpenDBRequest MDN documentation](https://developer.mozilla.org/en-US/docs/Web/API/IDBOpenDBRequest)

We can update our closure with a parameter called event. We've changed the signature and size of the closure, which requres us to update the `as Box::<dyn Fn()->()>` to match. 

```rust
let upgrade_cb = wasm_bindgen::closure::Closure::wrap(  
    Box::new(move |event| {  
        leptos::log!("Doing database upgrade or setup");  
    }) as Box::<dyn Fn(Event)->() >  
);
```

We can't just write `Fn(Event) -> ()` and we need to add the type for the parameter to work with it. So how do we go about finding th type? We can go to the [documentation page for the method](https://developer.mozilla.org/en-US/docs/Web/API/IDBOpenDBRequest/upgradeneeded_event) and take a look at the event type listed. It is stated as [`IDBVersionChangeEvent`](https://developer.mozilla.org/en-US/docs/Web/API/IDBVersionChangeEvent). If I look that up but in Rust's required PascalCase `IdbVersionChangeEvent` I'll find this [`web_sys::IdbVersionChangeEvent`](https://rustwasm.github.io/wasm-bindgen/api/web_sys/struct.IdbVersionChangeEvent.html). We do need to add the feature to our cargo.toml as per the documentation as well. The feature is `IdbVersionChangeEvent`. By now you should be seeing a pattern of how we're progressing through solving this problem. We'll search for javascript examples (as Rust examples are few and far between), and then look up the web_sys equivalents.

Let's update the event with our expected type and the type in the `as Box::<>` part.

```rust
let upgrade_cb = wasm_bindgen::closure::Closure::wrap(  
    Box::new(move |event: web_sys::IdbVersionChangeEvent| {  
        leptos::log!("Doing database upgrade or setup");  
    }) as Box::<dyn Fn(web_sys::IdbVersionChangeEvent) -> ()>  
);
```

>**Important:** The upgrade callback will only run if the database has not been initialized or if it has a version change where the integer version number is greater than the existing version number. We haven't discussed version changes so for the time being you can change the name of the database to create a new one, always triggering the update callback.

You'll frequently run into issues where you won't know whats the type is from the JavaScript side of things. In this case, we want to work with `web_sys::IdbVersionChangeEvent.target()` but we don't know what the return type of `target()` is. What I'll often do is log the value to the browser's console and look for hints for the type that I should cast the value into.

```rust
let upgrade_cb = wasm_bindgen::closure::Closure::wrap(  
    Box::new(move |event: web_sys::IdbVersionChangeEvent| {  
  
        match event.target() {  
            Some(event_target) => {  
                leptos::log!("{:?}", event_target);
            },
			None => {}  
        }    
	}) as Box::<dyn Fn(web_sys::IdbVersionChangeEvent) -> ()>  
);
```

The above logs `EventTarget { obj: Object { obj: JsValue(IDBOpenDBRequest) } }` to the console. This tells me that I can cast the value by calling `unchecked_into::<IdbOpenDBRequest>()`. It's important to note two things here; 1) The Rust trait has different casing than the JavaScript object type listed in JsValue; 2) You will likely need to enable the feature for that trait in web-sys.

We'll continue this same pattern to get the result, cast the result, and we'll be left with our database which we can initialize as a store for some form of data:

```rust
let upgrade_cb = wasm_bindgen::closure::Closure::wrap(  
    Box::new(move |event: web_sys::IdbVersionChangeEvent| {  
  
        leptos::log!("Doing database upgrade or setup");  
        
        match event.target() {  
            Some(event_target) => {  
                let open_request = event_target
	                .unchecked_into::<web_sys::IdbOpenDbRequest>();  
                match open_request.result() {  
                    Ok(newly_opened_idb) => {  
                        let newly_opened_idb = newly_opened_idb
	                        .unchecked_into::<web_sys::IdbDatabase>();  
                        // Do things here  
                    }  
                    Err(_) => {}  
                }
			}, 
		   None => {}  
        }    
	}) as Box::<dyn Fn(web_sys::IdbVersionChangeEvent) -> ()>  
);
```
>Recall that we can use `web_sys::IdbDatabase` beacuse `web_sys` is brought into scope via `use leptos::*`

### Structuring the database

We now have a newly opened database in `newly_opened_idb`. We need to setup some tables in the database. IndexedDB doesn't use tables though. They use object stores. Each object in an object store is associated with a key. An object store can use a key path (you tell it how to source the key from the object being stored, `key_path`) or from a key generator (`auto_increment`). Object stores can contain objects and primitive data. If they contain objects, they may also have indexes which can enforce specific rules, and make queries faster.

### Writing data


### Querying data


### Creating stores and indexes as storage bounds

