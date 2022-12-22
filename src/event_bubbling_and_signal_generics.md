# Event Bubbling and Signal Generics

## What we know
- We can monitor activity in the browser by responding to events
- We can add functions that run when events happen to Leptos components. These functions are called event handlers. Event handlers added to Leptos components are also added to their DOM node counterparts and connected behind the scenes with Leptos' use of `wasm_bindgen` making their use transparent.
- The syntax for an event handler is similar to adding a property to a component, but with a prefix of `on:` followed by the event name. e.g. `<LeptosComponent on:click=my_event_handler />`.
- Event handlers (event callbacks) are closures (a one time function that encapsulates the values used in it) that are assigned to a variable. e.g. `let my_event_handler = |event|{ ... }`.
- the `move` keyword that can preceed a closure's parameters, indicating that variables used in the closure's function body will be moved into the closure itself and removed from the current scope as if they were passed into a function. Variables that are types that support the Copy trait will automatically be copied and will still be available in the current scope. 
- Signal read and write components support Copy.

## What we'll learn
- How events can be captured in parent components
- What generics are in Rust's type system at an introductory level

## The Lesson
We've established that the document object model (DOM) is a tree like representation of DOM nodes which is a browsers data structure containing information about what's on a web page. When events happen in a browser, the event will triggered at the lowest, most specific, DOM node. That event will bubble up until it's handled or prevented from continuing. Bubbling up means that the original event will be given the opportunity to handled by the originating element's parents, one at a time, until it reaches the top of the DOM tree.

If we take the following HTML:

```html
<html>
	<body>
		<div id="application">
			<div class="button-container">
				<button>Click me</button>
			</div>
		</div>
	</body>
</html>
```
Clicking on the click me button would create the intial event. `on:click` events handlers on this element will run first. Then, `on:click` handlers for the ".button-container" would run, and so forth.

This means that you can place handler logic on a parent component that has multiple children who emit events. For example, you could use this way of thinking to run a validation script on a form any time any input field is changed. Or, imagine if you wanted to capture any click as some form of analytics. You could setup a click handler in your main app which will capture all events that bubble up to it.

### Using bubbled events to update Leptos component properties

The following is an example of how we can move the lucky number value's handler out of the component and into a new Leptos component we're calling `RadApp`.

To start, we'll create a new component called `RadApp`, add it to the mount_to_body `view!`, and setup our `LuckyNumber` component as a child.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <RadApp />  
        }  
    })  
}  
  
#[component]  
fn RadApp(cx: Scope) -> Element {  
    view!{  
        cx,  
        <LuckyNumber the_lucky_number=12 />  
    }  
}  
  
#[component]  
fn LuckyNumber(cx: Scope, the_lucky_number: i32) -> Element {  
    view!{  
        cx,  
        <div>  
            <p>"Today's lucky number is " {the_lucky_number}</p>  
            <button>"Pick a better number"</button>  
        </div>  
    }  
}
```

`LuckyNumber` has a button that we want to activate. That event will bubble up so we can put the on click handler on the Leptos component instead of on the button, like we did previously.

Let's add the `on:click` and we'll use the leptos log macro to write a message to the browser console.
```rust

#[component]  
fn RadApp(cx: Scope) -> Element {  
	let update_the_lucky_number = |_|{  
	  leptos::log!("We should be updating the lucky number");  
	};
	  
    view!{  
        cx,  
        <LuckyNumber on:click=update_the_lucky_number the_lucky_number=12 />  
    }  
}  
```

Rust's compiler may complain saying ` cannot find type MouseEvent in this scope` followed by `consider importing this struct`:

```rust
use crate::web_sys::MouseEvent;
```

You can literally copy and paste this into your main.rs file right after `use leptos::*`.

Now we need to create our signal so that we can read and update the data over time. We need to register it in our scope.
`
```rust
#[component]  
fn RadApp(cx: Scope) -> Element {  
	let (value, set_value) = create_signal(cx, 12);
	let update_the_lucky_number = |_|{  
	  leptos::log!("We should be updating the lucky number");  
	};
	  
    view!{  
        cx,  
        <LuckyNumber on:click=update_the_lucky_number the_lucky_number=12 />  
    }  
}  
```

We might intuitively think, "Hey, we can just put value where the number 12 previously was as a property of `LuckyNumber`," like this:

```rust
 <LuckyNumber on:click=update_the_lucky_number the_lucky_number=value />  
```

But this won't work. There's a problem. Value is a `ReadSignal`, and our property is supposed to be a 32 bit integer. We can see this in the function definition of the `LuckyNumber` component.

```rust
fn LuckyNumber(cx: Scope, the_lucky_number: i32) -> Element {
```
> `the_lucky_number` is supposed to be any 32 bit integer, denoted by `i32`.

Rust's compiler will actually give you an error showcasing what was expected and what it received:

```
 note: expected type `i32`
       found struct `ReadSignal<{integer}>`

```
> All of these errors will appear in the terminal that you typed `trunk serve` in.

The type ReadSignal<{integer}> probably looks a little bit weird to you. You might ask yourself, why is there a bunch of stuff after the type's name? What does `<{integer}>` mean?

Recall that functions have parameters which follow the function the function name and are encapsulated by parenthesis.

```rust
// this is pseudo code to show you the structure of the signature
fn function_name(parameter_name: SomeType)
```

Types have parameters called generics which follow the type name and are encapsulated by angle brackets.

```rust
SomeType<SomeGenericType>
```

Generics allow us to configure a type with additional types. 

For example, let's say that we have a bunch of containers and we're preparing our lunch for the day. We can store things in all of the containers and we can eat the contents from each container. All of the containers, though different sizes and colours, share the same type. They're containers. But some containers may contain liquids and others will contain solids.
If we were to eat or drink from one of the containers we could use the type system to guarantee that we wouldn't try to drink our sandwich or chew our milk by using parameter types like, `Container<Solid>` and `Container<Liquid>` respectively. 

If we pop back over to Leptos, we can see how the context (scope) is similar. If we think about our wearhouse that we use to store and retrieve value from, we need some way to embed what type of values those are. 

If we create_signal with an integer like an i32, we're saying that the ReadSignal is actually `ReadSignal<i32>`. This tells rust, "Hey, this ReadSignal works like any other read signal, but when you get the contents out of it, it'll absolutely be a valid i32".

```rust
#[component]  
fn LuckyNumber(cx: Scope, the_lucky_number: ReadSignal<i32>) -> Element {  
    view!{  
        cx,  
        <div>  
            <p>"Today's lucky number is " {the_lucky_number}</p>  
            <button>"Pick a better number"</button>  
        </div>  
    }  
}
```

We need to update our RadApp component to pass the signal to the component as well. Our whole working example looks like this.

```rust
use leptos::*;  
use web_sys::MouseEvent;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <RadApp />  
        }  
    })  
}  
  
#[component]  
fn RadApp(cx: Scope) -> Element {  
    let (value, set_value) = create_signal(cx, 12);  
    let update_the_lucky_number = move|_|{  
      set_value(42)  
    };  
    view!{  
        cx,  
        <LuckyNumber on:click=update_the_lucky_number the_lucky_number=value />  
    }  
}  
  
#[component]  
fn LuckyNumber(cx: Scope, the_lucky_number: ReadSignal<i32>) -> Element {  
    view!{  
        cx,  
        <div>  
            <p>"Today's lucky number is " {the_lucky_number}</p>  
            <button>"Pick a better number"</button>  
        </div>  
    }  
}
```  