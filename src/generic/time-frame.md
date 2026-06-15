# `TimeFrame()`

Provides the appropriate frame based on time given (as duration).

The implementation of `time` is highly dependent on the language and
environment, but any conventional method should do.

If better suited, this function can be re-implemented for engine runtimes.

```typescript
function TimeFrame(
  time: Time,
  animation: Animation,
  reverse: Bool,
  isLoop: Bool,
): Int {
  elapsed: Float = time.asMillis() / 1e3;
  frametime: Float = 1.0 / animation.fps;

  frame: Int = elapsed / frametime;
  frame = FormatFrame(frame, animation, reverse, isLoop);

  return frame;
}
```
