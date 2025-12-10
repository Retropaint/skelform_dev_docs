# `TimeFrame()`

Provides the appropriate frame based on time given (as duration).

The implementation of `time` is highly dependent on the language and
environment, but any conventional method should do.

If better suited, this function can be re-implemented for engine runtimes.

```c
TimeFrame(
    time: Time, 
    animation: Animation, 
    reverse: bool, 
    isLoop: bool
): int {
    elapsed: float = time.asMillis() / 1e3
    frametime: float = 1. / animation.fps

    frame: int = (elapsed / frametime)
    frame = FormatFrame(frame, animation, reverse, isLoop)

    return frame
}
```
