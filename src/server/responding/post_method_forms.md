# Post Method Forms (Form to Request Body)

A very old pattern in web development involves setting up routes which will be the action targets for forms. Their post data would be handled on that page. In fact, you could serve pages that were the result of a `POST` request type. 

Leptos enforces a separation where routes respond to `GET` request types (urls or using `method="get"`), and `POST` request types are handled with server functions. They will provide a blank page as a response because they're not expected to respond. You can think of `GET` as a pull and `POST` as a push. We can, however, redirect after a `POST`/push to send the user to a route.

#### Server action dependencies

Server actions require communication between the client and the server. This requires data to be serialized and deserialized to transport the data from one to the other. We'll need to add the serde library to our `cargo.toml` to take care of this.

```toml
serde = {version = "1.0.152", features = ["derive"] }
```

#### Server actions hooked into the router

Leptos uses server functions to handle form actions. If we look at the main.rs of Cargo Leptos's setup file we'll see the following:

```rust
// main.rs

let app = Router::new()  
    .route("/api/*fn_name", post(leptos_axum::handle_server_fns))  
    .leptos_routes(leptos_options.clone(), routes, |cx| view! { cx, <App/> })  
    .fallback(file_and_error_handler)  
    .layer(Extension(Arc::new(leptos_options)));
```

I'd like to draw your attention to this line:

```rust
.route("/api/*fn_name", post(leptos_axum::handle_server_fns))  
```

Here we're setting up routes with the prefix "/api" followed by the function name. This is where server functions are hooked into the router to handle our `POST` requests.

#### Setting up the routes

We start off with two routes, the one that has the form and our destination after the form as been submitted.

```rust
// app.rs

use leptos::*;  
use leptos_router::*;  
  
#[component]  
pub fn App(cx: Scope) -> impl IntoView {  
    view! {  
        cx,  
        <Router>  
            <Routes>  
                <Route path="" view=|cx| view! { cx, <FormPage/> }/>  
                // We will not have a handler route here, 
                // because it will be created from the action  
                // <Route 
	            //    path="/form-action-handler" 
	            //    view=|cx| view! { cx, <FormActionHandler /> }
	            //  />                
	            <Route 
		            path="/form-action-processed" 
		            view=|cx| view! { cx, <FormActionProcessed /> }
				/>  
            </Routes>  
        </Router>  
    }}
```

We'll need some components setup which we'll do now:

```rust
// app.rs

#[component]  
fn FormPage(cx: Scope) -> impl IntoView {  
    view! { cx,  
        <form action="/api/form-action-handler" method="post">  
            <input 
	            type="text" 
	            name="secret" 
	            id="secret" 
	            placeholder="Tell me a secret" 
			/>  
            <input 
	            type="submit" 
		        value="Send request" 
			/>  
        </form>  
    }
}

#[component]  
fn FormActionProcessed(cx: Scope) -> impl IntoView {  
    view!{cx, 
	    "Server side response. This should 
	    display as a result of submitting the form."
	}  
}
```

>Note that our action is now located at `/api/form-action-handler`.

#### Setting up the server functions

We'll add our server function:

```rust
// app.rs

#[server(FormActionHandler)]  
async fn form_action_handler(cx: Scope) -> Result<(), ServerFnError> {  
    println!("Form submitted");  
    Ok(())  
}
```

And we need to register the server function in our system before the routes are generated, so that a route can be generated for it. The `#[server(FormActionHandler)]` macro will expand and create a handle for us to register the derived server function. Keep in mind that the macro is writing a lot of boiler plate code for us. We're just adding the implementation here and declaring the intent. The handle to the macro expanded server function is `FormActionHandler`.

We'll add our registration call in our main.rs file. We don't need to prefix this with anything because it's in the top level of our library.rs. (it's not in a sub module). 

```rust
// main.rs

let _ = FormActionHandler::register();

```

If we run our application with `cargo leptos watch` at this point we'll get something that doesn't quite work. Submitting a form will take you to a page with the following error:

```
Could not find a server function at the route form-action-handler. 

It's likely that you need to call ServerFn::register() on the server function type, somewhere in your `main` function.
```

Leptos generates its own special name spaced URL for the action. If we add the following code in `main.rs` we can get rust to spit out the actual "url".

```rust
let _ = FormActionHandler::register();

// 2 new temporary lines 
println!("{:?}", FormActionHandler::url() );
return;                                 
```

Which happens to be `src-app.rs-form_action_handler`. 

Delete those two lines we added so that the application will run as expected and let's update our form with the new url part.

