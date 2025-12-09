# `TimeFrame()`

Either calls `TimeFrame()` from the generic runtime, or is re-implemented to
better fit the engine/environment that the runtime is made for.

```c
TimeFrame(time: Time, animation: Animation, reverse: bool, isLoop: bool) {
  GenericRuntime::TimeFrame(time, animation, reverse, isLoop)

  // or, copy it's implementation with a more appropriate Time type from the engine
}
```
