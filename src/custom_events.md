# Custom Events

## What we know

- One of the key ways applications change data over time is in response to stimulus. We can witness these changes through a browser's runtime through the browser's event system.

## What we'll learn

- Which events exist and/or are supported
- How we can create our own events, specific to the domain of our problem
- An introduction to event driven architecture

## The Lesson

There are some common events that you can probably intuitively guess. Going from pure intuition is only going to get you so far. 

We frame the way we solve problems through the lense of the tools we have at hand. For this reason, it's a good idea to familiarize yourself with the HTML tags that exist and the web events that exist. The web platform has a ton of features that a lot of people don't know about because they stopped learning HTML at `<div>` and `<p>` tags.

The mozilla foundation has a wonderful website called MDN which contains invaluable reference to help expand your knowledge.

- [HTML elements and reference](https://developer.mozilla.org/en-US/docs/Web/HTML)
- [List of web events](https://developer.mozilla.org/en-US/docs/Web/Events#event_listing)

The List of web events will provide everything you need to know to respond to actions on elements which you can attach to your Leptos components.

### Custom Events

You may wish to create your own custom events. Custom events can be useful when you want to differentiate a generic behaviour in the web, from a specific behaviour or event in your application. 

For example:

```rust
#[component]  
fn MyLunchbox(cx: Scope) -> Element {  
    let consume_sandwich = |_|{  
      // do something in response to  
      // the sandwich being eaten.
    };  
    view!{  
        cx,  
        <Sandwich on:eat=consume_sandwich/>  
    }  
}

#[component]  
fn Sandwich(cx: Scope) -> Element {  
    let trigger_eating_event = |_|{  
      // Code that triggers the custom event
      // which will bubble up to from the 
      // botton to it's parent
    };  
    view!{  
        cx,  
        <button on:click=trigger_eating_event/>
	        I'm a snack
        </button> 
    }  
}
```

If we were to write this with standard events, we would need some more introspection and features of the platform start to leak up into higher levels of our application. This issue can be simplified as saying, knowledge of the application needs to span across the boundary of multiple components. More things to keep in your head makes programs harder to reason about, more difficult to extend/modify, and less clear to newcomers wishing to contribute to the application. Or, maybe you just came back from a vacation and forgot all about how sometihng worked. Ideally you shouldn't need to know how (which is imperative), being able to focus on what (which is declarative).

Ultimately we probably don't care if a click event triggered the sandwich to be eaten, or if it was a key press that triggered it. Maybe the button was focused and they hit the enter key. 

This is my opinion but I would say that In some sense the custom event simplifies our application because we're handling the _what_ of the event insted of the _how_ (click, key press, etc). 

In the following standard event example, MyLunchbox needs to be aware of which events might exist to bubbling up to it as clicks. The event handler needs to filter out the appropriate event, introspect the event (look inside of it), and then take the appropriate action. Imagine that our sandwich in our system has a special identifier. 

We could should out, "Eat #2" which happens to be the sandwich, emitting an eat event with a payload (data assocaited with the event) that is the food's identifier in your lunchbox.

The standard event equivalent would, "I'm doing a thing with my lunchbox stuff," requiring someone to then ask, "Ok, so um... what are you doing? Are you trying to eat something? What are you trying to eat? Does it have an identifier? Can I get that identifier?"

```rust
#[component]  
fn MyLunchbox(cx: Scope) -> Element {  
    let maybe_consume_sandwich = |event|{  
      // Introspection may be required in more
      // complicated use cases to make sure the 
      // right event bubbled up to be handled
      // and that it has the correct data to be
      // able to follow through with the desired
      // application behaviour.
    };  
    view!{  
        cx,  
        <Sandwich on:click=maybe_consume_sandwich/>  
    }  
}

#[component]  
fn Sandwich(cx: Scope) -> Element {  
    view!{  
        cx,  
        <button on:click=trigger_eating_event/>
	        I'm a snack
        </button> 
    }  
}

```

#### Caution: Beware complexity!

It can be tempting to cut your application up into a ton of domain specific—specific to the problem you're solving with language appropriate to that problem—events, but that comes at a cost. You will lose some forms of flexibility as you add focus and specificity to your application. 

In the standard event example, we do still have the ability to introspect the event when handling it in MyLunchbox. That might be really useful. If we needed some additional data with our custom event we'd need to go into the Sandwich component and include it.

And that's the thing with programming. It always depends.

I advocate for, favour simplicity and only cut things apart when they get too big to keep together. Some problems are inherently complicated because of the types of problems they are. Ideally you should be able to walk away from your program, come back, and understand what's happening. We can not rely on being in the flow state or "zone" as the required mode to understand what we wrote. I would say this is actually a liability. Besides, we should create applications that allow us to be interrupted by life without causing frustration. 

### Creating a custom event

Creating a custom event normally happens in JavaScript, because it's part of the browser's runtime. The code looks like this:

```javascript

const event = new Event('build');

// Listen for the event.
elem.addEventListener('build', (e) => { /* … */ }, false);

// Dispatch the event.
elem.dispatchEvent(event);

```
> https://developer.mozilla.org/en-US/docs/Web/Events/Creating_and_triggering_events

We need to do something similar in our Rust code. To do this we'll use the web_sys crate.

There is a struct in `web_sys` called `CustomEvent` [Docs](https://docs.rs/web-sys/latest/web_sys/struct.CustomEvent.html#method.new)

Let's go over some struct basics since we're going to e using them for this lesson and going forward.

#### Introduction to Structs

A struct is like a class in a lot of object oriented languages. It is a category of `type` that has the ability to group data, functionality related to that data, and functionality related to its general idea, all around a single name. Recall that a `type` is the name that describes a set of possible values.

##### Struct data

One of the key features of structs in Rust is that they specify a grouping of data types and values which we call properties. I suspect this is why they're called structs—structured data or data structure. If we had a Bacon Lettuce and Tomato sandwich struct it's definition would look like this:

```rust
struct BLTSandwich {
	bread: TypeOfBread,
	lettuce: TypeOfLettuce,
	tomato: TypeOfTomato,
	bacon: TypeOfBacon,
	mayo: bool
}
```
> The above example expects that TypeOfBread, TypeOfLettuce, TypeOfTomato, and TypeOfBacon are all defined earlier. They are used here to illustrate that BLTSandwich has constrained which values it's specific proeprties can have. You can not have a BLTSandwich with rocks as a value for bacon, because rocks are not a type of bacon! This is why type systems are important. They help prevent us from eating rocks... or... making mistakes in our programs. :)

If you have keen eyes you'll recognize something here. There's a pattern that we've seen a few times before.

```rust
// a function definition
fn function_name( parameter: type ) {}

// a struct definition
struct StructName{ property: type }
```

This pattern can be abstracted to the following:
- Rust keyword to define context/subject (`fn`, `struct`)
- A name to be able to use the noun(`function_name`, `StructName`)
- Some form of encapsulation with configuration

##### Make a new thing from an idea (a concretion)

A struct or structure is like an idea. And ideas aren't real in a sense that we can't hold them. We have an idea of what a BLT Sandwich is, but we can't eat the idea. But we have written specification for what the BLT is in the definition of our struct.

If we were to take **the idea of a  BLT sandwich** and _make_ **A BLT sandwich** we would say that we were making a concretion. A thing that is concrete or real. In object oriented programming (OOP) we would say that we are instantiating the idea (in oop ideas are classes). We are creating an instance of it. 

The syntax to create a struct includes writing the stuct's name, followed by curly braces, and a list of the property names and their values. 

```rust
// Assuming that the values for these 
// properties were already defined in scope
// with statements like
// let canadian_rye = get_the_best_sandwich_bread();
BLTSandwich{  
	bread: canadian_rye,
	lettuce: romaine,
	tomato: black_krim,
	bacon: farm_smoked_apple_bacon,
}
```

Most library (crate) authors write functions associated with a struct (with the idea of it) to make a concretion. It's convention for this function to be called 'new'. Calling the function follows this syntax:

```rust
	let my_thing = SomeStruct::new();
```

##### Functionality associated with the idea (static methods)

Structs can have functionality associated with the name of the struct. Some would described as functions that are namespaced, meaning that they are prefixed to or expected to be understood in the context of the name (being the struct's name).

The following showcases a `new` function in the `BLTSandwich` namespace which returns a new `BLTSandwich` (a concretion).

```rust
impl BLTSandwich {
	pub fn new() -> BLTSandwich {
		BLTSandwich{  
			bread: canadian_rye,
			lettuce: romaine,
			tomato: black_krim,
			bacon: farm_smoked_apple_bacon
		}
	}
	pub fn name() -> String {
		"Bacon, Lettuce and Tomato Sandwich".to_string()
	}
}
```
> Normally there would be parameters in the new function to accept arguments to configure the new thing being created. I skipped on that for the sake of simplicity. 

We can see here that we also have a name function which returns a long form name of the sandwich as a string.

We could call this by writing:
```rust
let the_sandwich_name : String = BLTSandwich::name();
```

Again, we can see that these are functions associated with the idea and separated by two colons. 

But again, look closely! A pattern emerges! We previously used a function called ``` leptos::log! ```. But leptos isn't a struct, it's a crate! 

Rust uses the same pattern of double colons to say, "We're setting the context to qualify which thing we're talking about". When we say BLTSandwich::name, we're telling Rust "Ok, think about BLTSandwich things... when I say name, you know what I'm talking about."

The Rust language designers have done a superb job at making these things easy to remember *if* you're aware that there is a pattern and design behind the decision. I can only assume that these design decisions were very deliberate.

##### Functionality associated with the a concretion (methods)

We can associate functionality with a specific concretion (a struct made real) which we often call methods. 

If we had a mthod called `calories` we could call with the following Rust code:

```rust
let sandwich = BLTSandwich::new();
let calories = sandwich.calories();
```

Here we make a sandwich and call calories on it.

The context here is so tightly coupled that we use a single dot as a separator. I like to think of it as this.

1) 4 dots — A namespace is a grouping of many things, so we use many dots.
2) 1 dot — A value is a single thing, so we use one dot.

The neat thing about the above is that if you didn't need to use sandwich you can chain these all together:

```rust
let calories = BLTSandwich::new().total_calories();
			   ^----------------^
				This will evaluate into a 'sandwich'
				which we can call total_calories() on.
```

Methods always have a special &self parameter as the first argument to denote that they're able to make reference to itself. This is how a function has the ability to do anything with it's own data. Recall that we can not use a piece of data unless it is in scope.

```rust

// Imagine that there is some function called calories, 
// which accepts things that can be turned into a calories
// value which is a 32 bit integer. Don't worry about how this
// would work. This is just a simple example.

impl BLTSandwich {
	// imagine the other static methods or namespace 
	// function from before were still here.

	pub fn total_calories(&self) -> i32 {
		calories(self.bread) + 
		calories(self.lettuce) + 
		calories(self.tomato) + 
		calories(self.bacon)
		// Recall that this function will evaluate to the 
		// last statement in its body. That's why there's no
		// semicolon at the end of this last item.
	}
	pub fn name() -> String {
		"Bacon, Lettuce and Tomato Sandwich".to_string()
	}
}
```

Note that we're able to use the value of the struct's properties with `.bread`.
If we look at the function `total_calories` it starts to look really similar to `.bread`. with the exception of us adding parenthesis at the end to call the fucntion. Yet another pattern emerges, methods on a value are properties on the value that you can call!

```rust
let sandwich = BLTSandwich::new();
sandwich.bread;
sandwich.total_calories; //<- but then we add () to call it
```

#### Using web_sys::CustomEvent
We're well positioned to use the web_sys crate's CustomEvent struct.

If we zip over to the documentation we can see that there is a new method on the struct [Docs](https://docs.rs/web-sys/latest/web_sys/struct.CustomEvent.html#):

```rust
web_sys::CustomEvent::new("my-custom-event");
```

But there's a notice under the definition of the `new` method that states the following: 

>_This API requires the following crate features to be activated: `CustomEvent`_

I did a quick search for "web_sys enable feature" which lead me to this support doc [Enable the cargo features for the APIs you're using](https://rustwasm.github.io/wasm-bindgen/web-sys/using-web-sys.html#enable-the-cargo-features-for-the-apis-youre-using).

My cargo.toml now has the following:

```toml
[dependencies]  
leptos = { git = "https://github.com/gbj/leptos" }  
  
[dependencies.web-sys]  
features = [  
    "CustomEvent"  
]
```

If we go back to the `new` method's definition in the web_sys::CustomEvent docs we'll see the following definition:

```rust
pub fn new(type_: &str) -> Result<CustomEvent, JsValue>
```

Notice that it returns `after the ->` a Result type, which has some type arguments (generics). The first one refers to what we get if new is run and the result is Ok, the second is the result that we get if new runs and the result is an Error. We can handle these with some in build pattern matching which we'll go into more later.

Our component code now looks like this:

```rust
#[component]  
fn MyComponent(cx: Scope) -> Element {  
    let trigger_sending_of_custom_event = |_|{  
        match web_sys::CustomEvent::new("my-custom-event") {  
            Ok(event) => {  
                // We have an event that we can send  
            },  
            Err(_) => {  
                // There as an error in creating the event  
                // We're not doing anything with this for now
				// so we'll use an '_' to destructure it's error
				// message            
			}  
        }  
    };  
    view!{  
        cx,  
        <div>  
            <button on:click=trigger_sending_of_custom_event>  
                "Trigger custom event"  
            </button>  
        </div>  
    }  
}
```

The `match` keyword requires that we create branches/arms for each possible option. Recall that we talked about types as restrictions that describe possible values. A result is an enumeration (a list of possible values or strict set of options) which can be one of two values. It can be Ok or Err. The options are called variants. 

In both of those cases there is a value that we can destructure out of the variants. Their types are listed as the the first and second type arguments in the returned type's signature. `Result<CustomEvent, JsValue>` means that we'll have an `Ok( CustomEvent )` or an `Err(JsValue)`.

I know from the JavaScript custom event documentation that it's not enough to create the event. We need to emit it. This is called `dispatching`. The javascript looks like this.

```javascript
elem.dispatchEvent(event);
```

What we need is some way to refer to our `<MyComponent />` so that we can dispatch the event on it. We need a reference to it.

### Getting a reference to self as a DOM node with NodeRef

Leptos provides us with the ability to get a reference to the DOM node created by its `view!` template. Think of it like a direct line to its DOM counterpoint.

The first step is to create the nodeRef, and add it as a special `_ref` property to the parent/root element in the `view!` template. Recall that the Leptos component is proxy for the `view!` template's root element. Putting the reference on this div is the same as putting the reference on `<MyComponent />`.

```rust
#[component]  
fn MyComponent(cx: Scope) -> Element {  
    let dom_node_ref = NodeRef::new(cx);
    // abbreviated/folded Rust code here for space saving
    view!{  
	    cx,  
	    <div _ref=dom_node_ref>  
	        <button on:click=trigger_sending_of_custom_event>  
	            "Trigger custom event"  
	        </button>  
	    </div>  
	}
}
```

The dom_node_ref uses signals under the hood so we can move it into our handler closure without stressing about move semantics. We'll add the move keyword to the closure  and we'll add some more matching if we are able to make our custom event.

```rust
match dom_node_ref.get() {  
	None => {  
		// None will only happen if this component isn't  
		// mounted to the DOM, but it has to be in order                   
		// for the click event to fire, so we can ignore this                    
	}  
	Some(dom_element) => {  
		// Emit/dispatch our custom event  
	}  
}  
```
> We call 'get' on the dom_node_ref to get the actual DOM element in Rust form. There are cases when the DOM element/node might not exist. Rust requires us to account for all possibilities, which is why the get method returns a option type. It's return type definition is `Option<web_sys::Element>`. Option is an enum which can be `None` or `Some` with the type argument provided in it's signature. In this case it's of the `web_sys::Element` type. We're destructing it and giving it the label `dom_element`

```rust
#[component]  
fn MyComponent(cx: Scope) -> Element {  
    let dom_node_ref = NodeRef::new(cx);  
  
    let trigger_sending_of_custom_event = move |_|{  
        match web_sys::CustomEvent::new("my-custom-event") {  
            Ok(event) => {  
                match dom_node_ref.get() {  
                    None => {  
                        // None will only happen if this component isn't  
		                // mounted to the DOM, but it has to be in order                   
			            // for the click event to fire, so we can ignore this                    
		            }  
                    Some(dom_element) => {  
                        // Emit/dispatch our custom event  
                    }  
                }  
            },  
            Err(_) => {}  
        }  
    };  
    view!{  
        cx,  
        <div _ref=dom_node_ref>  
            <button on:click=trigger_sending_of_custom_event>  
                "Trigger custom event"  
            </button>  
        </div>  
    }  
}
```

Intuitively, we'll probably want to try something like this for the actual event sending. This is a focused view of the happy path match arm:

```rust
match dom_node_ref.get() {  
    None => {}  
    Some(dom_element) => {  
        dom_element.dispatch_event(event);  
    }  
}
```

Unfortuantely this doesn't work. Rust tells us that dispatch_event is expecting a `&event`, a reference to an event. Let's add an ampersand before event to send a reference.

Rust's compiler may complain about unhandled results from the event dispatch. We can add another match statement to handle those.

```rust
match dom_element.dispatch_event(&event) {  
    Ok(_) => { 
	    leptos::log!("Custom event sent") 
	},  
    Err(_) => { 
	    leptos::log!("Failed to send") 
	}  
}
```

We can now listen to our custom event from our Leptos component:

```rust
#[component]  
fn RadApp(cx: Scope) -> Element {  
    let log_response = |_| {  
        leptos::log!("Our custom event happened")  
    };  
    view! {  
        cx,  
        <MyComponent on:myCustomEvent=log_response/>  
    }  
}
```
> Note that event names are camelCased

And just like that we have custom events on components with references! 

#### The Complete Code
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
        <MyComponent on:myCustomEvent=log_response/>  
    }  
}  
  
#[component]  
fn MyComponent(cx: Scope) -> Element {  
    let dom_node_ref = NodeRef::new(cx);  
  
    let trigger_sending_of_custom_event = move |_| {  
        match web_sys::CustomEvent::new("my-custom-event") {  
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