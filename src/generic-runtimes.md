# Generic Runtimes

Generic runtimes handle [animations](./animate-generic.md) and
[armature construction](./construct-generic.md).

These runtimes should be engine & render agnostic, with the 'generic' nature
allowing it to be expandable to other environments.

Example: A [generic Rust runtime](https://github.com/Retropaint/rusty_skelform)
can be expanded for Rust game engines like Macroquad or Bevy.
