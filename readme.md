# SkelForm Developer Documentation

Developer documentation for [SkelForm](https://github.com/Retropaint/SkelForm), made with [mdBook](https://github.com/rust-lang/mdBook).

This is different from the [User Documentation](https://github.com/retropaint/skelform_user_docs).

## Building

[Install mdBook](https://rust-lang.github.io/mdBook/guide/installation.html).

Then, run `mdbook build`.

This will create the distributable `book` dir, which can be published to web (and is required for native SkelForm distributions).

## Formatting

Formatting markdown files is done with [prettier](https://prettier.io/), with the
following relevant config file:

````json
{
  "proseWrap": "always",
  "printWidth": 80
}
````
