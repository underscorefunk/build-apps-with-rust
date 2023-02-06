# Configuring Cargo Leptos

## Cargo.toml

Configuration for Cargo Leptos can be done in the cargo.toml. We do this by adding package metadata to leptos.

```toml
[package.metadata.leptos]

# The name used by wasm-bindgen/cargo-leptos for the JS/WASM bundle. Defaults to the crate name

output-name = "start-axum"

# The site root folder is where cargo-leptos generate all output. WARNING: all content of this folder will be erased on a rebuild. Use it in your server setup.

site-root = "target/site"

# The site-root relative folder where all compiled output (JS, WASM and CSS) is written

# Defaults to pkg

site-pkg-dir = "pkg"

# [Optional] The source CSS file. If it ends with .sass or .scss then it will be compiled by dart-sass into CSS. The CSS is optimized by Lightning CSS before being written to <site-root>/<site-pkg>/app.css

style-file = "style/main.scss"

# Assets source dir. All files found here will be copied and synchronized to site-root.

# The assets-dir cannot have a sub directory with the same name/path as site-pkg-dir.

#

# Optional. Env: LEPTOS_ASSETS_DIR.

assets-dir = "public"

# The IP and port (ex: 127.0.0.1:3000) where the server serves the content. Use it in your server setup.

site-address = "127.0.0.1:3000"

# The port to use for automatic reload monitoring. Make sure this port is not the same as the port used in site-address.

reload-port = 3001

# [Optional] Command to use when running end2end tests. It will run in the end2end dir.

# [Windows] for non-WSL use "npx.cmd playwright test"

# This binary name can be checked in Powershell with Get-Command npx

end2end-cmd = "npx playwright test"

end2end-dir = "end2end"

# The browserlist query used for optimizing the CSS.

browserquery = "defaults"

# Set by cargo-leptos watch when building with that tool. Controls whether autoreload JS will be included in the head

watch = false

# The environment Leptos will run in, usually either "DEV" or "PROD"

env = "DEV"

# The features to use when compiling the bin target

#

# Optional. Can be over-ridden with the command line parameter --bin-features

bin-features = ["ssr"]

# If the --no-default-features flag should be used when compiling the bin target

#

# Optional. Defaults to false.

bin-default-features = false

# The features to use when compiling the lib target

#

# Optional. Can be over-ridden with the command line parameter --lib-features

lib-features = ["hydrate"]

# If the --no-default-features flag should be used when compiling the lib target

#

# Optional. Defaults to false.

lib-default-features = false
```

### Error: Address already in use

Interrupting Cargo Leptos may result in the port being bound after Cargo Leptos terminates. Using `cargo leptos watch` again will yield an error message stating:

```
thread 'main' panicked at 'error binding to 127.0.0.1:3000: error creating server listener: Address already in use (os error 48)', 
```

You will receive the same error if your `site-address` is using the same port as the `reload-port`.

