# Form Actions with `<ActionForm>`

As we've seen from the other form lessons, working with forms can involve some work to hook things up. Leptos comes with a special component called an ActionForm which allows us to more easily set an action as a form handler. Even better yet, the form will attempt to resolve client side and fallback to the server. This is exactly what we want for progressively enhanced applications.

This article makes the assumption that you've reviewed [server functions](./server_functions.md) and have a basic cargo leptos setup.

## Initial Setup

	Before we can start we'll need to add Serde.

```bash
cargo add serde
```


We'll start by creating a route for our form page.

```rust
// app.rs

#[component]  
pub fn App(cx: Scope) -> impl IntoView {  
    view! {  
        cx,  
        <Router>  
            <Routes>  
                <Route path="" view=|cx| view! { cx, <FormPage/> }/>  
            </Routes>  
        </Router>  
    }}
```

We need to setup the Leptos Component that will handle our root route with path "". We're calling it `FormPage`. This page will have a form that accepts a user's super secret message.

```rust
// app.rs

#[component]  
fn FormPage(cx: Scope) -> impl IntoView {  
    view! { cx,  
        <form>  
            <input type="text" name="secret" placeholder="Tell me a secret" />  
            <input type="submit" value="submit" />  
        </form>  
    }
}
```

The above component uses a standard html `form` element. We're going to upgrade this to Leptos' `ActionForm` element (note the capitalization).

```rust
// app.rs

#[component]  
fn FormPage(cx: Scope) -> impl IntoView {  
    view! { cx,  
        <ActionForm>  
            <input type="text" name="secret" placeholder="Tell me a secret" />  
            <input type="submit" value="submit" />  
        </ActionForm>  
    }
}   
```

Next we'll need to create a server action and provide it as the argument for the `ActionForm`'s `action` parameter.

We'll start by creating a server action which will be called `MyServerAction` `(1)` and adding the action to our `ActionForm` `(2)` .

```rust
// app.rs

#[component]  
fn FormPage(cx: Scope) -> impl IntoView {  
	
	let my_form_action = create_server_action::<MyServerAction>(cx); // 1
	
	view! { cx,  
        <ActionForm action=my_form_action>  // 2
            <input type="text" name="secret" placeholder="Tell me a secret" />  
            <input type="submit" value="submit" />  
        </ActionForm>  
    }
}
```

We need to write the function that will process our server action. Note the server function name `MyServerAction` is listed in the macro. This is where the server function receives its name `(1)`. The name of the function itself is usually similar as convention but it is not a requirement.

```rust
// app.rs

#[server(MyServerAction, "/api")] // 1 
pub async fn my_server_action(cx: Scope ) -> Result<(), ServerFnError> {  
    println!("Form submitted");  
    Ok(())  
}
```

And we'll register this server function `(1)` in our applications entry point. We'll also comment out the simple_logger `(2)` in the cargo leptos template so that we can more easily see the "Form submitted" log message from our server action `MyServerAction`.

```rust
// main.rs

#[cfg(feature = "ssr")]  
#[tokio::main]  
async fn main() {
	let _ = MyServerAction::register(); // 1
	
	// simple_logger::init_with_level(log::Level::Debug)
	//	.expect("couldn't initialize logging"); // 2

	// ...
	}
```

## Using ActionForm Data

The really neat thing about Leptos ActionForm is that the names of fields become arguments to the server function. As a result, we can easily get the values of the form fields without needing to pick apart the request parts with something like `use_context::<leptos_axum::RequestParts>(cx)`.

```rust
#[server(MyServerAction, "/api")]  
pub async fn my_server_action(cx: Scope, secret: String ) -> Result<(), ServerFnError> {  
    println!("Form submitted with {}", secret);  
    Ok(())  
}
```

## Returning ActionForm Data

We can return data from our action form to use in our view by calling the `value()` method on the server_action `(1)`. We'll print it out to the screen to get a better sense of how this all works together `(2)`.

```rust
#[component]  
fn FormPage(cx: Scope) -> impl IntoView {  
  
    let my_form_action = create_server_action::<MyServerAction>(cx);  
  
    // value contains most recently-returned value  
    let form_value = my_form_action.value();  // 1
  
    view! { cx,  
        <ActionForm action=my_form_action>  
            <input type="text" name="secret" placeholder="Tell me a secret" />  
            <input type="submit" value="submit" />  
        </ActionForm>  
        "Returned from the server fn: " {form_value} // 2
    }}
```

We'll need our server function to actually return something for this to work. To do this we';; change the return type to `Result<String, ServerFnError>` and provide an `Ok(String)` as the last statement to hard code the return value.

```rust
#[server(MyServerAction, "/api")]   
pub async fn my_server_action(cx: Scope ) -> Result<String, ServerFnError> {  // 1
    println!("Form submitted");  
    Ok("Server function result".to_string())  // 2
}
```

We could combine these two ActionForm features to echo the secret back out.

```rust
#[server(MyServerAction, "/api")]
pub async fn my_server_action(cx: Scope, secret: String ) -> Result<String, ServerFnError> {  
    let echoed_string_from_server = format!("{} from server", secret);  
    Ok(echoed_string_from_server)  
}
```

