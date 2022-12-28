# Custom event module with data

## What we know

- Data can be assocaited with custom events
- We can hide the complexity of creating custom events behind an easy to use function inside a module.

## What we'll learn

- How to add data to our module.

## The lesson

In a previous lesson we created a module that made dispatching a custom event super easy. We discussed adding data to events before and how useful it is, but we can't add data to be disptched with the events in our custom event module. We should fix that!

Our simple module looks like this:

```rust
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
  
    pub fn dispatch( name: &str, target_ref : NodeRef) 
    -> Result<bool, JsValue>{  
  
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
```

### Adding JsValue data to the event

I have an example from a previous lesson where I added a payload to an event through the details method on the event configuration. Let's add that back in. 

The first step is adding a new parameter to the `new` function which allows us to get a JsValue in there.

We'll add an Option, because sometimes we might not have a payload. Alternately we could create a function called 'new_with_payload' but this is fine.

```rust
fn new(name: &str, payload: Option<JsValue>) -> Result<web_sys::CustomEvent, JsValue> 
```

Then we'll conditionally add it to the event_config if we're given Some(data) for the payload.

```rust
if let Some(data) = payload {  
	event_config.detail(&data);  
}  
```

All finished, we have this:

```rust
fn new(name: &str, payload: Option<JsValue>) -> Result<web_sys::CustomEvent, JsValue> {  
    let mut event_config = web_sys::CustomEventInit::new();  
    event_config.bubbles(true);  
    if let Some(data) = payload {  
        event_config.detail(&data);  
    }  
    web_sys::CustomEvent::new_with_event_init_dict(name, &event_config)  
}
```

### Simplified sending methods (API)

There is no way to specify a default parameter value in Rust. We want our api to be simple and declarative. We don't want to force people to always provide "None" if they're not including a payload. To fix this we can make a private method called real_dispatch and then a public method to dispatch and event without and with a payload respectively, called `dispatch` and `dispatch_with_data`.

```rust
fn real_dispatch( 
	name: &str, 
	target_ref : NodeRef, 
	payload: Option<JsValue>
	) -> Result<bool, JsValue>{  
  
    let event = new(name, payload);  
    let target = target_ref.get();  
  
    match (event, target) {  
        (Ok(event), Some(target)) => {  
            target.dispatch_event(&event)  
        },        (_,_) => Err(JsValue::null())  
    }
}

pub fn dispatch( 
	name: &str, 
	target_ref : NodeRef
	) -> Result<bool, JsValue>{  
    real_dispatch( name, target_ref, None)  
}

pub fn dispatch_with_data( 
	name: &str, 
	target_ref : NodeRef, 
	data: JsValue
	) -> Result<bool, JsValue>{  
    real_dispatch( name, target_ref, Some(data))  
}
    
```

While we're at it, let's add a function to grab the value a bit more easily too:

```rust
pub fn extract_data( event: web_sys::Event) -> JsValue {  
    event
	    .unchecked_into::<web_sys::CustomEvent>()
	    .detail()  
}
```
> We've seen this in a prior lesson. Were just packaging it as part of the module here.

### Structured Data

Here's where things get interestng. We probably don't to just send a single value. We might want to send a few values. If we went back to our BLT example, maybe we want to send a struct of the whole BLT Sandwich config.

JavaScript requires everything to be sent as text. We need a way to convert structured data into a sequence of characters that can faithfully represent it. The process of producing this is called serialization. Converting data from a serialized representation to it's typed and structured form is called deserialization. 

Currently what we have will allow us to send stuctured data but it's on the application developer to serialize data and convert it into a JsValue for use with `dispatch_with_data()`. I think it would be convenient to do this for them so that they don't have to think about serialization. 

Let's start by adding a new function. I don't know what the type of data will be so I'm writing UNKNOWN for the sake of this example. This is not a Rust thing. It's just for you, the reader, to help you follow my thought process.

```rust
pub fn dispatch_with_data_serialized(  
    name: &str,  
    target_ref : NodeRef,  
    data: UNKNOWN
    ) -> Result<bool, JsValue>{  
	// ..
}
```

I did some searching and found a great crate called "serde" which received it's name from ser-ialize de-serialize. It turns out that there is a version of serde specifically designed to work with wasm, which is supposidly more efficient than converting structured data into JSON (JavaScript Object Notation) and it gives us a JsValue! How great is that!

