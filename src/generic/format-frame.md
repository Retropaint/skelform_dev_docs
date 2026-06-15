# `FormatFrame()`

Provides the appropriate frame based on the animation, along with looping and
reverse options.

```typescript
function FormatFrame(
    frame: Int,
    animation: Animation,
    reverse: Bool,
    isLoop: Bool,
): Int {
    lastFrame: Int = animation.keyframes[-1].frame;

    if (isLoop) {
        frame %= lastFrame + 1;
    }

    if (reverse) {
        frame = lastFrame - frame;
    }

    return frame;
}
```
