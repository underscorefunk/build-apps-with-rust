# Custom Event Data with Signals

## What we know
- Events allow us to signal changes on the client side (in browser)
- Attaching handlers (listeners) to events in the DOM has non trivial performance cost
- Custom events have the ability to add data to them in the form of _details_.
- Serializing and deserializing data to transport it between JavaScript and WASM has a non trivial performance cost 
- Events can bubble up from the target (the dispatcher) node to be handled by parent/ancestral DOM node's handlers.
- Event handlers can be passed down as properties to Leptos components

## What we'll learn
- How we can listen to signal value changes and respond with actions using _effects_

## The lesson

### Why bother?

It's possible to use events that exist inside your Rust application instead of relying heavily on the browser's event system. There are some major benefits that you receive by doing this. 

First, you can use data in your events that isn't serializable. Recalll that JavaScript events have details, but the data assigned to it has to be able to be turned into a string. It has to be serializable. Functions and few other data types can not be serialized. Handling events in rust solves this problem.

Secondly, serializing and deserializing data is costly. If we're handling an event in rust, serializing the details (data/payload) and then dispatching it to handle it in rust a few ms later, we're better off to just keep it all in Rust.

Thirdly, passing data across the WASM boundary isn't very efficient in browsers yet.

Fourthly, I'm sure there are other reasons.

### Our objectives

We'll use a sandwich shop as our example for this lesson. We'll aim to create a simple application that accepts an order which the system/application can choose to react to. Think of it like a server taking the order and relaying it to the kitchen.

### Boilerplate

Let's start off with some basic components for our sandwich shop.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <SandwichShop />  
        }    
	})
}  
  
#[component]  
fn SandwichShop(cx: Scope) -> Element {  
  
    view! {  
        cx,  
        <div>  
            <Sandwich/>  
        </div>  
    }
}  
  
#[component]  
fn Sandwich(cx: Scope) -> Element{  
    view! {  
        cx,  
        <div>  
           <button>  
                "Order Sandwich"  
            </button>  
        </div>  
    }
}
```

We'll need something to capture the initial client side event. To do this we can add an event handler for the `click` event.

```rust
#[component]  
fn Sandwich(cx: Scope) -> Element{  
    let place_order = |_|{  
        leptos::log!("Place order");  
    };  
    view! {  
        cx,  
        <div>  
           <button on:click=place_order>  
                "Order Sandwich"  
            </button>  
        </div>  
    }
}
```

We're just echoing out "Place order" to the browser console on click to make sure the event is dispatching correctly

### Shared data

Here's where things get interesting. We need some sort of shared space where we can write down that an order came in. We'll then make sure that a bell gets run to say, "order up" for the kitchen staff to check the order. 

We'll use Leptos' reactive system to create that shared bit of data. Using signals we can read and write to the space with orders as the buttons are clicked. 

We're going to simplify and ignore some edge cases and assume that an order can be fulfilled the second it comes in. This lesson is more about message orchestration than anything else. 

With that in mind, if we create a signal, the setter could be called "new order" because it's adding the order to the shared/observed space. We can call the getter "last order" because the value of the shared space will always be the most recent just fulfilled order.

```rust
let (  
    last_order,  
    new_order  
) = create_signal(cx, None);
```

You'll note that I wrote None here because we're going to use an Option type for what the order is. In fact, it'll be `Option<Sandwiches>`;

```rust
enum Sandwich{  
    BLT
}
```

If we tried to compile our application Rust would complain. Currently rust doesn't know how much memory to allocate for create_signal because it's Some type isn't specified. We can add this to the None with our handy trubofish syntax.

```rust
let (  
    last_order,  
    new_order  
) = create_signal(cx, None::<Sandwich>);
```

Now we want to pass the `new_order` write signal to our Sandwich Leptos component. It's going to use this to place orders when the respective button is clicked.

```rust
#[component]  
fn SandwichShop(cx: Scope) -> Element {  
  
    let (  
        last_order,  
        new_order  
    ) = create_signal(cx, None::<Sandwich>);  
  
    view! {  
        cx,  
        <div>  
            <Sandwich new_order=new_order />  
        </div>  
    }}
