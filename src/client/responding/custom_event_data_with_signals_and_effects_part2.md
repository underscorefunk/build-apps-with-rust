# Custom Event Data with Signals and Effects - Part 2

## What we know
- We can use signals as message busses and effects as message bus watchers to react to changes in our application

## What we'll learn
- How to create a state struct to hold application state and accept events to update the data in an application.

## The lesson

In our previous lesson, our code looked like this:

```rust
use leptos::*;  
  
#[derive(Clone, Debug)]  
enum Sandwich{  
    BLT,  
    Rubin,  
    PBandJ  
}  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <SandwichShop />  
        }    })}  
  
#[component]  
fn SandwichShop(cx: Scope) -> Element {  
  
    let (  
        last_order,  
        new_order  
    ) = create_signal(cx, None::<Sandwich>);  
  
    create_effect(cx, move |_| {  
        match last_order.get() {  
            Some(sandwich) => {  
                leptos::log!("A sandwich was ordered: {:?}", sandwich);  
            },  
            None => {}  
        }    });  
  
    view! {  
        cx,  
        <div>  
            <Sandwich new_order sandwich=Sandwich::BLT label="Bacon, Lettuce. Tomato"/>  
            <Sandwich new_order sandwich=Sandwich::Rubin label="Rubin"/>  
            <Sandwich new_order sandwich=Sandwich::PBandJ label="Peanutbutter and Jelly"/>  
        </div>  
    }}  
  
#[component]  
fn Sandwich(
	cx: Scope, 
	new_order: WriteSignal<Option<Sandwich>>, 
	sandwich: Sandwich, label: &'static str 
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

There are some problems  with this approach. The application component has a lot of functionality rolled into it. It would be preferrable if we could split this stuff out so that it's a bit easier to see the application logic from it's user iterface. 

First things first, let's split our application state out from the application Leptos component. The state of our application is a snapshot of the application's data.

We'll also make a piece of data in the state to hold the last_event. We'll also go so far as to make an enum that stores the possible events as well.

```rust

// Things that can happen in a sandwich shop
#[derive(Debug, Clone, Copy)]
enum Event {
	OrderSandwich(Sandwich),
	None
}