I've added the dependency to cargo.toml as such:
```toml
serde-wasm-bindgen = "0.4"
```

And now I can author the body of the function which is actually realtively simple:

```rust
match serde_wasm_bindgen::to_value(data) {  
	Ok(data) => dispatch_with_data( name, target_ref, data),
	Err(_) => Err( JsValue::null() )  
}
```

We're matching on the result of converting a reference of our data to the JsValue, if it's ok, we destructure it and return the result of dispatch_with_data, otherwise we'll return a null JsValue as an error. 

Again, recall that we're keeping the return types the same as `EventTarget.dispatch()`.

We're not quite done though. We don't know what type to put for the data. Rust requires that we specific the type so that it can verify that we're calling the appropriate methods on it, and correctly managing memory for it.

To do this we'll revisit type generics, which are those type arguments that I talked about before. They're like parameters/variables but for types. People often use 'T' as a character for a generic 'Type' but you can actually use anything you want that isn't a reserved word. I'm going to use `Data`. Note that `data` is the parameter name and `Data` is the generic type name. 

```rust
pub fn dispatch_with_data_serialized<Data>(  
    name: &str,  
    target_ref : NodeRef,  
    data: &Data
    ) -> Result<bool, JsValue>{  
    //....
}
```

Here we're saying, a generic will be used called `Data` and the property `data` will be whatever type `Data` is, as a referenced value. We're telling Rust, this type can change.

As is, Rust will complain because we're using the value of `data` as an argument for  `serde_wasm_bindgen::to_value()`. Rust wants to confirm that whatever is being stored in `data`, accepted through the function call, can be safely passed to that `serde_wasm_bindgen::to_value()` function, meeting its type requirements.

Let's look at the definition of `serde_wasm_bindgen::to_value()` for a clue. It reads as:

```rust
pub fn to_value<T: serde::ser::Serialize + ?Sized>(value: &T) -> Result<JsValue> {
```

>Translation: "Whatever you pass as the value of `value`  must be a reference to T. T is any value whoes type implements the the serde::ser::Serialize trait and is`Sized`.

That's it. Our `Data` needs to fulfill the same type requirements as  `serde::ser::Serialize + ?Sized` The colon after 'T' indicates qualifiers for 'T'. These qualifiers are Traits. A Trait is a name that refers to a specification of behaviour/capabilities. If you've ever written object oriented code, these would be similar to interfaces. 

### How do we serialize data?

```rust
pub fn dispatch_with_data_serialized<Data: Serialize + ?Sized>(  
    name: &str,  
    target_ref : NodeRef,  
    data: &Data
    ) -> Result<bool, JsValue>{  

	match serde_wasm_bindgen::to_value(data) {  
        Ok(data) => dispatch_with_data( name, target_ref, data),
        Err(_) => Err( JsValue::null() )  
	}
}
```


Serde is included in Leptos and adds support for a bunch of types out of the box. We can also add serialization support for our own types with a macro.  Writing `#[derive(Serialize, Deserialize)]`
 above a struct will tell Rust to write out the functionality to enable these fetures for you. You do, howver, need to import the traits `Serialize` and `Deserialize` with the following use statement:

```rust
use serde::{Serialize, Deserialize};
```
> We are destructuring here in the use statement so that Serialize and Deserialize are being brought into scope from the serde crate (external module).

```rust
#[derive(Serialize, Deserialize, Debug)]  
struct BLTSandwich {  
    bread: String,  
    lettuce: String,  
    tomato: String,  
    bacon: String,  
}
```
> We're also adding Debug so that we can print this struct later in the lesson with the log macro.

Now let's take a look at our BLT Sandwich component:

