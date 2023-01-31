# Full stack development with Cargo Leptos

>[Official documentation for Cargo Leptos](https://github.com/leptos-rs/cargo-leptos#features)

## Introduction

Cargo Leptos is a build tool for Leptos. When you setup a project with Cargo Leptos you'll receive a web server which is setup to handle web requests, including responding to certain web requests with your Leptos client side ui. It makes building applications where a front end communicates with a back end (full stack) a piece of cake.

A full set of features can be found in the official documentation. You'll find a lot of the conveniences you maybe have come to expect, or which have become industry standard for UI framework build tools.

## Installing Cargo Leptos

Cargo Leptos is a stand alone application which you will need to install on your computer. You will need to have cargo preinstalled. If you do, enter the following command in your shell/terminal.

```bash
cargo install --locked cargo-leptos
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

### Coming soon

Cargo Leptos has a staged `new` command which will walk you through setting up a project. For now, we'll be using git to clone an existing template as our starting point.

### A template project

The following will clone a repository which is a good starting point for a Leptos application with Axum as its web server. The last line of the bash script will run your web server. To stop the web server press ^C (control + c).

```bash
git clone https://github.com/underscorefunk/cargo-leptos-axum
mv cargo-leptos-axum my-leptos-app
cd my-leptos-app
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