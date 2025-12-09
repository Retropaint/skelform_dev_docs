# How To Read This

This is a developer-focused document for developing SkelForm runtimes.

This document is written under the assumption that a general-use runtime will be
made and accounts for all features and cases, but need not be followed this way
for personal runtimes.

## Runtime APIs

Both generic and runtime sections are based on their public-facing API
functions.

The functions _within_ their sections need not be implemented, and simply exist
to split their respective algorithms to be more easily digestible.

### Example

All general-use generic runtimes must have an `Animate()` function that works as
intended (interpolates bones & resets them if needed).

The `Animate()` function's implementation is covered in its own section, and the
functions within do not need to be public nor even implemented. All that matters
is `Animate()` working as intended.

## Pseudo Code

All code shown on this document is not meant to be run directly.

The base language is C, but 'universally' bastardized with the following:

- No end-of-line semicolons
- Variable type follows name (`varName: Type` instead of `Type varName`)
- Arrays are **passed by value** unless followed by a `*` (PBR: `value: int[]`,
  PBV: `value: int[]*`)
- `void` return type is implicit
- Tuples occasionally used
