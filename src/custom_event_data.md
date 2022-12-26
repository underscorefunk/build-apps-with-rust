# Custom Event Data

## What we know

- Custom events can be dispatched and bubbled up to handle events at different levels of your applications.
- Custom events allow us to convert imperative events based on DOM interaction into domain specific events that are more declarative.

## What we'll learn

- A deeper look in to declarative or domain specific code
- Why and how to add data with custom events

## The Lesson

We introduced custom events in a previous lesson, with the following code:

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
    let log_response = |_| {  
        leptos::log!("Our custom event happened")  
    };  
    view! {  
        cx,  
        <MyComponent on:myCustomEvent=log_response />  
    }  
}  
  
#[component]  
fn MyComponent(cx: Scope) -> Element {  
    let dom_node_ref = NodeRef::new(cx);  
  
    let trigger_sending_of_custom_event = move |_| {  
  
        let mut event_config = web_sys::CustomEventInit::new();  
        event_config.bubbles(true);  
        let event = web_sys::CustomEvent::new_with_event_init_dict(
	        "myCustomEvent", 
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
            <button on:click=trigger_sending_of_custom_event>  
                "Trigger custom event"  
            </button>  
        </div>  
    }  
}
```

In this code we have a Leptos component which contains a `view!` tempate with a div and a button. This button has a handler which is a closure passed to a `click` event via the `on:click` property (prop). 

Leptos has a special private property for elements in `view!` templates. It's called `_ref` and it's used to allow Leptos to refer to HTML elements (by refering to them) in the client side runtime (in the browser). We create a reference in a given scope/context, and apply it as the value of the property `_ref`. 

Recall that the root element of a `view!` template is interchangable with it's component tag. By placing a reference on the root `<div>` we're actually creating a reference to `<MyComponent>`.

The reference is used in the event handler so that our custom event appears to be dispatched from our Leptos component, allowing us to add a handler with `<MyComponent on:myCustomEvent=... />`.


### Platform specific to domain specific

Systems and applications are full of complex mechanism. They contain behaviours that are described in code that reveal how the platform was designed and implemented. 

Systems and applications are also full of domain specific complexity. They contain behaviours relating to the "business logic" or description of how the application solves a problem. How these problems are solved often describe activities relating to the problem, and not specifically relating to the technology that it runs on.

Or example, let's think about building a sandwich shop ecommerce application. And let's say we're clicking on a "buy sandwich button". That button would have an on click event to add a sandwich to your cart. The idea of clicking and dispatching an event when something is clicked doesn't actually have anything to do with buying sandwiches. It actually has to do with the platform. 

In our minds we may look at the button, the intention behind it, the text node as its label, and infering that clicking on the button should order a sandwich. This is implied and requires us to think about the intention of the application through its interaction with the platform.

If that event became an "order sandwich" event then we'd be in domain territory. It is specific to the language that describes activites and actions in the head space or domain of our problem—our sandwich shop.

Separating required knowledge of the platform from knowledge and interactions between business processes will allow you to focus on each area separately. This will allow you to change which things could trigger a sandwich order instead of introspecting and evaluating generic events.

This can make applications more flexible, robust, simple, and easier to understand. 


### The case for associated event data

We've outlined that it's useful to separate platform events from application events, but we have't discussed the importance of associated data with those events yet. 

Let's go back to our online sandwich shop as an example case. If have one sandwich, we're good. We can dispatch a custom event called "orderSandwich" and let that be that. But what do we do if we have more than one sandwich type? We'll need some way to know which sandwich we're ordering.

One solution could be to create one event per sandwich type. Perhaps we have orderRubinSandwich or orderBLTSandwich. This could be a completely valid solution if we only had a few sandwiches. Where things get tricky is when we start to think about configurations of sandwiches. We'll end with a combinatorial explosion of event types to match each sandwich configuration.

Our  Bacon Lettuce and Tomato sandwich, with the ability to select different breads, leafy greens, or tomato types, then we'd quickly end up with too many variants of event to manage.

This is a situation where we'd like our system to dispatch an event called orderSandwich, with data associated to configures what the sandwich is. It would be idea if we could send an orderBLTSandwich event where we specify which bread, leafy greens, or type of tomato are requeted by the customer.

> Beware, you may feel the urge to continue abstracting. It's not uncommon to think, "Well, what if we want to sell different things at our sandwich shop? Why don't we just have orderItem as an event type and the configure of that item can include the item type, being a sandwich. If we ordered a drink, then drink would be the item type, and so forth." One could say that this is a premature generalization. The more general a system becomes the less its components express the function of the application. Moving from specific to generic actually adds some complexity in that you need to apply a case to think about the generalization. Try to start with the concrete, known, specific, and within the domain. Then refactor and generalize as needed as the application grows. There are no hard an fast rules for when to do this, just be aware that you do not need to hyper generalize your solutions at the start. Write what you mean, be clear, and you'll thank yourself later when you have to go in and edit things a month or year from now. :)



### Getting the configuration

For the most simple example we can actually hard code the configuration into the event sender. We don't need to pull it out any HTML data attributes or input fields.

Our template could look like this:

```rust
 view! {  
    cx,  
    <div _ref=dom_node_ref>  
     <h3>"BLT Sandwich"</h3>  
        <button  
         on:click=trigger_order_sandwich_event  
         >  
            "Order Sandwich"  
        </button>  
    </div>  
}
```

We need to update the event handler so that our new custom event is sent with this extra data. Web events have a property on their JavaScript object called `detail`, which we can use to story and carry arbitrary data. 

After we initialize the `event_config`, we modify it so that it bubbles up (so that ancestors can respond to the event), and then we'll do another modification to add the detail data. Recall that if we're changing a piece of data we need to write `mut` before the name of it to specify that it can be changed, that it can be MUTated.

```rust
	let mut event_config = web_sys::CustomEventInit::new();  
    event_config.bubbles(true);  
    event_config.detail(&data);
```

But now you're probably wondering, what is `&data`. We're providing the detail method on `event_config` with a reference (denoted with the `&` ) to `data`. The detail method accepts any JsValue. In our simple example, we're only going to specify bread type. 

```rust
let bread_type = JsValue::from("Canadian Rye");  
event_config.detail( &bread_type );
```
> We are calling the from static method on the JsValue struct to create a new JsValue from our string slice "Canadian Rye". We're then using the bread_type as an argument for the detail method, but we're passing the data as a reference, denoted with the `&`; If you forget the ampersand the Rust compiler will actually make the recommendation for you to include it so that your usage matches the `detail()`  method's definition.

To use JsValue we need to bring it into scope. At the top of your main.rs file, add 

```rust
use crate::wasm_bindgen::JsValue;
```


Our whole BLT component looks like this:

```rust
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

Now let's look at the top part of our app with our mount_to_body and top level app component:

```rust
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
    let log_order = |_| {  
        leptos::log!("Our custom event happened");  
    };  
    view! {  
        cx,  
        <BltSandwich on:orderSandwich=log_order />  
    }  
}
```

Let's focus in on the orderSandwich event handler:

```rust
let log_order = |_| {  
	leptos::log!("Our custom event happened");  
};  
```

Note that before we had a underscore for the event parameter of the handler. We had no need of the event in the context of our closure's body (in-between the curley braces) so we wrote an underscore to tell Rust that we're not using. This is a Rust convention.

Now we need the event but we don't know what type it is. We can let Rust do the work for us. Put any type in there and run `trunk serve` if you're not already.

```rust
let log_order = |event: i32| {  
	leptos::log!("Our custom event happened");  
};  
```

The compiler will check for you and tell you about the mismatch.

```
expected closure signature `fn(Event) -> _`
   found closure signature `fn(i32) -> _`
```

This tells us that it should be an Event type, not i32. :D The compiler is so helpful.

```rust
let log_order = |event: Event| {  
	leptos::log!("Our custom event happened");  
};  
```

If we just write the above the compiler will also tell us that "Event" doesn't exist in our scope. It's telling us we need to be more specific about what we mean. Then it outlines ways that we can bring the definition of events into our scope.

```
help: consider importing one of these items
   |
1  | use crate::web_sys::Event;
   |
1  | use web_sys::Event;
   |

```

Use `web_sys::Event` would force all `Event` types to be `web_sys::Event` types. Think of it like we're importing the type. We can also just manually write the type with the namespace in our closure. I prefer to include the crate or module as context for clarity. 

```rust
let log_order = |event : web_sys::Event| {
```

...feels more clear than...

```rust
let log_order = |event : Event| {
```

Shorter code isn't always better code. Aim to be clear and to avoid ambiguity.

Now, unfortunately there is no `detail()` method on a `web_sys::Event`. But a `web_sys::Event` is a JsValue and we can turn it into a custom event:

```rust
let custom_event = event.unchecked_into::<web_sys::CustomEvent>();
```

Here we're calling the `unchecked_into` method on the event and using the turbo fish `::<>` syntax to provide the destination type argument, which is a `web_sys::CustomEvent`.

It should be noted that this is a unique behaviour to working with things that are `JsValue` types at their core. Rust doesn't normally work this way and you can not just smash one type into another type with this ease. JavaScript is not a typed language. When we work with JsValues we're often taking the raw data from JavaScript and pushing it into a Rust context where we enforce type safety from there forward. This is how we can call `unchecked_into` to convert the regular event to the custom event, granting us access to the `.detail()` method.

Our app event handler now looks like this:

```rust
let log_order = |event : web_sys::Event| {  
    let custom_event = event.unchecked_into::<web_sys::CustomEvent>();  
    let sandwich_type = custom_event.detail();  
    leptos::log!("Our custom event happened");  
    leptos::log!("{:?}", sandwich_type );  
};
```

You'll note that when we log the value of `sandwich_type`, the console in your browser will say `JsValue("Canadian Rye")`.  The value we pulled out of detail() is a JsValue and needs to be converted into a rust type to be used elsewhere in your system.

We can use a special method on `JsValue` values called `as_string()`, but it returns an Option type which we can handle with our match statements.

```rust
let log_order = |event : web_sys::Event| {  
  
	leptos::log!("Our custom event happened");  
    
    let custom_event = event.unchecked_into::<web_sys::CustomEvent>();  
  
    let bread_type_js = custom_event.detail();  
    let opt_bread_type_rs = bread_type_js.as_string();  
    
    match opt_bread_type_rs {  
        Some(bread_type) => { leptos::log!("{:?}", bread_type ) },  
        None => {}  
    }  
};
```

We can reduce assignments here by chaining all of these together.

```rust
let log_order = |event : web_sys::Event| {  
    leptos::log!("Our custom event happened");  
    let bread_type = event  
        .unchecked_into::<web_sys::CustomEvent>()  
        .detail()  
        .as_string()  
        .unwrap_or(String::new());  
  
    leptos::log!("{:?}", bread_type );  
};
```

The new method here is `unwrap_or`, which takes the Some value or uses a default value (provided as an argument) if none.

The whole thing together looks like this:

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