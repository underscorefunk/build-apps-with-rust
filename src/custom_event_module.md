# Custom Event Module 

## What we know

- Data can be assocaited with custom events
- There's a lot of boiler plate in with custom events

## What we'll learn

- How to turn our code into a module that we can reuse

## The lesson

### Module basics

Rust has the ability to create modules to encapsulate code. It allows you to expose parts of the code to the outside world, while keeping other parts of the code private to the module. 

A module is defined by using the key word `mod` followed by the name of the module and curley braces which encapsulate the code in a module.

```rust
mod my_module {  
	pub fn hello_world() {
		println!("Hi");
	}
	fn you_cant_call_me() {
		println!("Seeecrets");
	}
}
```
> We've seen this pattern of `set the context` then `noun` the `content/definition` all over the place. These patterns repeat all over the place.

The module can be used in the scope in which it is defined without any extra work. We must prefix functions with 'pub' in a module to specify that they are public. Functions in a module have access to private functions that are within the module because they're all in the same module scope. Calling a function inside a module requires you to specify the module's namespace followed by two colons and the function name. 

```rust
mod my_module {  
	pub fn hello_world() {
		println!("Hi");
	}
	fn you_cant_call_me() {
		println!("Seeecrets");
	}
}

fn main() {  
    // ✅ We can call this public function
    my_module::hello_world();
    
    // ❌ We can't call this private function
    my_module::you_cant_call_me();
}
```

#### Module files

Modules can be moved to their own files as well. 

1) Create a my_module.rs file in the ./src folder fo your application
```rust
pub fn hello_world() {
	println!("Hi");
}
fn you_cant_call_me() {
	println!("Seeecrets");
}
```

2) Bring it into scope in your ./src/main.rs file with `mod my_module` which will automatically hook up the file my_module.rs
```rust
mod my_module;

fn main() {  
    my_module::hello_world();
}
```

Now, while you can do this, it's not ideal. 

### lib.rs

The preferred orgnization is to create a lib.rs file, which is the entry point to your crate's functionality. It is called lib because it is a library of functionality and isn't intended to be directly executed. We'll deal with the details of this later. For this lesson we're going to create the module in the same main.rs file as your example application.


## The refactor

We started off with the following from a previous lesson:

```rust
use leptos::*;  
use crate::wasm_bindgen::JsValue;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <SandwichShopApp />  
        }  
    })  
}  
  
#[component]  
fn SandwichShopApp(cx: Scope) -> Element {  
    let log_order = |event : web_sys::Event| {  
        leptos::log!("Our custom event happened");  
  
        let bread_type = event  
            .unchecked_into::<web_sys::CustomEvent>()  
            .detail()  
            .as_string()  
            .unwrap_or(String::new());  
  
        leptos::log!("{:?}", bread_type );  
    };  
    view! {  
        cx,  
        <BltSandwich on:orderSandwich=log_order />  
    }  
}  
  
#[component]  
fn BltSandwich(cx: Scope) -> Element {  
    let dom_node_ref = NodeRef::new(cx);  
  
    let trigger_order_sandwich_event = move |event| {  
  
        let mut event_config = web_sys::CustomEventInit::new();  
        event_config.bubbles(true);  
        let bread_type = JsValue::from("Canadian Rye");  
        event_config.detail( &bread_type );  
        
        let event = web_sys::CustomEvent::new_with_event_init_dict(  
            "orderSandwich",  
            &event_config  
        );  
  
        match event {  
            Ok(event) => {  
                match dom_node_ref.get() {  
                    None => {}  
                    Some(dom_element) => {  
                        match dom_element.dispatch_event(&event) {  
                            Ok(_) => { leptos::log!("Custom event sent") },  
                            Err(_) => { leptos::log!("Failed to send") }  
                        }  
                    }  
                }  
            }  
            Err(_) => {}  
        }  
        
    };  
    view! {  
        cx,  
        <div _ref=dom_node_ref>  
           <h3>"BLT Sandwich"</h3>  
            <button on:click=trigger_order_sandwich_event>  
                "Order Sandwich"  
            </button>  
        </div>  
    }  
}
```

We'll start by making a module.

```rust 
mod component_custom_event {
	
}
```

For the time being we'll remove the complexity of dealing with the event data, omitting the following lines:

```rust
let bread_type = JsValue::from("Canadian Rye");  
event_config.detail( &bread_type );  
```

How it's time to start moving things into the module and generalizing them or making them configurable. 

I could see myself annotating the following code with a comment like `//first create a custom event`. 

```rust
	let mut event_config = web_sys::CustomEventInit::new();  
	event_config.bubbles(true);  
	
	let event = web_sys::CustomEvent::new_with_event_init_dict(  
		"orderSandwich",  
		&event_config  
	);  
  
```

