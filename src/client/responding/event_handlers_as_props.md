# Event Handlers as Props

## What we know
- We sometimes want to handle events at higher levels in the Leptos component tree

## What we'll learn
- How to pass event handlers down the component tree

## The Lesson

We know that we can setup click handlers on buttons. Here's a quick example:

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
    view! {  
        cx,  
        <ChildComponent/>  
    }
}  
  
#[component]  
fn ChildComponent(cx: Scope) -> Element {  
    let click_handler = |_|{  
        leptos::log!("Child button clicked");  
    };  
    view! {  
        cx,  
        <div>  
           <button on:click=click_handler>  
                "Click me"  
            </button>  
        </div>  
    }
}
```

We may want to delegate the responsibility of handing this event to a parent or ancestor of the initial event target. In this case, the originating DOM node, the event target, would be the button.

We previously solved this problem by having our event bubble up. Our `click` dispatched by the `ChildComponent` bubbles up to the parent `RadApp` which responds via `on:click` as seen here:

>Note that we needed to add `use web_sys::MouseEvent;`

```rust
use leptos::*;  
use web_sys::MouseEvent;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <RadApp />  
        }    })}  
  
#[component]  
fn RadApp(cx: Scope) -> Element {  
    let click_handler = |_|{  
        leptos::log!("Child button clicked");  
    };  
  
    view! {  
        cx,  
        <ChildComponent on:click=click_handler/>  
    }}  
  
#[component]  
fn ChildComponent(cx: Scope) -> Element {  
    view! {  
        cx,  
        <div>  
           <button>  
                "Click me"  
            </button>  
        </div>  
    }
}
```

The problem here is that we would need to introspect the event to differentiate which thing was clicked to make sure we're responding to the correct event.

We can solve this problem by dispatching a custom event, but we have extra complexity in adding custom events. It should also be noted that there is overhead (a computational cost that decreases performance) to adding additional event handlers.

### Strategy pattern, sort of

One solution to the aforementioned problem is to pass the handler down as a property. We can send what should happen down to where it happens so that it can be called. This is an inversion of having what happened bubbling up. We're sending what to do on down.

First, we need to send our click_handler as a property to the `ChildComponent`. 

```rust
#[component]  
fn RadApp(cx: Scope) -> Element {  
  
    let click_handler = |_|{  
        leptos::log!("Child button clicked");  
    };  
  
    view! {  
        cx,  
        <ChildComponent onclick=click_handler/>  
    }
}  
```

Next is the tricky part. We need to accept the value of the `click_handler` (a closure) into `ChildComponent`. We do this by declaring it as a function property of the component. We need to use a generic, herein referred to as `T`. When we add `on:click=onclick` to the `button` we're attaching the handler to the `click` event. This will trigger Rust to say, "HEY, if you're using this value for `on:click`, it has to be a specific type of closure!" We need to add some specificity to the closure.

We can do this by adding `where T: Fn(MouseEvent) -> () + 'static ` to the end of the function definition, but before the scope of the function's body opens. This tells Rust that the function parameter `onclick` can be `anything` provided that `anything` implements `Fn(MouseEvent) -> ()`. This is a `function trait`. It is also required to be `'static` lifetime, so we'll add that as well with `+ 'static`. The static lifetime tells Rust that this data will live for the duration of the application's running.

By doing this we can safely pass a callback that matches the requirements of the `on:click` assignment, which is verified at the component property level!

```rust
#[component]  
fn ChildComponent<T>(cx: Scope, onclick:T ) -> Element  
    where T: Fn(MouseEvent) -> () + 'static  
{  
    view! {  
        cx,  
        <div>  
           <button on:click=onclick>  
                "Click me"  
            </button>  
        </div>  
    }
}

```

I'll update some of the names to disambiguate between `on:click` and `onclick`. Here's the final code. 

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
  
    let log_clicked_button = |_|{  
        leptos::log!("Child button clicked");  
    };  
  
    view! {  
        cx,  
        <ChildComponent click_logger=log_clicked_button/>  
    }
}  
  
#[component]  
fn ChildComponent<T>(cx: Scope, click_logger:T ) -> Element  
    where T: Fn(MouseEvent) -> () + 'static  
{  
    view! {  
        cx,  
        <div>  
           <button on:click=click_logger>  
                "Click me"  
            </button>  
        </div>  
    }
}

```