#[derive(Debug, Clone, Copy)] 
struct State { 
	last_event: Event
}
```

We'll need an easy way to setup a default state. We can implement the default trait for the `State` struct. Recall that traits are a specification of behaviours/capabilities of a type. Traits are also types. We can use traits as bounds for argument types by writing their names as the required types for parameters.

```rust
fn example_trait_requirement( my_parameter: SomeTrait) { 
	//...
}
```

Implementing the default trait on a type requires the following:

```rust
impl Default for State {  
    fn default() -> Self {  
        Self {  
            last_event: Event::None  
        }  
    }
}
```

We now have the ability to call `State::default()` and we'll receive a `State` struct value with `last_event` set to the `Event` enum variant of `None`. This is a bit more streamlined than dealing with the option type in our previous example.

We also want to be able to update our state. We are going to force the updating of state through a `State` `update` method, requiring an event. This pattern of adding constraint like, "You MUST have THIS to do THAT" is how we make stable applications. You'll see this enforced all over the place in Rust.

To add functions associated with the `State` struct, we can write `impl` (for implement) `State` (the name of the struct), and define a scope with curley braces to contain the implementations of our methods. We are not writing the implementation of a Trait, so we don't need to write `impl TraitName for StructName`, like we did with the `Default` trait. It's the same idea though.

Our update method will take a mutable reference to itself so that it can update its own data, and it takes an event which dictates how it's own data will be updated. We then match on the events and handle the updates accordingly. At the end, we'll update the last_event with the event used for the update so that we know what happened.

```rust
impl State {  
    fn update(&mut self, event: Event) {  
        match event {  
            Event::OrderSandwich(sandwich) => {  
                leptos::log!("A sandwich was ordered: {:?}", sandwich);  
            },  
            Event::None => {}  
        }        
        self.last_event = event;  
    }  
}
```

As cool as all of this is, we're no further ahead. As developers we have to be careful of things that look like cool patterns but don't add any extra functionality. It's easy to get caught up in what feels satisfying to write because it's clever. Often things that are mentally taxing to write or figure out are the most stimulating. Try to avoid this siren song. Err on the side of simplicity.

We're now going to hook this into our reactive system so that it makes a meaningful change and we'll review the complexity to see if we've simplified our system or made it more complex.

Let's dig in...

We know when we start our app up, we're going to need to initialize a state. We want the state to handle its own updates though and we do not want the state to be rewritten. For this reason, we'll crate a signal to store a reactive value of type `State`, but we're only going to grab the read signal. 

We need a context/scope to create the signal, and our default values.

```rust
#[component]  
fn SandwichShop(cx: Scope) -> Element {  
    let (state, _) = create_signal(cx, State::default() );
```

This feels overly complicated. There's a lot happening here when I really want to just write, "Give me a `State` struct." 

Let's change this to something like...

```rust
let state = State::new(cx);
```

Notice how we're distilling down a previously complicated statement into one that expresses exactly what we want. 

Now we need to refactor our state struct to represent this. The first step is, let's just delete the whole `impl Default for State` block. We're not allowing people to create a default state anymore.

We do need to update our `State` implementations to include the addition of a `new` method:

```rust
impl State {  
  
    pub fn new(cx: Scope) -> ReadSignal<State> {  
        let init_state = Self {  
            cx,  
            last_event: Event::Init  
        };  
        let (state, _) = create_signal( cx, init_state );  
        state  
    }
	// ...
}
```

You can see where we took some of the complexity of things that happened in our application and pushed it into this method. It makes our SandwichShop Leptos component 
much more simple and clear.

We also need to update our struct's properties so that we can store a `Scope` within the state as well.

```rust
#[derive(Debug, Clone, Copy)]  
struct State {  
    cx: Scope,  
    last_event: Event  
}
```

Let's turn our eyes to this `last_event` property. It also needs to be made into a signal so that we can update it and respond reactively. We'll update the struct literal syntax with a create signal call for `last_event`'s value.

```rust
pub fn new(cx: Scope) -> ReadSignal<State> {  
    let init_state = Self {  
        cx,  
        last_event: create_signal( cx, Event::Init)  
    };  
    let (state, _) = create_signal( cx, init_state );  
    state  
}
```

We also need to update our struct to match this new value type.

```rust
#[derive(Debug, Clone, Copy)]  
struct State {  
    cx: Scope,  
    last_event: last_event: (ReadSignal<Event>, WriteSignal<Event>)
}
```

The last piece of this refactor is in the State `update` method. We were storing the event we were responding as the value of `last_event`. The type of this property on the `State` struct has changed. It's not an `Event` anymore. 

```rust
pub fn update(&mut self, event: Event) {  
        match event {  
            Event::OrderSandwich(sandwich) => leptos::log!("A sandwich was ordered: {:?}", sandwich),  
            _ => {}  
        }        
        self.last_event = event;  
    }  
```

We need to change:

```rust
self.last_event = event;  
```

To the following:

```rust
self.last_event.1.set( event );
```

`last_event` is a tuple with two values as index 0 and 1. Index 1 contains the write signal, which has a set method. We're using that value's set method to update the reactive value of the signal.

This feels unclear to me so i'll rewrite it as:

```rust
	self.update_last_event( event );
```

And create a private method on self that hides the read/write implementation feature.

```rust
fn update_last_event( &mut self, event: Event ) {  
    self.last_event.1.set(event );  
}
```

You may notice that the previous methods had the `pub` keyword before the `fn` keyword. Excusion of the `pub` keyword for `update_last_event` prevents the method from being called by external callers. Only methods on the `State` struct can call `update_last_event`.

### Updating the effect

Our previous example had an effect that would respond to changes to our application's last order.

```rust
create_effect(cx, move |_| {  
    match last_order.get() {  
        Some(sandwich) => leptos::log!(
	        "A sandwich was ordered: {:?}", 
	        sandwich
			),  
        None => {}  
    }
});
```

We actually don't need to use this anymore because we've got a state value that we can directly update and react to, all in one contained struct.

### Updating the sandwich components

We no longer need to pass more complicated handlers on down. We can just pass `state`.

```rust
#[component]  
fn SandwichShop(cx: Scope) -> Element {  
    let state = State::new(cx);  
    view! {  
        cx,  
        <div>  
            <Sandwich state sandwich=Sandwich::BLT label="Bacon, Lettuce. Tomato"/>  
            <Sandwich state sandwich=Sandwich::Rubin label="Rubin"/>  
            <Sandwich state sandwich=Sandwich::PBandJ label="Peanutbutter and Jelly"/>  
        </div>  
    }}
```

Recall that our `State::new()` gives us a read signal so that we can easily pass it around our system. We need to update our Sandwich components to match with a new property type for `state`:

```rust
	state: ReadSignal<State>
```

And we'll update the place_order closure so that it calles an update method on the actual state object.
```rust
#[component]  
fn Sandwich(
	cx: Scope, 
	state: ReadSignal<State>, // <- here
	sandwich: Sandwich, 
	label: &'static str 
) -> Element{

	let place_order = move |_|{  
	    state.get().update(Event::OrderSandwich(sandwich))  
	};
	//...
}
```

### What remains

When I had set out to do this refactor I was thinking that we'd need the `last_event` as a signal to respond to, so that we could build reactvity off of it with `create_effect()`. The reality is that in this example, we don't even actually need that. :)

My hope is that this lesson gives you some insight into the thought process of refactoring and adding constraint to changes.

Here's what the finished code looks like:

```rust
use leptos::*;  
  
#[derive(Debug, Clone, Copy)]  
enum Sandwich{  
    BLT,  
    Rubin,  
    PBandJ  
}  
  
#[derive(Debug, Clone, Copy)]  
enum Event {  
    OrderSandwich(Sandwich),  
    Init  
}  
  
#[derive(Debug, Clone, Copy)]  
struct State {  
    cx: Scope,  
    last_event: (ReadSignal<Event>, WriteSignal<Event>)  
}  
  
impl State {  
  
    pub fn new(cx: Scope) -> ReadSignal<State> {  
        let init_state = Self {  
            cx,  
            last_event: create_signal( cx, Event::Init)  
        };  
        let (state, _) = create_signal( cx, init_state );  
        state  
    }  
  
    pub fn update(&mut self, event: Event) {  
        match event {  
            Event::OrderSandwich(sandwich) => leptos::log!("Yay! A sandwich was ordered: {:?}", sandwich),  
            _ => {}  
        }        
        self.update_last_event(event );  
    }  
  
    fn update_last_event( &mut self, event: Event ) {  
        self.last_event.1.set(event );  
    }  
  
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
    let state = State::new(cx);  
    view! {  
        cx,  
        <div>  
            <Sandwich state sandwich=Sandwich::BLT label="Bacon, Lettuce. Tomato"/>  
            <Sandwich state sandwich=Sandwich::Rubin label="Rubin"/>  
            <Sandwich state sandwich=Sandwich::PBandJ label="Peanutbutter and Jelly"/>  
        </div>  
    }}  
  
#[component]  
fn Sandwich(
	cx: Scope, 
	state: ReadSignal<State>, 
	sandwich: Sandwich, 
	label: &'static str 
) -> Element{  

	let place_order = move |_|{  
        state.get().update(Event::OrderSandwich(sandwich))  
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