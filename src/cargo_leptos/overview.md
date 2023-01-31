# Cargo Leptos Overview

The following is an overview of the example project setup in the [Cargo Leptos Setup](cargo_leptos/setup.md) lesson. The example project looks like a lot. We'll step through the important parts so that you can find you way around. 

## Front end and back end together

One important think to keep in mind is that both front end and back end code exists in the same code base. Config macros are used to include or exclude code from the final binaries or WASM depending on the target. This allows us to take a whole code base and compile only the server code for the server application, while being able to compile only the front end code for the WASM application.

