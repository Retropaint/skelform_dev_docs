# Runtimes

## Generic Runtimes

Generic runtimes mostly handle [animations](./animating.md) and
[armature construction](./constructing_armature.md).

Other features such as loading `.skf` files or special effects must be made
optional if they require dependencies.

These runtimes should be engine & render agnostic, and the 'generic' nature
allowing it to be expandable to other environments.

Example: A [generic Rust runtime](https://github.com/Retropaint/rusty_skelform)
can be expanded for Rust game engines like Macroquad or Bevy.

## Engine Runtimes

Engine runtimes handle specific environments and must have a user-friendly API.

These runtimes may depend on a generic one to do the heavy lifting, leaving it
to handle features that are best dealt with the engine (eg; loading and/or
[rendering](./rendering.md)).

Example: The
[Macroquad runtime](https://github.com/Retropaint/rusty_skelform_macroquad)
depends on a generic Rust runtime, and takes care of drawing the bones with
Macroquad after animation logic has processed.
