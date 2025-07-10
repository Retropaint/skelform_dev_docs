# SkelForm User Documentation

User documentation for [SkelForm](https://github.com/Retropaint/SkelForm), made with [mdBook](https://github.com/rust-lang/mdBook).

## Building

[Install mdBook](https://rust-lang.github.io/mdBook/guide/installation.html).

Then, run `mdbook build`.

This will create the distributable `book` dir, which can be published to web (and is required for native SkelForm distributions).

## Formatting

Formatting markdown files is done with [dprint](https://dprint.dev/), with the
following relevant config file:

````json
{
    "markdown": {
        "textWrap": "always"
    },
    "plugins": [
        "https://plugins.dprint.dev/markdown-0.18.0.wasm",
    ]
}
````
