# Full stack development with Cargo Leptos

>[Official documentation for Cargo Leptos](https://github.com/leptos-rs/cargo-leptos#features)

## Introduction

Cargo Leptos is a build tool for Leptos. When you setup a project with Cargo Leptos you'll receive a web server which is setup to handle web requests, including responding to certain web requests with your Leptos client side ui. It makes building applications where a front end communicates with a back end (full stack) a piece of cake.

A full set of features can be found in the official documentation. You'll find a lot of the conveniences you maybe have come to expect, or which have become industry standard for UI framework build tools.

## Installing Cargo Leptos

Cargo Leptos is a stand alone application which you will need to install on your computer. You will need to have cargo preinstalled. If you do, enter the following command in your shell/terminal.

```bash
cargo install cargo-leptos
```

You can confirm the installation by checking the active version.

```shell
cargo leptos -V
```

## Dependencies

You may need to us nightly rust. You can set the default with:

```bash
rustup default nightly
```

You may be required to install the rust-up wasm target:

```bash
rustup target add wasm32-unknown-unknown
```

You should switch to nightly before installing the target wasm target,

## Setting up a new application

You can initialize a new axum project with:

```bash
cargo leptos new --git https://github.com/leptos-rs/start-axum
```

You'll be asked to enter a project name.

You can then change directory `cd` into the project folder and run yur app.

```bash
cd my-project-name
cargo leptos watch
```

The above bash script should print something like the following to your terminal:

```bash
 Finished dev [unoptimized + debuginfo] target(s) in 0.62s
       Cargo finished cargo build --package=start-axum --lib --target-dir=target/front --target=wasm32-unknown-unknown --no-default-features --features=hydrate
    Finished dev [unoptimized + debuginfo] target(s) in 0.67s
       Cargo finished cargo build --package=start-axum --bin=start-axum --target-dir=target/server --no-default-features --features=ssr
      Notify watching folders public, style, src
listening on http://127.0.0.1:3000
```

This means that we can now visit the url 127.0.0.1:3000, and see our placeholder Leptos app. Leptos will use port 3001 by default to watch for updates. 

## Adding HTTPS for local development

Axum, inside Cargo Leptos' setup, will attempt to stream content from the server to the client, especially in situations where suspense components are used. Streaming is only supported with HTTP/2, which is only available through TLS (via https).

We can use a program called Caddy to create a reverse proxy. This will allow you to visit a site on your computer at `https://leptos.localhost`, and have requests/responses forwarded through that boundary to the Leptos server. 

Setting up a reverse proxy is pretty easy.

Step 1 is to install caddy from https://caddyserver.com or from `brew install caddy` if you are on OS X and have homebrew installed

Step 2 is to create a `Caddyfile` in your project folder. The file has no extension.

```text
leptos.localhost { 
	reverse_proxy 127.0.0.1:3000 
}
```

Step 3 is to start the caddy server:

```bash
caddy run
```
