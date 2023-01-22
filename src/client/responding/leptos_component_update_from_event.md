# Leptos components updating from events

## What we know
- Leptos components are templates that can be added to a web page's document object model DOM as nodes, with separate text nodes for dynamic data used in text strings.
- Leptos components can have fuctions run in response to events on a given component.
- `wasm-bindgen` and other supporting crates work as bridges between our Rust code and the browser's JavaScript runtime.

## What we'll learn
- Updating the DOM in response to events.

## The Lesson

In our previous example we created a silly Leptos component that displays some text and has a button that, when clicked, echoes a number to the console.

We'll take this example and and instead of echoing 42 the console, we'll replace the lucky number with it.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <LuckyNumberCounter the_lucky_number=12 />  
        }  
    })  
}

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

First, let's adjust the names of some of the event callback and put a placeholder into the event handler (or callback) body.

```rust

#[component]  
fn LuckyNumber(cx: Scope, the_lucky_number: i32) -> Element {
	let update_the_number = |_|{  
	    // This functionality is unknown.
	};  
	view!{  
	    cx,  
	    <div>  
	        <p>"Today's lucky number is " {the_lucky_number}</p>  
	        <button on:click=update_the_number >
		        "I need a more lucky number"
			</button>  
	    </div>  
	}
}
```

We know from previous lessons that Leptos component templates are setup statically when Leptos starts up. Given that this is the case we may ask ourselves, how can we have dynamic content if the template is static? 

The secret to solving this problem involves combining two features of Leptos, one which we've seen before and one which is new.
1) How Leptos separates static and dynamic content
2) Signals which can be converted into data

### 1) Separation of static and dynamic template components
We saw from the previous example that Leptos differentiates parts of our template from components that are variable. By doing this, it can zip together data that changes with the static template that does not change.

```rust
<p>
	"Today's lucky number is " {the_lucky_number}
</p>  
```

The above could be interpreted as follows:

```
[STATIC DATA] {DYNAMIC DATA} [STATC DATA]
```
```
[<p>"Today's lucky number is "]  {the_lucky_number} [</p>]
```

By doing this Leptos can reuse the template while leaving a holes, like fields in a form, for variable data.

### 2) Signals
The problem with our Leptos component is that `the_lucky_number` is a variable who's value is defined outside of its component template. It's value is provided when the whole system starts up and the component is mounted to the body as shown in the "`fn main()`" main function.

Unfortunately our variable `the_lucky_number` doesn't have an opportunity to be updated or changed. It's been used in our template and it has been consumed. Rust has some very intereting rules about data. 

#### The idea of movement and scope
In a lot of programming languages, you can pass data into a function and then also use it elsewhere. In Rust if you pass data into a function it's considered to have been moved into the function. It has left the scope, or the space, which you were operating.

For example, consider a time where I gave my friend a sandwich and asked them to paint a picture of it for me—it was a beautiful sandwich. If I gave the sandwich to my friend, I no longer have it. It's been _moved_ into their hands. They may give it back to me, but until then, I won't have it. If you're working in Rust and see statements like, "such and such has moved," this is what that means.

If Rust can make a copy of the data, it'll do so to get around the issue of moved values, but that only happens with simple data types like numbers. There are exceptions and a lot to explain with what is called the copy trait, but we won't get into that here. The idea of 'copy' is important to how Leptos allows dynamic content though.

### Knowing what we are actually moving
Let's go back to this sandwich example. Perhaps I don't want to relinquish my sandwich. I could provide a reference to it and my friend could look at it to make the painting. Though, they wouldn't be allowed to touch it. They are only allowed to look at it. In this case, I'm not losing my sandwich, but Rust will prevent me from changing it while someone is referencing it. Rust won't allow me to take a bit of my sandwich while I told my friend they could look at it to make the painting. Independent of how hungry I am. I could actually set up a plinth, place my sandwich on top, and allow a whole class of artisans to paint my sandwich. 

Rust also allows me to loan out my sandwich by providing a mutable reference, but if I do that, I can't touch it, and no one else is allowed to reference it. It would be as if I told my friend, "You can paint my sandwich and organize the lettuice and tomato so that it makes a nice composition." No one could safely paint that sandwich because my friend might still be moving parts of it around.

