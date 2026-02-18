# `CheckBoneFlip()`

Flips the bone's rotation if either of the provided scale axes is negative (but
not both).

This is the standard method of 'flipping' sprites, hence it uses an arbitrary
scale rather than the bone's own.

```typescript
function CheckBoneFlip(bone: Bone, scale: Vec2) {
    bool both = scale.x < 0. && scale.y < 0.
    bool either = scale.x < 0. || scale.y < 0.
    if(either && !both) {
        bone.rot = -bone.rot
    }
}
```
