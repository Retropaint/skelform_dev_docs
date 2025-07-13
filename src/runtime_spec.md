# Runtimes

## Table of Contents

- [Generic Runtimes](#generic-runtimes)
- [Engine Runtimes](#engine-runtimes)

## Generic Runtimes

Generic runtimes mostly handle [core animation logic](./core_anim_logic.md).

Other features such as loading `.skf` files or special effects must be made
optional if they require dependencies.

These runtimes should be engine & render agnostic, and the 'generic' nature
allowing it to be expandable to other environments.

Example: A [generic Rust runtime](https://github.com/Retropaint/rusty_skelform)
can be expanded for Rust game engines like Macroquad or Bevy.

## Engine Runtimes

Engine runtimes handle specific environments and must have a user-friendly API.

Because of the user-friendly nature, features such as loading `.skf` files can
be made mandatory. It is still preferrable to be optional to allow user
customization.

Ideally, engine runtimes should simply depend on a generic runtime to do the
heavy lifting, and leave itself to only handle rendering and engine-specific
quirks.

Example: The
[Macroquad runtime](https://github.com/Retropaint/rusty_skelform_macroquad)
depends on a generic Rust runtime, and takes care of drawing the bones with
Macroquad after animation logic has processed.
