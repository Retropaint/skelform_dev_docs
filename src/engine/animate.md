# `Animate()` - Engine

Simply calls `Animate()` from the generic runtime.

**Parameters**:

- Bones
- Animations
- Frames
- Smooth frames

**Returns**:

- Void

```c
Animate(bones: Bone[]*, animations: Animation[], frames: int[], smoothFrames: int[]) {
    GenericRuntime::Animate(bones, animations, frames, smoothFrames)
}
```
