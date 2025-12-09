# `CheckBoneFlip()`

Flips the bone's rotation if either of its scale axes is negative (but not
both).

```c
CheckBoneFlip(bone: *Bone) {
    bool both = bone.scale.x < 0. && bone.scale.y < 0.
    bool either = bone.scale.x < 0. || bone.scale.y < 0.
    if either && !both {
        bone.rot = -bone.rot
    }
}
```
