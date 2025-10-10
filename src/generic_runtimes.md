# Generic Runtimes

Generic runtimes handle [animations](./animating.md) and
[armature construction](./constructing_armature.md).

These runtimes should be engine & render agnostic, with the 'generic' nature
allowing it to be expandable to other environments.

Example: A [generic Rust runtime](https://github.com/Retropaint/rusty_skelform)
can be expanded for Rust game engines like Macroquad or Bevy.