This immediately tells me that there's a name I can give to these lines that summarizes them. We're creating a new custom event. We do need to give this new event a name, which instantly makes me think, "Name is a parameter!" We also know that `new_with_event_init_dict()` returns a Result type, which we handled before with our match statement. 

Let's start by stubbing out the definition:

```rust
mod component_custom_event {
	fn new(name: &str) -> Result<web_sys::CustomEvent, JsValue> {  
		// do stuff
	}
```

If this worked we could use `component_custom_event::new("orderSandwich")` and we should get what we expect to continue in our event handler.

A module is a separate scope. It acts in a similar way to main.rs, which has it's own scope. Rust is very good at being congruent like that.  web_sys and JsValue aren't defined in the module. To fix this we'll add some use statements.

```rust
use leptos::*;  //web_sys is imported as part of leptos's prelude
use leptos::wasm_bindgen::JsValue;
```

Let's copy the code block in as the body:

```rust
mod component_custom_event {
	use leptos::*;  
	use leptos::wasm_bindgen::JsValue;

	fn new(name: &str) -> Result<web_sys::CustomEvent, JsValue> {  
	    let mut event_config = web_sys::CustomEventInit::new();  
	    event_config.bubbles(true);  
	    let event = web_sys::CustomEvent::new_with_event_init_dict(  
			"orderSandwich",  
			&event_config  
		);  
	}
}
```

And, we need to hook up our property so that it's arguments are used in the events configuration. To do this we need to replace the literal "orderSandwich" with name. Now the value of the `name` function parameter will be used as the event's `name`, passed as the first argument to `new_with_event_init_dict()`.

```rust
mod component_custom_event {
	
	fn new(name: &str) -> Result<web_sys::CustomEvent, JsValue> {  
	    let mut event_config = web_sys::CustomEventInit::new();  
	    event_config.bubbles(true);  
	    let event = web_sys::CustomEvent::new_with_event_init_dict(  
			name, 
			&event_config  
		);  
	}
}
```

But we're not quite done here. We have an assignment for the last expression with `let event =`. And the last expression has a semicolon `;` at the end. This would result in the new function returning a unit type, written as `()`. If we want to return the event we could write `event` at the end, without a semicolon, so that it would be the "last word" in the function. Recall that Rust is expression based and the last open expression is used as the return of functions and scope blocks (unless you write `return` and provide it something to explicitly return). Let's remove the assignment and semicolon, and we're done with this one.

```rust
mod component_custom_event {
	
	fn new(name: &str) -> Result<web_sys::CustomEvent, JsValue> {  
		// configuration
	    let mut event_config = web_sys::CustomEventInit::new();  
	    event_config.bubbles(true);  
		// generation
	    web_sys::CustomEvent::new_with_event_init_dict(  
			name, 
			&event_config  
		)
	}
}
```

You might be tempted to try to do some form of chaining or nesting to make this even smaller. But `CustomEventInit::new()` returns a value that we need to mutate. It's the most clear to separate out the configuration stage from the custom event generation stage.

So now, my custom event dispatcher/handler looks like this:

```rust
let trigger_order_sandwich_event = move |event| {  
		
		component_custom_event::new("orderSandwich");
  
        match event {  
            Ok(event) => {  
                match dom_node_ref.get() {  
                    None => {}  
                    Some(dom_element) => {  
                        match dom_element.dispatch_event(&event) {  
                            Ok(_) => { leptos::log!("Custom event sent") },  
                            Err(_) => { leptos::log!("Failed to send") }  
                        }  
                    }  
                }  
            }  
            Err(_) => {}  
        }  
        
    };  
```

I'm looking at this and that whole `match event` block really looks like it summarizes as "send event". In fact, it feels like that's what I'm doing with this whole thing. I'm just dispatching a custom event on a specific node, through it's reference.

Maybe what I'm looking for is something like this:

```rust
let trigger_order_sandwich_event = move |event| {  
	component_custom_event::dispatch("orderSandwich", dom_node_ref);
}
```

Yeah, that's starting to look good! That says what I want to happen.

Now let's write the how. We'l start with defining a new function in the module. The name is going to pass right through, and we'll accept a NodeRef as a parameter. We know this because `NodeRef::new(cx)` returns a NodeRef type. If you got this wrong, Rust will actually inform you, "Oh, you tried to use a NodeRef where your dispatch method was expecting a (whatever type you used)." Our return type will be the same return type as `EventTarget.dispatch_event()`. We'll try to not deviate from the interface used in the standard methods. This will help us use these interchangably in the future.

```rust
// in mod component_custom_event {
pub fn dispatch( name: &str, target_ref : NodeRef) -> Result<bool, JsValue>{

}
```

