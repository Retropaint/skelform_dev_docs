# `Animate()` - Engine

Simply calls `Animate()` from the generic runtime.

**Parameters**:

- Bones
- Animations
- Frames
- Smooth frames

**Returns**:

- Void

```rust
/// Process bones to be used for animation(s).
pub fn animate(
    bones: &mut Vec<Bone>,
    animations: &Vec<&Animation>,
    frames: &Vec<i32>,
    smooth_frames: &Vec<i32>,
) {
    generic_runtime::animate(bones, animations, frames, smooth_frames);
}
```
