# Get Method Forms (Form to Query String)

## Submitting requests to a server (via url)

The internet works on a basis of request/response. We ask for something, and a server responds (or doesn't, which is still a response of sorts). When we enter a URL in our browsers, a request for that resource is sent to the server handling requests for the domain. This is an actual computer or network endpoint somewhere on the web. The DNS system looks up the address of that computer and sends the request on down to that actual address. If all goes well, the server will respond with a new page, static resource, etc—what you asked for. 

We can configure those requests by adding query string variables after the url, separated by a question mark (`?`). Each parameter and argument pair—often called a key/value pair—in the query string is separated by an ampersand (`&`). Query strings can not have spaces or certain special characters. You can imagine how problematic it would be if the value of one of the query string parameters had an ampersand in it. It's customary to encode those special characters for use as query string values/arguments. You'll have seen this all over the web. If you see %20, that is the encoded value of a space. Interestingly, they're called query strings because it is adding specificity to our resource (response) query (what we're asking for).

For example:
`https://some-non-existant-store.com/catalog?page=1&per_page=12&title=Cool%20Products`

This is the most common way that we make requests online. Adding a url in an image source, entering a url into your web browser, linking a css file, they all use this same approach. Take the request and respond with a resource.


## Submitting requests to a server (via forms)

>**Important:** The following uses `<form>` the HTML Tag. This should not be confused with `<Form>` the Leptos component.

### Get method

It is possible to submit requests to servers while allowing user input. Forms are the foundational web tool for doing this. We do this by authoring a form with `<form> ... </form>` tags, and setting the action property on it to the url that will process the request. It's where the form will be sent. One of the methods forms can use is called `get` which will embed the form's fields to the action as a query string. You can think of the form field ids as the parameters, with their values as the arguments. 

For example:
```html
<form 
	  action="https://some-non-existant-store.com/catalog" 
	  method="get"
>
	<input type="number" name="page" value="1" />
	<input type="number" name="per_page" value="12" />
	<input type="text" name="title" value="Cool Products" />
	
	<inpt type="submit" value="Submit request">
</form>
```
> Clicking the submit button would send the input field values a query string appended to the action url.

The benefits of using `get`:
- URLs are visible
- URLs are stored in your history with the query string allowing for navigation to and from the submitted form's response

Drawbacks of using `get`:
- URLs are visible and make data that is submitted public
- Data is stored in the history and can be introspected
- URLs have a maximum length, limiting the amount of data that can be sent

### Post method

Changing the method of our form to `post` will take us to the same url as written in the action property's value. However, the post method will collect the data and submit it as a payload _with_ the url to the server. `Post` does not append the form's values to the request as a query string.

Benefits of using `post`:
- Data is hidden from the history
- Data doesn't pollute the url
- Complex and multipart data can be sent

Drawbacks of using `post`:
- When pressing reload, it's possible to resubmit the form

## Why forms are an important part of the web platform

Direct resource requests and form submissions are the two main ways that we can use the web platform to submit requests and retrieve new data from servers. An important thing to note is that forms work out of the box. We do not need to iterate over fields, extract values, and then submit some javascript event. They just work, even with Javascript disabled! I encourage you to get comfortable with forms and what we're about to learn here. Forms, though old technology, will make your applications more accessible, robust, and depending on how you load your application, faster to load the initial useful render.

## Starting with a simple form

We'll build out a new Leptos application using cargo leptos. See [Cargo Leptos Setup](/server/cargo_leptos/setup,md) for details.

We'll modify our `src/app.rs` file to scaffold out two routes. One that holds our form and the other that holds the page loaded in response.

```rust
use leptos::*;  
use leptos_router::*;  
  
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
	                path="/form-action-page" 
	                view=|cx| view! { cx, <FormActionPage /> }
				/>  
            </Routes>  
        </Router>  
    }}  
  
#[component]  
fn FormPage(cx: Scope) -> impl IntoView {  
    view! { cx,  
        <form action="/form-action-page">  
            <input type="submit" value="Send request" />  
        </form>  
    }
}  
  
#[component]  
fn FormActionPage(cx: Scope) -> impl IntoView {  
    view! { cx,  
        <h1>"You submitted a form and we ended up here!"</h1>  
    }
}
```