References in rust are achieved by adding an ampersand before a value. `42` is a number. `&42` is a reference to that number. We can dereference or follow the reference to the original by placing an asterisk before a variable containing a reference. We'll expand on this later and explain how you references, and when to use them.

To summarize, the rules are:
- We can move an owned value (the sandwich)
- We can create and move one or more references to the sandwich
- We can create one single mutable reference, but we can not also have regular references if we do

### The borrow checker
The above two concepts are key parts of what we call the Rust borrow checker. The purpose of the borrow checker is to make sure that our system (application) has predictable access to data. To do this, it tracks where we move things and how we reference them, to guarantee that we haven't inadvertantly written something stupid that will break our program or create secrity vulnerabilities. And trust me, we will write things like that. The borrow checker is your friend and asks you to do your best work. You will learn to appreciate how amazing it is in time.

Now that we know this, we can see how what felt like a simple problem to solve is actually pretty complicated. If we move a value into a component, it's gone. We can't update a value that doesn't exist as a result of some event. It might take some time to wrap your mind around this idea. It'll feel uncomfortable at first.

### Leptos' solution 
What we really need is some sort of special variable. We need something that we can put in the template which can be notified when its value changes, and something that can transparently act as its value. 

Imagine if we had a wearhouse of data who we could call and ask for data. "Hey, I need the value of Aisle 2 bin 4." If we had the location of the data, we could always ask the wearhouse for whatever is stored there.

Or what if we could ask them to store something and they'd do so, responding with its location in the wearhouse. "Can you store this gigantic novelty taco beanbag chair for me?" we'd ask. "Sure, and it's in aisle 2 bin 5," they'd respond. 

This is what signals do. Signals are a formalized way of being able to communicate with the wearhouse (which in the context/scope in Leptos) to store data and retrieve data. When data changes, leptos can follow where it is being used, and update those usages accordingly. 

I introduced the idea of copy earlier because the signals are actually indexes, storage positions in the context, which will be duplicated as you use them. This allows you to move a signal into a closure which will be handling an event while still using it in the view template. 


### Reactivity in action 

To create a signal, we need to call the function `create_signal()` and provide a scope (or context) as the first argument, and the default value as the second. It returns a tuple, a set of two values, which we can immediately give names to so that we can use them in the scope of our function.
The first part of the signal allows us to retrieve a copy of the value from the wearhouse. The second part of the signal allows us to set the value at the signal's location.

```rust
let (value, set_value) = create_signal(cx, the_lucky_number);  
```

The above is called destructuring. We could also have written in the long form way but it is actually harder to read and requires additional temporary assignments like `lucky_number_signal`. 

```rust
let lucky_number_signal = create_signal(cx, the_lucky_number);  
let value = lucky_number_signal.0;
let set_value = lucky_number_signal.1;
```
> Note that `.0` and `.1` are properties on the `lucky_number_signal`. They're indexes for the first and second component in the tuple.

Now that we have a signal we can update our callback and move the set_value signal into it. Not the addition of the move keyword before the closure's pipes which encapsulate it's properties, and the underscore which denotes that it will be provided a property when called, but we won't be using it.

```rust
let update_the_lucky_number = move|_|{  
        set_value(42);  
    };  
```

And in the view template we can update our previous value with our signal which can be used to derive the value.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <LuckyNumber the_lucky_number=12 />  
        }  
    })  
}  
  
#[component]  
fn LuckyNumber(cx: Scope, the_lucky_number: i32) -> Element {  
    let (value, set_value) = create_signal(cx, the_lucky_number);  
  
    let update_the_lucky_number = move|_|{  
        set_value(42);  
    };  
    view!{  
        cx,  
        <div>  
            <p>"Today's lucky number is " {value}</p>  
            <button on:click=update_the_lucky_number >"Pick a better number"</button>  
        </div>  
    }  
}
```

The coolest part about this is that the signal is responsible for updating itself on the web page if it's value changes. LuckyNumber doesn't run again to create a new template. Leptos updates that special little text node, where `{value}` is used.