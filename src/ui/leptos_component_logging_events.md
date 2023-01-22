
# Leptos components and logging events

## What we know
- Leptos makes distinctions between which things are static (will not change) and which are dynamic (can change)

## What we'll learn
- How we can make our application dynamic through client side events

## The Lesson
We've worked with static templates so far but it would be great if we can start to interact with our application. It would be wonderful if we could do something and see that change happen before our very eyes. 

The secret to achieving this is in the "if we could do something," part of what I just wrote.

When something happens in the browser—a key is pressed, a button is clicked, the mouse is moved—an event is fired. A piece of functionality gets called as part of the browser's JavaScript runtime declaring what happened along with some information about that event. 

If we were writing these applications in JavaScript, the native language of the web's interactive runtime, we could declare a function as a callback (will be called when) an event happens, reacting to the interaction. Unfortunately, we can't directly do that because we're writing Rust which becomes WASM. But never fear, we have some pretty great tools to help us around this shortcoming.

Leptos comes bundled with a crate called `wasm-bindgen`. WASM bindgen which acts as an interface, allowing Rust to call JavaScript features and vise-versa. There are some crates that add extra helper fuctionality on top of that like `web_sys` which provides types and wrappers to interact with web and browser APIs.

Let's refresh our memory and look at our lucky number Leptos component:

```rust
#[component]  
fn LuckyNumber(cx: Scope, the_lucky_number: i32) -> Element {  
    view!{  
        cx,  
        <div>  
            <p>"Today's lucky number is " {the_lucky_number}</p>  
        </div>  
    }  
}
```

Let's add a button to this which will eventually print a message in the client's web browser console (abbreviated for clarity).

```rust
	<div>  
		<p>"Today's lucky number is " {the_lucky_number}</p>  
		<button>"Your Secret Lucky Number"</button>  
	</div>  
```

Leptos makes it really easy to add event handlers—functionality triggered by events. We simply write 'on:' followed by the name of the event, and the name of the handler. 

```rust
<button>"Your Secret Lucky Number"</button>  
```

... is made interactive as follows...

```rust
<button on:click=whisper_in_the_console >"Your Secret Lucky Number"</button>  
```

But this is incomplete. We've written the name of a handler that we made up, which we're calling `whisper_in_the_console` but we haven't declared it anywhere. Rust doesn't know what this means. To fix this we'll declare this variable and assign a value to it which is a closure, a function which can run, but which is stored in a variable.

```rust
#[component]  
fn LuckyNumber(cx: Scope, the_lucky_number: i32) -> Element {
	let whisper_in_the_console = |_|{  
	    web_sys::console::log_1(&42.into());
	};  
	view!{  
	    cx,  
	    <div>  
	        <p>"Today's lucky number is " {the_lucky_number}</p>  
	        <button on:click=whisper_in_the_console >
		        "Your Secret Lucky Number"
			</button>  
	    </div>  
	}
}
```

Let's break down what this assignment means. We'll be less verbose in later chapters, but it's important to start thinking in terms of what we're communicating to the compiler. This will help you become a native speaker of the Rust language. :)

```rust
let whisper_in_the_console = |_|{  
    web_sys::console::log_1(&42.into());
};  
```

And like that, now when we click on the button we get the number 42 printed out in the console!

> SUPER IMPORTANT: You do not need to understand the body of this closure to continue. There will be numerous examples of event handlers in future chapters. I am specifically using this example so that you can log something to the browser's console. Further reading will expose you to some advanced concepts about the rust programming language.

#### Event handler closure signature

First let's look at how we even declared the closure and assigned it as a value to a variable. In case you're curious, this is what it means when functions are first class citizens. You might hear programmers mention this idea. It just means that a function can be assigned as a value and passed around.

Here's what the first line says:

```rust
	let whisper_in_the_console = |_|{ 
		// this is a comment, letting you know that we removed the
		// event handler's body which we'll step through soon
	};
```

`let` (let it be/declare) that `whisper_in_the_console` is equal to (there for, it is) a closure which accepts an argument which is never used (marked by the `_` /underscore character ).

 Event handlers must be a function with a single parameter for the event that triggered the handler (the respondant). In our case we're not using that argument, so it's convention to write an underscore in its place instead of giving it a name.

#### Event handler body

There is a lot to unpack in this single line of code:

```rust
web_sys::console::log_1(&42.into());
```

The first bit is the crate's, `web_sys`. Crates are what Rust nomenclature uses when refering to packages or libraries. `web_sys` can be used in this scope because it was made public and brought into the leptos crate's scope. By writing `use leptos::*;` at the top of our main.rs file we're bringing in everything that is public in Leptos'.

> You may ask "How did you know to use web_sys?" To be honest, it's a matter of asking around, seeing other examples and discovering useful crates. Some are very popular like web sys and you will use them because they're used in the libraries or frameworks that you're consuming. Developers don't magically know about all of these tools. It takes a lot of exposure to the community to develop this awareness and understanding. Stay cool about it. The knowledge will come with time. For now, trust the process.

When we write `web_sys::console::log_1` we're saying, "Use the log_1" function in the console module, from the web_sys crate. 

This is the `web_sys` crate's equivalent of calling console.log in the JavaScript console. In JavaScript you can include as many arguments as you want when calling the log function. This is what programmers call a "variadic" function. Rust does not allow this. To accommodate multiple arguments, console has functions for log, log_1, log_2, log_3 and so on. We have to specify the number of arguments in rust, which is sort of awkwardly, but clearly, done with the suffixed number.

The log function in Rust accept a reference to JsValue data type. Recall that we talked about how types are constraints for possible values. JsValue exists because JavaScript doesn't have any type system what so ever. Rust developers needed to make a type that could interoperate safely with JavaScripts untyped world. 

To convert our number (an i32 litera) to a JsValue type we call the `into()` method on it, which will convert it ot the required type of the log_1 signature which happens to be a JsValue. The log method also requires that we do not pass the actual owned value. It is expecting a reference to data. We can add an ampersand to specify that this is a reference.
And like that we get...

```rust
web_sys::console::log_1(&42.into());
```

Call web_sys's console module's log 1 function which we'll pass a single argument to using the number 42 which we'll convert to the accepted/required type and pass it as a reference.