```rust
#[component]  
fn BltSandwich(cx: Scope) -> Element {  
    let dom_node_ref = NodeRef::new(cx);  
  
    let trigger_order_sandwich_event = move |event| {  
        component_custom_event::dispatch_with_data_serialized(  
            "orderSandwich",  
            dom_node_ref,  
            &BLTSandwich {  
                bread: "canadian_rye".to_string(),  
                lettuce: "romaine".to_string(),  
                tomato: "black_krim".to_string(),  
                bacon: "farm_smoked_apple_bacon".to_string(),  
            }        
        );  
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

It looks like there's one last missing piece of the pizzle. We need a function that will extract our serialized data back into our struct. We'll use as similar tactic to find the type requirements. The generic `Data` will end up in the return type as a type argument for Option. This means that the function may return Some of `Data`, which must implement the traits `serde::de::DeserializeOwned + ?Sized` or None.

Then we call `from_value()`, match it to handle a potential error and return our option types as the result of the match arm expressions.

```rust
pub fn extract_serialized_data<Data: serde::de::DeserializeOwned + ?Sized>(event: web_sys::Event) -> Option<Data> {  
    match serde_wasm_bindgen::from_value(extract_data(event)) {  
        Ok(data) => Some(data),  
        Err(_) => None  
    }  
}
```

When using this function we need to provide it the type for `Data` which we can do with our type argument syntax, for example:

```rust
component_custom_event::extract_serialized_data::<BLTSandwich>(event)
```
> The ::<> is a turbofish and used to inject a concrete type as an argument for a generic.

### Wrapping it up

Here we have a working example of the whole thing!

```rust
use leptos::*;  
use serde::{Serialize, Deserialize};  
  
mod component_custom_event {  
    use leptos::*;  
    use crate::wasm_bindgen::JsValue;  
  
    fn new(name: &str, payload: Option<JsValue>) -> Result<web_sys::CustomEvent, JsValue> {  
        let mut event_config = web_sys::CustomEventInit::new();  
        event_config.bubbles(true);  
        if let Some(data) = payload {  
            event_config.detail(&data);  
        }  
        web_sys::CustomEvent::new_with_event_init_dict(name, &event_config)  
    }  
    fn real_dispatch(name: &str, target_ref: NodeRef, payload: Option<JsValue>) -> Result<bool, JsValue> {  
        let event = new(name, payload);  
        let target = target_ref.get();  
  
        match (event, target) {  
            (Ok(event), Some(target)) => target.dispatch_event(&event),  
            (_, _) => Err(JsValue::null())  
        }    }  
    pub fn dispatch(name: &str, target_ref: NodeRef) -> Result<bool, JsValue> {  
        real_dispatch(name, target_ref, None)  
    }  
    pub fn dispatch_with_data(name: &str, target_ref: NodeRef, data: JsValue) -> Result<bool, JsValue> {  
        real_dispatch(name, target_ref, Some(data))  
    }  
    pub fn dispatch_with_data_serialized<T: serde::ser::Serialize + ?Sized>(  
        name: &str,  
        target_ref: NodeRef,  
        data: &T) -> Result<bool, JsValue> {  
        match serde_wasm_bindgen::to_value(data) {  
            Ok(data) => dispatch_with_data(name, target_ref, data),  
            Err(_) => Err(JsValue::null())  
        }    }  
    pub fn extract_data(event: web_sys::Event) -> JsValue {  
        let custom_event = event.unchecked_into::<web_sys::CustomEvent>();  
        custom_event.detail()  
    }  
    pub fn extract_serialized_data<Data: serde::de::DeserializeOwned + ?Sized>(event: web_sys::Event) -> Option<Data> {  
        match serde_wasm_bindgen::from_value(extract_data(event)) {  
            Ok(data) => Some(data),  
            Err(_) => None  
        }  
    }}  
  
#[derive(Serialize, Deserialize, Debug)]  
struct BLTSandwich {  
    bread: String,  
    lettuce: String,  
    tomato: String,  
    bacon: String,  
}  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <SandwichShopApp />  
        }    })}  
  
#[component]  
fn SandwichShopApp(cx: Scope) -> Element {  
    let log_order = |event: web_sys::Event| {  
        leptos::log!("Our custom event happened");  
        leptos::log!( "{:?}", component_custom_event::extract_serialized_data::<BLTSandwich>(event));  
    };  
    view! {  
        cx,  
        <BltSandwich on:orderSandwich=log_order />  
    }}  
  
#[component]  
fn BltSandwich(cx: Scope) -> Element {  
    let dom_node_ref = NodeRef::new(cx);  
  
    let trigger_order_sandwich_event = move |event| {  
        component_custom_event::dispatch_with_data_serialized(  
            "orderSandwich",  
            dom_node_ref,  
            &BLTSandwich {  
                bread: "canadian_rye".to_string(),  
                lettuce: "romaine".to_string(),  
                tomato: "black_krim".to_string(),  
                bacon: "farm_smoked_apple_bacon".to_string(),  
            },        );  
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