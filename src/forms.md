# Forms

## What we know
- We can capture events and respond to them
- Signals allow us to persist data across events

## What we'll learn
- How to respond to multipart form data

## The Lesson

Back in the day we used to interact with websites by submitted form data to a server with a requested resource (like a specific page). The page would render, often using or processing the form data that was sent with it, would generate HTML, and then provide us a response. This is how the majority of the web still works to this day!

We're going to replicate a similar data flow so that you can collect sets of data using forms, but process them all on the client (in the browser).

We'll start with a Rad app component and some boiler plate, mounting it to the body:

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
        <form>  
        </form>  
    }  
}
```

Our form has no submit button, so we'll add that in:

```rust
#[component]  
fn RadApp(cx: Scope) -> Element {  
    view! {  
        cx,  
        <form>  
	        <input type="submit" value="Submit"/>
        </form>  
    }  
}
```

We now have a basic form that we can submit. If we click submit, the form takes it data (of which there is none) and posts it to the form's action destination, which if non will default to the current page. This effectively looks like the page has reloaded even though it's actually being re-requesed with the updated form data.

Let's add a text field so that we can submit some data.

```rust
<form>  
	<input type="text"  
	    name="fav_thing_to_paint"  
	    placeholder="Your fav thing to paint..."  
	    value=""  
	/>
	<input type="submit" value="Submit"/>
</form>  
```
> I broke the lines up here purely for formatting. HTML elements don't care about line breaks inbetween attributes.

If we type in something into the text field and hit submit, you'll see the page load, and the field gets reset. What is happening here is that the form is taking its form data and submitting it to the form's action url as part of a new request. The action property is a property on the form element telling the form where to send its data. It defaults to the current page that it's on if the action is not set.

If we had `<form action="https://www.rust-lang.org">` and we clicked submit, the form would send our data to rust-lang.org! And isntead of looking like a page reload, we'd see the rust-lang.org home page.

We always have to remember that here we're just making a more complicated request with some configuration (the form data) and our brower is rendering the response.

In old school website, a server would render a template and process form submissions for that template at the same time. If a request came in without form data, the fields would be blank. If a request came in that had posted data (submitted via the form submission), whoever coded the form template which is processed on the server could pluck out that posted data, and enter in the submitted values as the values of the input fields in the form. This way form submission data doesn't get erased if, for example, some form validation failed. The data just gets passed back and forth. It is not persisting anywhere.

>Fun Fact! Forms default to 'post' as their method of sending data. You can change this method to 'get' and your data will become query string variables.

### Responding to the event

Let's add a from handler for the `submit` event. But this point, things should look pretty familiar.

```rust
#[component]  
fn RadApp(cx: Scope) -> Element {  
	
	// We create a form handler
    let form_handler = |_|{  
        leptos::log!("The form was submitted");  
    };  

	// And we added it with `on:submit` to the form element
    view! {  
        cx,  
        <form on:submit=form_handler>  
            <input type="text"  
                name="fav_thing_to_paint"  
                placeholder="Your fav thing to paint..."  
                value=""  
            />  
            <input type="submit" value="Submit" />  
        </form>  
    }  
}
```

### Preventing the form from sending

When we click submit, the form submits so quickly that we can't even see the `form_handler`'s message. Also, we're working on a client side application in this context, so we don't want this page to reload and rerender. We want to _prevent the default behaviour_.

To do this we need to actually do something with the event in our event handler that we've been ignoring this whole time. Let's change it from an underscore to something easy to understand, like `submission_event`. 

```rust
	let form_handler = |submission_event|{  
        submission_event.prevent_default();
    };  
```

The above won't work though, because the closure doesn't know where it will be used. Rust doesn't know that this closure will be called from the event system and that the first argument will be an event. To fix this problem we'll give it a type `web_sys::SubmitEvent`.

```rust
	let form_handler = |submission_event: web_sys::SubmitEvent|{  
        submission_event.prevent_default();
    };  