```rust
#[component]  
fn FormPage(cx: Scope) -> impl IntoView {  
    view! { cx,  
        <form action="/api/src-app.rs-form_action_handler" method="post">  
            <input type="text" name="secret" id="secret" placeholder="Tell me a secret" />  
            <input type="submit" value="Send request" />  
        </form>  
    }}
```

If we run our application now, we'll see "Form submitted" in the `cargo leptos` log stream. Commenting out the following line in main.rs will silence the debug log, making your dev logging easier to read:

```rust
// simple_logger::init_with_level(log::Level::Debug).expect("couldn't initialize logging");
```

#### Redirecting on submission

So now we're capturing the action, but we'd like to go to our destination "processed" page. We can use the 

```rust
#[server(FormActionHandler)]  
async fn form_action_handler(cx: Scope) -> Result<(), ServerFnError> {  
    println!("Form submitted");  
    leptos_axum::redirect(cx, "/form-action-processed");  
    Ok(())  
}
```

>Important note about SSR (Server Side) vs CSR (Client Side): If you write a use statement like `use leptos_axum::redirect;`, your server side binary will compile. You will receive an error compiling the client side version because `leptos_axum` is not a dependency for the client side code. When a server macro is expanded it runs in the context of the server code where `leptos_axum` _is_ included. For this reason we can write `leptos_axum::redirect()` and be ok. A use statement exists in both server and client contexts. To solve this problem we can wrap the use statement in a conditional config. 
```rust
cfg_if::cfg_if! {  
    if #[cfg(feature = "ssr")] {  
       use leptos_axum::redirect;  
    }  
}
```

#### Capturing post data

We want to capture form data from a form submission. We can do this in the server function by introspecting the request parts which are stored in the context provided to the server function.

We'll use `use_context::<leptos_axum::RequestParts>(cx)` to get the context with our type as a parameter to extract those components from the context.

We can match over these to start pulling data out. But the body of our parts, where our form data is stored, is in a bytes. We'll need to pass a reference to the "body" of our parts and convert the byes into a string slice `&str`. 

```rust
match use_context::<leptos_axum::RequestParts>(cx) {  
    Some(parts) => {  
        let body: &str = std::str::from_utf8(&parts.body).unwrap_or_default();  
    },  
    None => {}  
}
```

`body` at this point will look like a query string with `key=value&key=value` formatting. We can naively parse this by splitting the string on ampersand  (`&`) characters to get the key=value pairs. This is an extremely naive implementation and doesn't account for a variety of edge cases.

```rust
let body= std::str::from_utf8(&parts.body).unwrap_or_default();  
let data: Vec<Vec<String>> = body  
    .split('&')  // split the string into key=value, key=value
    .map(|kv| {  // convert the split kv strings into an array of [key, value]
        kv.split('=').collect()  
    })
	.collect();
```

