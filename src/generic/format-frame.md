# `FormatFrame()`

Provides the appropriate frame based on the animation, along with looping and
reverse options.

```c
FormatFrame(
    frame: int, 
    animation: Animation, 
    reverse: bool, 
    isLoop: bool
): int {
    lastFrame: int = animation.keyframes.last().frame

    if isLoop {
        frame %= lastFrame + 1
    }

    if reverse {
        frame = lastFrame - frame
    }

    return frame
}
```