```

And we'll add the property to the Sandwich component's function definition so that it can accept the write signal.

```rust
fn Sandwich(
	cx: Scope, 
	new_order: WriteSignal<Option<Sandwich>> 
) -> Element{
	// ... 
}
```
>Note that new_order is of type `WriteSignal` which has a type argument of `Option<Sandwich>`

This looks a bit odd because the value and property are the same name on the Sandwich object. Leptos allows you to just write the property/name once if they're both the same. We can write 

```rust
<Sandwich new_order /> 
```

Now we need to put that WriteSignal to use. 

```rust
#[component]  
fn Sandwich(
	cx: Scope, 
	new_order: WriteSignal<Option<Sandwich>> 
) -> Element{  

	let place_order = move |_| {  
	    leptos::log!("Place order");  
	    new_order.set( Some(Sandwich::BLT) );  
	};
    
    view! {  
        cx,  
        <div>  
           <button on:click=place_order>  
                "Order Sandwich"  
            </button>  
        </div>  
    }
}
```

The new_order write signal enters the Sandwich component function and is moved into the place_order closure. This closure is run every time the `click` event is dispatched on the button. By doing so, it updates that shared order space with a sandwich!

### Effects

Leptos has all sorts of tricks up its sleeve. One of them is `create_effect`. You can think of `create_effect` as an on-change event handler for Leptos's reactive system. It accepts a context and closure (callback) function as its two property arguments. Use of signals within the closure will flag the closre to run if their values change. The closure will observe the signals used in it.

We can create an effect to observe the last and new order signals as follows.

```rust
#[component]  
fn SandwichShop(cx: Scope) -> Element {  
  
    let (  
        last_order,  
        new_order  
    ) = create_signal(cx, None::<Sandwich>);
	
	create_effect(cx, move |_| {  
	    if let Some(sandwich) = last_order.get() {  
	        leptos::log!("A sandwich was ordered");  
	    }  
	});
	
	//...
}
```

The rust compiler will complain here because the enum doesn't support clone. The statement `if let Some(sandwich) = last_order.get()` is using the signal's get() method to return a value of `Option<Sandwich>`. It needs to clone the data to give you a copy of it.

To solve this problem we can allow rust to derive the clone trait for the Sandwich enum with :

```rust
#[derive(Clone)]
enum Sandwich{  
    BLT
}
```

We also want to be able to print this with debug formatting so we'll derive the debug trait too.

```rust
#[derive(Clone, Debug)]
enum Sandwich{  
    BLT
}
```

And just like that, we've got a working system:

```rust
use leptos::*;  
  
#[derive(Clone, Debug)]  
enum Sandwich{  
    BLT  
}  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <SandwichShop />  
        }
	})
}  
  
#[component]  
fn SandwichShop(cx: Scope) -> Element {  
  
    let (  
        last_order,  
        new_order  
    ) = create_signal(cx, None::<Sandwich>);  
  
    create_effect(cx, move |_| {  
        match last_order.get() {  
            Some(sandwich) => leptos::log!(
	            "A sandwich was ordered: {:?}", 
	            sandwich),  
            None => {}  
        }    
	});  
  
    view! {  
        cx,  
        <div>  
            <Sandwich new_order />  
        </div>  
    }}  
  
#[component]  
fn Sandwich(
	cx: Scope, 
	new_order: WriteSignal<Option<Sandwich>> 
) -> Element{  
    let place_order = move |_|{  
        leptos::log!("Place order");  
        new_order.set( Some(Sandwich::BLT) );  
    };  
    view! {  
        cx,  
        <div>  
           <button on:click=place_order>  
                "Order Sandwich"  
            </button>  
        </div>  
    }
}
```

### Adding other sandwiches

If we want to add additional sandwiches, we can create additional enum variants.

```rust
#[derive(Clone, Debug)]  
enum Sandwich{  
    BLT,  
    Rubin,  
    PBandJ  
}
```

We'll add some additional sandwiches to our order menu:

```rust
view! {  
    cx,  
    <div>  
        <Sandwich new_order sandwich=Sandwich::BLT/>  
        <Sandwich new_order sandwich=Sandwich::Rubin/>  
        <Sandwich new_order sandwich=Sandwich::PBandJ/>  
    </div>  
}
```

And we'll add that new property 'sandwich' that we're using to configure the component.

```rust

#[component]  
fn Sandwich(
	cx: Scope, 
	new_order: WriteSignal<Option<Sandwich>>, 
	sandwich: Sandwich 
) -> Element{
	// ...
}
```

And, we'll use the new argument in our on click handler.

```rust
let place_order = move |_|{  
    leptos::log!("Place order");
    new_order.set( Some(sandwich) );  
};
```

The above won't work just yet though. When we move the sandwich argument into this closure, the rust compiler will complain. This closure is actually like a struct behind the scenes, with properties for the values moved into it. We need to clone sandwich into this struct so that the closure can guarantee that it doesn't have any ties to the outside scope. We solve this problem by calling `clone` on the sandwich. This will evaluate the value of Some() to a clone of the sandwich because the statement inside the parenthesis are evaluated first.

```rust
let place_order = move |_|{  
    leptos::log!("Place order");
    new_order.set( Some(sandwich.clone()) );  
};
```

### Adding labels

Let's add some new properties for sandwiches for the labels.

```rust
view! {  
    cx,  
    <div>  
        <Sandwich new_order sandwich=Sandwich::BLT label="Bacon, Lettuce, and Tomato"/>  
        <Sandwich new_order sandwich=Sandwich::Rubin label="Rubin"/>  
        <Sandwich new_order sandwich=Sandwich::PBandJ label="Peanutbutter and Jelly"/>  
    </div>  
}
```

And then we'll add the label too the function properties and in the `view!` template:

```rust
#[component]  
fn Sandwich(
	cx: Scope, 
	new_order: WriteSignal<Option<Sandwich>>, 
	sandwich: Sandwich, 
	label: &'static str 
) -> Element{  

	let place_order = move |_|{  
        leptos::log!("Place order");  
        new_order.set( Some(sandwich.clone()) );  
    };  
    
    view! {  
        cx,  
        <div>  
           <button on:click=place_order>  
                "Order " {label}  
            </button>  
        </div>  
    }
    
}
```

It's worth noting that I added the static lifetime to the label so that rust knows the string won't be changing as the application runs. This is important because these component functions are more like setup functions and template builders. They are not render functions.

And like that, we have a pretty cool system that allows us to transmit a messsage up the chain! Pretty neat!

In the next lesson we'll buid on this with a more robust pattern.