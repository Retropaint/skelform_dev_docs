# `Animate()` - Engine

Simply calls `Animate()` from the generic runtime.

```typescript
function animate(
    armature: Armature,
    animations: Animation[],
    frames: Int[],
    smoothFrames: Int[],
) {
    GenericRuntime.animate(armature, animations, frames, smoothFrames);
}
```
