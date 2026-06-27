# `IsFacingLeft()`

Returns true if either X or Y of the provided scale is negative (but not both).

Used in [engine Construct()](../engine/construct.html) and
[Draw()](../engine/draw.html) to flip bones or pivots if necessary.

```typescript
function IsFacingLeft(scale: Vec2): Bool {
    const both: Bool = scale.x < 0 && scale.y < 0;
    const either: Bool = scale.x < 0 || scale.y < 0;
    return either && !both;
}
```
