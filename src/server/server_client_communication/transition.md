# `<Transition />`
#async-bridge

>[Official documentation](https://docs.rs/leptos/latest/leptos/fn.Transition.html)

A transition is like suspense with one key difference; if the resources used in the children are updating (not initially loading), the current ui will stay in place until new UI can be created when the updated resource is ready. Transition also provides a pending or "updating" state that we can set to display loading messages.

Let's create a little example where we can add to a list of items. This is a pretty common pattern of update a parameter with interaction, query new data, and update the ui with the new data.

## The async function

To start, we'll make our async function.

Our function will need a delay so that we can simulate network latency. We'll add a crate to our project by typing the following in the terminal when in the rust project folder:

```bash
cargo add futures_timer
```

We can now use futures_timer to provide a WASM friendly async delay. Using standard thread sleeping would sleep the main thread of the application, including its async runtime. We don't want that.

Our function will accept a number and generate an array of numbers from 1 up to the count, so the return type will be a Vec of numbers.

We'll also add a delay from the `futures_timer` crate and await it so that the time passes before we return with our new array of enumerated items.

```rust
async fn pretend_external_list_items(count: u32) -> Vec<u32> {  

    futures_timer::Delay::new( 
	    std::time::Duration::from_secs(1)
	).await;  
	
    (1..=count).collect()  
}
```

## The component

Now we'll create our example component:

```rust
#[component]  
fn TransitionExample(cx: Scope) -> impl IntoView {  
	  
}    
```

We'll setup a resource like we did in the server functions tutorial.

```rust
#[component]  
fn TransitionExample(cx: Scope) -> impl IntoView {  
	
	// we create a signal that stores the size of the list 
    let (list_size, set_list_size) = create_signal(cx, 1_u32); 

	// we use the size as arguments for our 
	// function as a resource
    let list_items = create_resource(
	    cx, 
		list_size, 
		pretend_external_list_items
	);
}
```

We'll now add a view to our component that uses our resource. We'll start with a `<Transition>` tag and add a fallback view, just like with `<Suspense>` items. Recall that Transition is like `<Suspence>` but with glitter.

`<Transition>`s children will be set to a view that contains a `<For />` tag to loop/iterate over the resource's value. The async function returns an option so we'll need to `read()` it and then `upwrap_or_default()`. Then for the view we create output for each item as an HTML list item.

```rust
#[component]  
fn TransitionExample(cx: Scope) -> impl IntoView {  
	
	let (list_size, set_list_size) = create_signal(cx, 1_u32); 

	let list_items = create_resource(
	    cx, 
		list_size, 
		pretend_external_list_items
	);
	
	view!{cx,  
	    <Transition  
	        fallback=move||view!{cx, <p>"Loading"</p>}  
		> 
			{view!{cx,  
	            <ul>  
	                <For  
	                    each=move||list_items.read().unwrap_or_default()  
	                    key=move|item|item.clone()  
	                    view=move|item|view!{cx,<li>{item}</li>}  
	                />            
				</ul>  
		        }       
			}    
		</Transition>  
	}
}
```

To make this more interactive we'll add a button that increments the list size:

```rust
<button on:click=move|_| set_list_size.set( list_size.get() + 1 ) >  
    "Add an item"  
</button>
```

And we'll add a header that shows us how big the list should be. This is a nice indicate of expectation on action, showing the delay between the header being updated and the list growing.

```rust
	<h2>"A list of " {list_size} </h2>
```

When assembled we're left with the following component:

```rust
```rust
#[component]  
fn TransitionExample(cx: Scope) -> impl IntoView {  
	
	let (list_size, set_list_size) = create_signal(cx, 1_u32); 

	let list_items = create_resource(
	    cx, 
		list_size, 
		pretend_external_list_items
	);
	
	view!{cx,  
		<h2>"A list of " {list_size} </h2>
		<button on:click=move|_| set_list_size.set( list_size.get() + 1 ) >  
		    "Add an item"  
		</button>
	    <Transition  
	        fallback=move||view!{cx, <p>"Loading"</p>}  
		> 
			{view!{cx,  
	            <ul>  
	                <For  
	                    each=move||list_items.read().unwrap_or_default()  
	                    key=move|item|item.clone()  
	                    view=move|item|view!{cx,<li>{item}</li>}  
	                />            
				</ul>  
		        }       
			}    
		</Transition>  
	}
}
```

The last piece of this example is adding a signal to hold the boolean value for if the transition component is updating (by default it won't be because it'll be loading):

```rust
let (is_list_updating, set_is_list_updating) = create_signal(cx, false);
```

And we'll update the `<Transition>` component, setting the property to accept the signal.

```rust
<Transition  
    fallback=move||view!{cx, <p>"Loading"</p>}  
    set_pending=set_is_list_updating.into()  
>
```

The above  `into()` call on `set_is_list_updating` might look weird to you. The optional property `set_pending` accepts a `SignalSetter<bool>`. We know that `set_is_list_updating` is a `WriteSignal<bool>`. An implementation is written allowing us to turn the `WriteSignal` into the `SignalSetter`, which we can perform with the `into()` method.

We'll use our hand `<Show>` Leptos UI tag to conditionally display an updating message with a fallback of an empty view.

```rust
<Show  
    when=move || is_list_updating.get()  
    fallback=|_| ""  
>  
    <p><i>"Updating the list"</i></p>  
</Show>
```

When all is said and done we're left with a nice clear example of all of the great features wrapped up in `<Transition>`, showcasing how it works with some of Leptos' event handlers, signals, and UI tags.

```rust
use leptos::*;  
  
#[component]  
pub fn App(cx: Scope) -> impl IntoView {  
    view! {  
        cx,  
        <TransitionExample/>  
    }}  
  
#[component]  
fn TransitionExample(cx: Scope) -> impl IntoView {  
  
    let (list_size, set_list_size) = create_signal(cx, 1_u32);  
    let list_items = create_resource(cx, list_size, pretend_external_list_items);  
  
    let (is_list_updating, set_is_list_updating) = create_signal(cx, false);  
  
    view!{cx,  
        <h2>"A list of " {list_size} </h2>  
  
        <button on:click=move|_| set_list_size.set( list_size.get() + 1 ) >  
            "Add an item"  
        </button>  
  
        <Show  
            when=move || is_list_updating.get()  
            fallback=|_| ""  
        >  
            <p><i>"Updating the list"</i></p>  
        </Show>  
  
        <Transition  
            fallback=move||view!{cx, <p>"Loading"</p>}  
            set_pending=set_is_list_updating.into()  
        >           
			{ view!{cx,  
                <ul>  
                    <For  
                        each=move||list_items.read().unwrap_or_default()  
                        key=move|item|item.clone()  
                        view=move|item|view!{cx,<li>{item}</li>}  
                    />
				</ul>  
	            }
			}        
		</Transition>  
    }  
}  
  
async fn pretend_external_list_items(count: u32) -> Vec<u32> {  
    futures_timer::Delay::new( std::time::Duration::from_secs(1)).await;  
    (1..=count).collect()  
}
```