If you run this this with `cargo leptos watch` and click on the submit button you'll be taken to the `/form-action-page`. Note that it's listed as the form's action. This is our simple proof of how forms traditionally work. Note that if no method is set, `method=post` will be assumed.

Let's update the form page to have some data.

```rust
#[component]  
fn FormPage(cx: Scope) -> impl IntoView {  
    view! { cx,  
        <form action="/form-action-page">  
            <input type="text" name="secret" placeholder="Tell me a secret" />  
            <input type="submit" value="Send request" />  
        </form>  
    }
}
```


### Get method

When a route gets processed, the route selected becomes part of the context. There are a set of helper functions that facilitate grabbing parts of the request for us to work with. As the query string is part of the request, we can look to `leptos_router::use_query_map` as a tool to extract query values.

We'll start by using use_query_map to get a memoized query map from the context. Memoization can be thought of as a cache that says, if we've retrieved the value once, just serve that value again instead of deriving/calculating it all over again.

```rust
fn FormActionPage(cx: Scope) -> impl IntoView {  
    let qm : Memo<ParamsMap> = use_query_map(cx);
    // ...
}
```

We need to get the value from the memoizing container.

```rust
let pm : ParamsMap = use_query_map(cx)
    .get();
```

But now we have a `ParamsMap` which we can try to get our query value from by providing it with the key.

```rust
let maybe_secret : Option<&String> = use_query_map(cx)
    .get()
    .get("secret");
```

And then of course we need to unwrap the string or provide a default if it doesn't exist.

```rust
let maybe_secret : Option<&String> = use_query_map(cx)
    .get()
    .get("secret");
	.unwrap_or("No secret was provided");
```

The above will not work though. The value that we've provided for the unwrap_or is a string slice `&str`. The return type of getting the `secret` is a `&String`. We can't have two potential types, a `&String` or `&str`.

A solution to this is to convert the fallback message into a string with `.to_string()`

```rust
let maybe_secret : Option<&String> = use_query_map(cx)
    .get()
    .get("secret");
	.unwrap_or("No secret was provided".to_string());
```

But we still have a problem with it the fallback being an owned value, not a reference. We can add an ampersand (`&`) to make it a reference.

```rust
let maybe_secret : Option<&String> = use_query_map(cx)
    .get()
    .get("secret");
	.unwrap_or(&"No secret was provided".to_string());
```

We're close, but Rust's compiler will remind us that we're doing something a bit silly here. We're evaluating this expression to create a string and then telling unwrap to use a reference to that string  as the fallback. The original string that the reference is for, doesn't actually stick around though. So what we need to do is move the fallback outside of this, and provide a reference to it, so that the value lives long enough.

Params map also needs to be declared separately because it gives up a reference to some of its data as well, through the `Option<&String>`. Rust is telling us, "Hey, you can't give a reference to a thing that we clean up right away."

```rust
use leptos::*;  
use leptos_meta::*;  
use leptos_router::*;  
  
#[component]  
pub fn App(cx: Scope) -> impl IntoView {  
    view! {  
        cx,  
        <Router>  
            <Routes>  
                <Route path="" view=|cx| view! { cx, <FormPage/> }/>  
                <Route path="/form-action-page" view=|cx| view! { cx, <FormActionPage /> }/>  
            </Routes>  
        </Router>  
    }}  
  
#[component]  
fn FormPage(cx: Scope) -> impl IntoView {  
    view! { cx,  
        <form action="/form-action-page" method="get">  
            <input type="text" name="secret" placeholder="Tell me a secret" />  
            <input type="submit" value="Send request" />  
        </form>  
    }}  
  
#[component]  
fn FormActionPage(cx: Scope) -> impl IntoView {  
    let fallback = "No secret was provided".to_string();  
    let qm = use_query_map(cx);  
    let pm = qm.get();  
    let the_secret = pm.get("secret")  
        .unwrap_or(&fallback);  
  
    view! { cx,  
        <h1>"You submitted a form and we ended up here!"</h1>  
        <p>{the_secret}</p>  
    }}
```

But as you can see, there's a problem here. Our secret is out in the open! Anyone look at the history will see! Now the whole world will know how much I love enchiladas. 

Take a look at post_method_forms to help these secrets! 

Keep them secret, keep them safe.