It would be a better idea to use the [form_urlencoded](https://docs.rs/form_urlencoded/latest/form_urlencoded/) crate. The above is included for educational purposes.

Now we need something to store our secret in.

```rust
#[derive(Debug)]
struct SecretData(String);
```

And we'll implement default for this as well.

```rust
impl Default for SecretData {  
    fn default() -> Self {  
        Self("".to_string())  
    }
}
```

We can now kind of hack together a parser that will loop (iterate) over the key/value pairs to pluck the data out.

```rust
let form_data = SecretData::default();

for key_val_pairs in data.iter() {  
    match key_val_pairs.get(0).map(|k|k.deref()) {  
        Some("secret") => {  
            match key_val_pairs.get(1) {  
                Some(data) => {  
                    form_data = SecretData( data.to_string());  
                }  
                _ => {}  
            }  
        },        
        _ => {}  
    }
}

```
> This is a verbose version to show you the different stages of unwrapping. 

The whole thing looks pretty gnarly though:

```rust
let mut form_data = SecretData::default();  
  
match use_context::<leptos_axum::RequestParts>(cx) {  
    Some(parts) => {  
        let body= std::str::from_utf8(&parts.body).unwrap_or_default();  
        let data: Vec<Vec<&str>> = body  
            .split('&')  
            .map(|kv| {  
                kv.split('=').collect()  
            })            .collect();  
  
        for key_val_pairs in data.iter() {  
            match key_val_pairs.get(0).map(|k|k.deref()) {  
                Some("secret") => {  
                    match key_val_pairs.get(1) {  
                        Some(data) => {  
                            form_data = SecretData( data.to_string());  
                        }  
                        _ => {}  
                    }  
                },                
                _ => {}  
            }        
		}        
		println!("{:?}", form_data );  
  
    },  
    None => {}  
}
```

Let's wrap this all up in a nice function with some early returns to make the code less nested.

```rust
#[cfg(feature = "ssr")]  
fn parse_secret_data(cx: Scope) -> SecretData {  
    let parts = match use_context::<leptos_axum::RequestParts>(cx){  
        None => return SecretData::default(),  
        Some(parts) => parts  
    };  
  
    let body = match std::str::from_utf8(&parts.body) {  
        Err(_) => return SecretData::default(),  
        Ok(data) => data  
    };  
  
    let key_val_pairs: Vec<Vec<&str>> = body  
        .split('&')  
        .map(|kv| kv.split('=').collect() )  
        .collect();  
  
    for kvp in key_val_pairs.iter() {  
        match ( 
	        kvp.get(0).map(|k|k.deref()), 
	        kvp.get(1).map(|k|k.deref()) 
		) {  
            ( Some("secret"), Some(data)) => return SecretData( data.to_string() ),  
            _ => return SecretData::default(),  
        }    }  
    SecretData::default()  
}
```


#### Forwarding post data to the displayed/redirected route

The server isn't carrying state between the redirects. And actions are run independently of other aspects of Leptos. You can think of them as mini programs. We can, however, set a cookie that carries the data back to the client. We can also clear the data when we reach the destination page so that it's a short lived secret. 

We'll use the following function to set our cookie:

```rust
#[cfg(feature = "ssr")]  
fn set_cookie(cx: Scope, name: &str, value: &str ) {  
    use axum::http::header::{HeaderMap, HeaderValue, SET_COOKIE};  
    use leptos_axum::{ResponseOptions, ResponseParts};  
  
    let response = use_context::<ResponseOptions>(cx)
        .expect("to have leptos_axum::ResponseOptions provided");  
    let mut response_parts = ResponseParts::default();  
    let mut headers = HeaderMap::new();  
    headers.insert(  
        SET_COOKIE,  
        HeaderValue::from_str(&format!("{name}={value}; Path=/"))  
            .expect("to create header value"),  
    );  
    response_parts.headers = headers;  
    response.overwrite(response_parts);  
}
```

We'll call set cookie from our handler:

```rust
#[server(FormActionHandler)]  
async fn form_action_handler(cx: Scope) -> Result<(), ServerFnError> {  
    let secret_data = parse_secret_data( cx );  
    set_cookie(cx, "my-secret", &secret_data.0);   // <--new
    leptos_axum::redirect(cx, "/form-action-processed");  
    Ok(())  
}
```

We'll use the following function read our cookie:

```rust
#[cfg(feature = "ssr")]  
fn cookie(cx:Scope, name: &str) -> Option<String> {  
  
    let parts = match use_context::<leptos_axum::RequestParts>(cx){  
        None => return None,  
        Some(parts) => parts  
    };  
  
    let cookies_hv = match parts.headers.get("cookie") {  
        None => return None,  
        Some(cookies_hv) => cookies_hv.as_bytes()  
    };  
  
    let cookies_str = match std::str::from_utf8(cookies_hv) {  
        Ok(cookies_str) => cookies_str,  
        Err(_) => return None  
    };  
  
    let key_val_pairs: Vec<Vec<&str>> = cookies_str  
        .split("; ")  
        .map(|kv| kv.split("=").collect() )  
        .collect();  
  
    for kvp in key_val_pairs.iter() {  
        if kvp.get(0).map(|k|k.deref()) == Some( name ) {  
            return kvp.get(1).map(|k|k.to_string());  
        }  
    }
      
    None  
  
}
```

When we submit the form, we trigger a server function in response. We're then forwarded to a route that displays the FormActionProcessed component. We'll update that component so that it reads our cookie's value and sets it to nothing after to clear it out.

```rust
#[cfg(feature = "ssr")]  
#[component]  
fn FormActionProcessed(cx: Scope) -> impl IntoView {  
    let secret = cookie(cx, "my-secret");  
    set_cookie(cx, "my-secret", "");  
    view!{cx,  
        "Server side response. This should display as a \  
        result of submitting the form. Your secret is: "  {secret}  
    }
}
```

## In Summary

The final `main.rs` looks like this. Keep in mind, this is focusing on Server Side Rendering (SSR)

```rust
// main.rs

use std::ops::Deref;  
use std::str::{FromStr, Utf8Error};  
use leptos::*;  
use leptos_router::*;  
  
#[derive(Debug, Clone)]  
struct SecretData(String);  
  
impl Default for SecretData {  
    fn default() -> Self {  
        Self("".to_string())  
    }
}  

#[component]  
pub fn App(cx: Scope) -> impl IntoView {  
  
    view! {  
        cx,  
        <Router>  
            <Routes>  
                <Route 
	                path="" 
	                view=|cx| view! { cx, <FormPage/> }
				/>  
                <Route 
	                path="/form-action-processed" 
	                view=|cx| view! { cx, <FormActionProcessed /> }
				/>  
            </Routes>  
        </Router>  
    }
}  

  
#[cfg(feature = "ssr")]  
fn set_cookie(cx: Scope, name: &str, value: &str ) {  
    use axum::http::header::{HeaderMap, HeaderValue, SET_COOKIE};  
    use leptos_axum::{ResponseOptions, ResponseParts};  
  
    let response = use_context::<ResponseOptions>(cx)
	    .expect("to have leptos_axum::ResponseOptions provided");  
    let mut response_parts = ResponseParts::default();  
    let mut headers = HeaderMap::new();  
    headers.insert(  
        SET_COOKIE,  
        HeaderValue::from_str(&format!("{name}={value}; Path=/"))  
            .expect("to create header value"),  
    );  
    response_parts.headers = headers;  
    response.overwrite(response_parts);  
}  
  
#[cfg(feature = "ssr")]  
fn cookie(cx:Scope, name: &str) -> Option<String> {  
  
    let parts = match use_context::<leptos_axum::RequestParts>(cx){  
        None => return None,  
        Some(parts) => parts  
    };  
  
    let cookies_hv = match parts.headers.get("cookie") {  
        None => return None,  
        Some(cookies_hv) => cookies_hv.as_bytes()  
    };  
  
    let cookies_str = match std::str::from_utf8(cookies_hv) {  
        Ok(s) => s,  
        Err(_) => return None  
    };  
  
    let key_val_pairs: Vec<Vec<&str>> = cookies_str  
        .split("; ")  
        .map(|kv| kv.split("=").collect() )  
        .collect();  
  
    for kvp in key_val_pairs.iter() {  
        if kvp.get(0).map(|k|k.deref()) == Some( name ) {  
            return kvp.get(1).map(|k|k.to_string());  
        }  
    }  
    
    None  
}  
  
#[server(FormActionHandler)]  
async fn form_action_handler(cx: Scope) -> Result<(), ServerFnError> {  
    let secret_data = parse_secret_data( cx );  
    set_cookie(cx, "my-secret", &secret_data.0);  
    leptos_axum::redirect(cx, "/form-action-processed");  
    Ok(())  
}  
  
#[cfg(feature = "ssr")]  
fn parse_secret_data(cx: Scope) -> SecretData {  
    let parts = match use_context::<leptos_axum::RequestParts>(cx){  
        None => return SecretData::default(),  
        Some(parts) => parts  
    };  
  
    let body = match std::str::from_utf8(&parts.body) {  
        Err(_) => return SecretData::default(),  
        Ok(data) => data  
    };  
  
    let key_val_pairs: Vec<Vec<&str>> = body  
        .split('&')  
        .map(|kv| kv.split('=').collect() )  
        .collect();  
  
    for kvp in key_val_pairs.iter() {  
        match ( kvp.get(0).map(|k|k.deref()), kvp.get(1).map(|k|k.deref()) ) {  
            (Some("secret"), Some(data)) => return SecretData( data.to_string() ),  
            _ => return SecretData::default(),  
        }    
	}  
	
    SecretData::default()  
}  
  
#[component]  
fn FormPage(cx: Scope) -> impl IntoView {  
    view! { cx,  
        <form 
	        action="/api/src-app.rs-form_action_handler" 
	        method="post"
	    > 				 
            <input 
	            type="text" 
		        name="secret" 
		        id="secret" 
		        placeholder="Tell me a secret" 
			/>  
            <input 
	            type="submit" 
	            value="Send request" 
			/>  
        </form>  
    }
}  
  
#[cfg(feature = "ssr")]  
#[component]  
fn FormActionProcessed(cx: Scope) -> impl IntoView {  
    let secret = cookie(cx, "my-secret");  
    set_cookie(cx, "my-secret", "");  
    view!{cx,  
        "Server side response. This should display as a \  
        result of submitting the form. Your secret is: "  {secret}  
    }
}  
  
#[cfg(not(feature = "ssr"))]  
#[component]  
fn FormActionProcessed(cx: Scope) -> impl IntoView {  
    view!{cx, 
	    "Client side response. This should display as a result 
	    of submitting the form on the client."
	}
}
```