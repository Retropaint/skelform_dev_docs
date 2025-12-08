# Engine Runtimes

Engine runtimes handle specific environments such as [loading](./load-engine.md) and
[drawing](./draw-engine.md), and must have a friendly user-facing API.

These runtimes may depend on a generic one to do the heavy lifting, leaving it
to handle features that are best dealt with the engine (eg; rendering).

Example: The
[Macroquad runtime](https://github.com/Retropaint/rusty_skelform_macroquad)
depends on a generic Rust runtime, and takes care of drawing the bones with
Macroquad after animation logic has processed.
