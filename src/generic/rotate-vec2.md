# `RotateVec2()`

Helper for rotating a Vec2.

Used in [generic Construct()](../generic/construct.html) and
[Draw()](../engine/draw.html) to rotate bones and pivots.

```typescript
function RotateVec2(point: Vec2, rot: float): Vec2 {
    return Vec2 {
        x: point.x * rot.cos() - point.y * rot.sin(),
        y: point.x * rot.sin() + point.y * rot.cos(),
    }
}
```
