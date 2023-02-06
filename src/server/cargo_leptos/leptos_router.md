# Leptos Router

>[Official documentation for `leptos_router`](https://docs.rs/leptos_router/latest/leptos_router/)

## Location as state

Our web apps start in an initial state. They load and present us with a default user interface. In the context of these lessons, that default interface would come from the `view!` macro result returned by our App component. Recall that the App component is passed into our axum serve integration and serves the same purpose for Leptos as  `fn main()` does for a rust binary (application). 

The way we change the state of our application is by interacting with it. Intuitively this might seem obvious. We move our mouse, click on something press some keys, and if our application has deemed those events as "state changing interactions" then our state would update accordingly. What might seem less intuitive is thinking of navigating to a different page as a state change as well.

For example clicking on a link to go to an about page changes the state of the application from "viewing the default" to "viewing about". It's almost as if the context of your future interactions have changed. It can be very tempting to think about paths like `/about` as a location or file. I encourage you to think about it as a location in a state graph. Your application is running the same main program to handle all requests. `/about` is not a different program. It is your program in a `/about` state.

This is a very powerful mental model that is seldom discussed, but which will serve you well; **The user interface that you are seeing is a function of your application with events applied.** 

In Leptos you can think of routes as conditional statements that act over the current uri (location that you are at in the website with query variables). My hope is that this mental model will serve you well as you use paths/locations as routes to slice up states of your application. If done well, your application will have discreet states with well established guards between those states, which will make your application ui development an absolute joy to develop.

## Router

`leptos_router` is the external Leptos module that facilitates switching over which components are displayed to the user based on the location of a web request. This location may come from client side routing or server side routing thanks to the routers flexibility. 

- **Client side routing:** A user clicks on a link and the browser's history receives a new entry for the new page. The WASM module will render a new UI for the give location, but this ui is generated purely on the client. A new page is not requested from the server because the client is able to route requests or conditionally show content based on the active history. interesting the browser's javascript call to add a new location to its history is called `pushState`. Like I said, location is state! 

- **Server side routing:**  A user clicks on a link with a specific path. The path is added to the history and a request is made to the server. Information about the request like the requester's IP, browser, cookies set for the domain of the application, and query variables (variables specified after the `?` character in the request path) as sent with the request. The server looks at the request and make a judgement about what to return given the state of the application set by that input. *The response is a function of the request!* Additionally, a user may click submit on a form, sending form data as part of the other request info to the form's action url. It's like clicking on a link, but adding extra data along with it. The action url, like a web link, may also have query variables. If the form's submit method is "GET", all values will be appended to the action url as query variables. If the submit method is "POST" the values will be sent as part of the web request as the aforementioned form data, leaving the action url untouched.

## Setting up routes

Leptos routes are setup by creating one `<Router>...</Router>`. Within the router is a `<Routes>` container which may have one or more `<Route>` components. And again, all of these are places in the top level components `view!` macro template.

Each `<Route>` component has the following properties:

- `path`: A string that identifies the location. Tokens can be created to extract values from the location by prefixing the name of the token with a colon `:`.
- `view`: A closure that returns a value that implements IntoView, like the result of a `view!` macro. The value of the view property must be a closure so that it can be executed as required.

```rust
<Router>
	<Routes>
		<Route path="/" view=move |cx| {
		  view! { cx, "This is the home page" }
		}/>
		<Route path="/about" view=move |cx| {
		  view! { cx, "This is the about page" }
		}/>
		<Route path="/articles/:id" view=move |cx| {
		  view! { cx, "This is a view for an article with 
				the id extracted from whatever is after 
				`/articles/` in the uri" 
		  }
		}/>
	</Routes>
</Router>
```

## Route components

A single route can be extracted and turned into a route component by using the modified component macro `#[component(transparent)]`

```rust
use leptos::*;
use leptos_router::*;

#[component]
pub fn App(cx: Scope) -> impl IntoView {
	view! { cx,
		<Router>
			<Routes>
				<Route path="/" view=move |cx| {
				  view! { cx, "This is the home page" }
				}/>
				<Route path="/about" view=move |cx| {
				  view! { cx, "This is the about page" }
				}/>
				<ArticleRoutes />
			</Routes>
		</Router>
	}
}

#[component(transparent)]
pub fn ArticleRoutes(cx: Scope) -> impl IntoView {
	view! { cx,
		<Route path="/articles" view=move |cx| {
		  view! { cx, "A list of available articles"}
		}/>
	}
}
```

## Carrying route views forward with nested routes

Let's imagine that you have an online magazine. Each magazine is an issue with a cool header graphic and links to the articles in that issue. This header carries across all of the articles. We'd like it to stay consistent instead of re-rendering every time. Nested routes allow us to do just that. A nested route will match the next part of a uri. The matching route's view will be displayed in the parent route's `<Outlet />` tag.

Here's an example of the structure.

```rust
view! {  
    cx,  
    <Router>  
        <Routes>  
            <Route path="" view=move |cx| view!{cx, "App index page"} />  
  
            <Route path="/a-parent-route" view=move |cx| view!{cx,  
                <h1>"A parent route"</h1>
                <p>  
                    "A heading for the child routes.  
                    The active route will replace the   
                    Outlet tag"  
                </p>  
                <fieldset>  
				    <legend>"Outlet"</legend>  
					
					<Outlet/>  
					
				</fieldset>  
            } >  
            
                <Route path="/" view=move |cx| view!{cx,  
                    <p>"The parent page's default Outlet content"</p>
				}/> 
				 
                <Route path="/sub-route" view=move |cx| view!{cx,  
                    <p>"The parent page's Outlet content will 
	                 be populated by this if we navigate to 
	                 /a-parent-route/sub-route"</p>
                } />  
  
            </Route>  
        </Routes>  
    </Router>  
  
}
```


## Navigating to a state/route

Simple links will work for navigating throughout your site/app as you would expect.

```rust
view! { cx,
	<a href="/about">About</a>
}
```

Forced navigation can be achieved by calling the navigate function.

```rust
let navigate = use_navigate(cx);

view! { cx,
	<button on:click=move |_| { _ = navigate("/about", Default::default()); }>
}
```

## Accessing state/route params

A view can extract values of parameters by using the view's supplied Scope `cx` and the `use_params_map` function. The function returns a `memo<ParamsMap>`. Memo is short for memoize. It's a type of cache value that allows us to access the data without computing it over each time. A `ParamsMap` as the method `get(key: &str)` which returns an optional value. We want to clone it so that we can safely use it going forward, peeling off the reference. We then unwrap the result of cloned, because cloned() returns an option type. When we write `.unwrap_or_default()` we're writing , "Use the value of thing from Some(thing), or whatever the default method returns for the type that we're using." At this spoint we want to parse whatever that was into a string.

```rust
<Route path="/:some_token_name" view=move |cx| {  
	let params = use_params_map(cx);  
	let uri_param: String = params()  
		.get("some_token_name")  
		.cloned()  
		.unwrap_or_default();

	view! { cx, {uri_param} }  
}  
/>
```
>String as a uri_param type is not necessary, but I added it so that you can see which type uri_param is.

You may have specific data types that you want to use which you can parse from the string value provided as the param.