```

Calling prevent_default() on the submit event will prevent the form from actually being submitted. We've short circuited the default behaviour!

>Sometimes I find that I don't know exactly what to write for the type so I'll put in some form of type, try to compile the application, and then let Rust's compiler tell me what was supposed to be there. It's right most of the time.

### Capturing form data

Events have the source stored at a proeprty called `target`. We can grab the element that emitted the event by calling it. 

```rust
let form_handler = |submission_event: web_sys::SubmitEvent|{  
	submission_event.prevent_default();
	let form = submission_event.target();
};  
```

We don't know for sure if the target will actually be a proper element. The return type of the `target` method is `Option<EventTarget>`. As we learned in the previous lessons, we can match on the form's value to account for Some(form) or None.

```rust
let form_handler = |submission_event: web_sys::SubmitEvent|{  
    submission_event.prevent_default();  
    match submission_event.target() {  
        None => {},  
        Some(form_event_target) => {  
            // we need to do things here
		}  
    }  
};
```

form_event_target doesn't have a specific type yet, so we need to explicitly tell Rust, "Hey, this is a HtmlFormElement" which we need to derive a form data object. 

>It should be noted that it took research to sort through this which is why I'm presenting it to you. This way you have one place to look it all up. :)

We're going to add the following line once we've destructured our `form_event_target`.

```rust
let form_element = form_event_target.unchecked_ref::<web_sys::HtmlFormElement>();  
```
> Here we take our target, which is untyped and called `unchecked_ref()` to type it. We add a turbofish `::<SomeType>` between the name of the method and the parenthesis to specify the generic type. In this case, it's the type that it will become when we call unchecked_ref on it.

This will fail to work, and Rust's compiler wil complain. If we look at the definition of web_sys::HtmlFormElement we'll see that it needs to be set as a feature dependency in cargo.toml.

We'll add the following to our cargo.toml to ensure that websys uses the two features we'll need:

```toml
[dependencies.web-sys]  
features = [ "FormData", "HtmlFormElement"]
```

Next we'll setup form data which will use data from the form element. 

```rust
let form_data = web_sys::FormData::new_with_form(&form_element);
```

This returns a result type, with its return type being `Result<FormData, JsValue>`. As we've sen before, we'll need to destructure it to pull out the value that is of type `FormData`.

```rust
let form_data = web_sys::FormData::new_with_form(&form_element);  
match form_data{  
    Err(_) => {},  
    Ok(data) =>{  
        // the data here is a FormData thing.
    }  
}
```

FormData has some useful methods, one of which we can use to extract values from fields by name.

```rust
let fav_thing = data.get("fav_thing_to_paint").as_string();
```
> Here we ask the form data to give us its value for "fav_thing_to_paint" as a string value. This is still an option, so we'll have to deal with `Some(the_value)` or `None`.

I'm specifically showing you pattern matchin as the simplest way to deal with these result and option types. There are many shorter ways of doing that which you will learn later.

It is also possible to inline the match statement and avoid assigning the temporary variable. We could write the following:

```rust
let fav_thing = data.get("fav_thing_to_paint").as_string();  
match fav_thing {  
    Some(actual_fav_thing_value) => {},  
    None => {}
}
```

or

```rust
match data.get("fav_thing_to_paint").as_string() {  
    Some(fav_thing) => {},  
    None => {}
}
```

The whole thing all together looks like this:

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
    let form_handler = |submission_event: web_sys::SubmitEvent|{  
        submission_event.prevent_default();  
        match submission_event.target() {  
            None => {},  
            Some(form_event_target) => {  
                let form_element = form_event_target.unchecked_ref::<web_sys::HtmlFormElement>();  
                let form_data = web_sys::FormData::new_with_form(&form_element);  
                match form_data{  
                    Err(_) => {},  
                    Ok(data) =>{  
                        match data.get("fav_thing_to_paint").as_string() {  
                            Some(fav_thing) => {  
                                leptos::log!("{:?}", fav_thing);  
                            },  
                            None => {}  
                        }  
                    }  
                }  
            }  
        }  
    };  
    view! {  
        cx,  
        <form on:submit=form_handler>  
            <input type="text"  
                name="fav_thing_to_paint"  
                placeholder="Your fav thing to paint..."  
                value=""  
            />  
            <input type="submit" value="Submit" />  
        </form>  
    }  
}
```

### Adding signals

We can now create a signal and use it to store the posted/submitted data.

```rust
let (last_fav_thing, set_last_fav_thing) = create_signal(cx, String::new());
```

We will add `move` to the handler, so that we can move the signal into it:

```rust
let form_handler = move|submission_event: web_sys::SubmitEvent|{
```

And we'll store the value using the signal:

```rust
set_last_fav_thing(fav_thing);
```

The last piece is displaying the last submission in our `view!` template:

```rust
<p>"Your last fav thing was: " {last_fav_thing}</p>
```

All togehter we have a nice example of how to collect form data so that we can work with it!

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
  
    let (last_fav_thing, set_last_fav_thing) = create_signal(cx, String::new());  
  
    let form_handler = move|submission_event: web_sys::SubmitEvent|{  
        submission_event.prevent_default();  
        match submission_event.target() {  
            None => {},  
            Some(form_event_target) => {  
                let form_element = form_event_target.unchecked_ref::<web_sys::HtmlFormElement>();  
                let form_data = web_sys::FormData::new_with_form(&form_element);  
                match form_data{  
                    Err(_) => {},  
                    Ok(data) =>{  
                        match data.get("fav_thing_to_paint").as_string() {  
                            Some(fav_thing) => {  
                                set_last_fav_thing(fav_thing);  
                            },  
                            None => {}  
                        }  
                    }  
                }  
            }  
        }  
    };  
    view! {  
        cx,  
        <form on:submit=form_handler>  
            <p>"Your last fav thing was: " {last_fav_thing}</p>  
            <input type="text"  
                name="fav_thing_to_paint"  
                placeholder="Your fav thing to paint..."  
                value=""  
            />  
            <input type="submit" value="Submit" />  
        </form>  
    }  
}
```