Now we need to create our new event from the name, and we need to send the event with our node ref. Our node ref needs to be converted into a target because we can only call dispatch_event methods on `EventTarget` type values.

And so we create a new event:

```rust
let event = new(name);  
```
> We can just write `new` because `new` is defined in the local module scope! This is a great example of why modules are so convenient. They're like structs that have no data and only have class methods.

And we create our target from the reference:

```rust
let target = target_ref.get();  
```

Now, here's a really cool part. `event` is a `Result` and `target` is an `Option`. We know this because we defined the return type for the `new()` function. We can look up the return type of NodeRef.get(). We only want to dispatch the event if our event is valid and we have a target to send it on. Rust allows you to create tuples (groups of values where their type is known at specific locations) which we can use in matchs statements. They're like super powered pattern matching if statements.

We can create a `match` for a tuple with `event` and `target` to do something if both are Ok() and Some() respectively!

Take a look at this.

```rust
match (event, target) {  
	// We are matching on the Result and Option enums
	// and we're destructuring, all in one step!
	( Ok(event), Some(target) ) => target.dispatch_event(&event),  
	// The underscore indicates any other option that didn't match.
	// You can think of it as any other possible value that is a 
	// valid value within the type (recall that types jsut define 
	// the bounds of valid values)
	(_,_) => Err(JsValue::null())  
}  
```

Out whole method is finished!

```rust
pub fn dispatch( name: &str, target_ref : NodeRef) -> Result<bool, JsValue>{  
  
    let event = new(name);  
    let target = target_ref.get();  
  
    match (event, target) {  
        ( Ok(event), Some(target) ) => target.dispatch_event(&event),
        (_,_) => Err(JsValue::null())  
    }  
}
```

A new things to note here is that match is the last statement in the dispatch method. The result of will be used as the return value. In our match arms, we don't include semiconons because we want those match arms to become the evaluated value of the match statement, which becomes the evaluated value of the dispatch method. It sounds complicated at first, but if you take your time to read it carefully it'll click and the beauty of this will shine through.

There are also a few interesting Rust syntax things here that might have you scratching your head.

1) We use `event` in `match (event,target)` but then we also use `event` in the match arm's destructuring statement `( Ok(event), Some(target) )` and we use `&event` in the match arm's body. We can do this because we're actually reassigning `event` to a different value as we go along. This is called `variable shadowing`. We can use `event` to evaluate the match arms. When evaluating the body of the match arm, Rust will destructure and assign the values stored in the `Ok()` and `Some()` enums to their `event` and `target` names respectively.
2) We've removed the curley brances from the branch arms. Rust allows us to drop curley brances for match arms if the contents of an arm's body is a single statement. It just helps keep visual clutter down.

And like that, we're done.

If we look at our whole component, it's very easy to see the behaviour. We're able to focus on what is happening and not how it's happening. This is the power of declarative code. It allows our mind to think at one level of detail.

```rust
use leptos::*;  
  
mod component_custom_event {  
    use leptos::web_sys;  
    use leptos::wasm_bindgen::JsValue;  
  
    fn new(name: &str) -> Result<web_sys::CustomEvent, JsValue> {  
        // configuration  
        let mut event_config = web_sys::CustomEventInit::new();  
        event_config.bubbles(true);  
        // generation  
        web_sys::CustomEvent::new_with_event_init_dict(  
            name,  
            &event_config,  
        )  
    }  
  
    pub fn dispatch( name: &str, target_ref : NodeRef) -> Result<bool, JsValue>{  
  
        let event = new(name);  
        let target = target_ref.get();  
  
        match (event, target) {  
            (Ok(event), Some(target)) => {  
                target.dispatch_event(&event)  
            },  
            (_,_) => Err(JsValue::null())  
        }  
    }  
}  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <SandwichShopApp />  
        }  
    })  
}  
  
#[component]  
fn SandwichShopApp(cx: Scope) -> Element {  
    let log_order = |event: web_sys::Event| {  
        leptos::log!("Our custom event happened");  
    };  
    view! {  
        cx,  
        <BltSandwich on:orderSandwich=log_order />  
    }  
}  
  
#[component]  
fn BltSandwich(cx: Scope) -> Element {  
    let dom_node_ref = NodeRef::new(cx);  
  
    let trigger_order_sandwich_event = move |event| {  
        component_custom_event::dispatch("orderSandwich", dom_node_ref);  
    };  
    view! {  
        cx,  
        <div _ref=dom_node_ref>  
           <h3>"BLT Sandwich"</h3>  
            <button on:click=trigger_order_sandwich_event>  
                "Order Sandwich"  
            </button>  
        </div>  
    }  
}
```
