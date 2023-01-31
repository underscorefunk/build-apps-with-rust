# File Structure

The following outlines the folder structure of our Cargo Leptos project and the purposes of each folder to provide an overview of what goes where and why. 

- `/public`: Contains static assets that will be served. Will be copied to path set in the config for `site-root`.
- `/style`: Contains scss files that will be processed and written to the location of two concatenated configs ( `site-root` + `site-pkg-dir` ). Recall that `site-pkg-dir` is relative to the `site-root`.
- `/src:` The rust source of your application.
	- `main.rs`: The file with `main` functions as entry points for the server and client applications. This file configures and starts our server with our Leptos app as a service to handle requests.
		- `#[cfg(feature=ssr)]`: A macro that prefixes code that will only be compiled for the *server side* application binary (the app). 
		- `#[cfg(not(feature=ssr))]`: A macro that prefixes code that will only be compiled for the *client side* application. The client side application and its javascript will be written to the `site-root`/`site-pkg-dir`.
	- `app.rs`: This file is where we setup our main app component which contains routes and specifies what ends up in responses to client requests. From here it's just a matter of creating Leptos components as views for paths and building out your app.
	- `lib.rs`: Cargo Leptos supports hydration. This allows us to serve (via SSR) minimal content to the client. It's usually a skeleton or shell of a ui. Hydration is the act of adding the substance to the shell. We "hydrate" it and bring it to life. The purpose of this is to reduce the initial response time. The hydration function in `lib.rs` is used as WASM on the client to request a hydrated version of a response once it's loaded. Leptos knows when it is or is not in hydration mode, allowing it to serve the shell ui and populated/hydrated ui separately.
	- `error_templates.rs`: Contains Leptos Views that will serve as responses for axum errors like 404 for missing endpoints/urls.
	- `fileserv.rs`: Serves static files if ssr is enabled
- `/end2end:` Contains end to end tests and `playright` config 
