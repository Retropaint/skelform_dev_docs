# `Animate()` - Generic

Interpolates bone fields based on provided animation data, as well as initial
states for non-animated fields.

```rust
pub fn animate(
    bones: &mut Bone[], anims: Animation[], frames: int[], smooth_frames: int[]
) {
    for a in range(anims) {
        for bone in bones {
            interpolate_bone(
                &mut bone, anims[a].keyframes, bone.id, frames[a], smooth_frames[a]
            )
        }
    }

    for bone in bones {
        reset_bone(bone, frames[0], smooth_frames[0], anims)
    }
}
```

## `InterpolateBone()`

Interpolates one bone's fields based on provided animation data.

```rust
fn interpolate_bone(
    bone: &mut Bone,
    keyframes: &Vec<Keyframe>,
    bone_id: int,
    frame: int,
    blend_frame: int,
) {
    interpolate_keyframes("PositionX", &mut bone.pos.x,   ...)
    interpolate_keyframes("PositionY", &mut bone.pos.y,   ...)
    interpolate_keyframes("Rotation",  &mut bone.rot,     ...)
    interpolate_keyframes("ScaleX",    &mut bone.scale.x, ...)
    interpolate_keyframes("ScaleX",    &mut bone.scale.y, ...)

    bone.tex = get_prev_frame("Texture" ...)
    bone.ik_constraint = get_prev_frame("IkConstraint", ...)
}
```

## `ResetBone()`

Interpolates one bone's fields to their initial values if not being animated.

```rust
pub fn reset_bone(bone: &mut Bone, frame: int, smooth_frame: int, anims: Animation[]) {
    if !is_animated("PositionX", ...)
        interpolate(bone.pos.x, bone.init_pos.x, ...)

    if !is_animated("PositionY", ...)
        interpolate(bone.pos.y, bone.init_pos.y, ...)

    if !is_animated("Rotation", ...)
        interpolate(bone.rot, bone.init_rot, ...)

    if !is_animated("ScaleX", ...)
        interpolate(bone.scale.x, bone.init_scale.x, ...)

    if !is_animated("ScaleY", ...)
        interpolate(bone.scale.y, bone.init_scale.y, ...)

    // non-interpolated fields are set immediately
    if !is_animated("IkConstraint", ...)
        bone.ik_constraint = bone.init_ik_constraint
}
```

## `InterpolateKeyframes()`

With the provided animation frame, determines the keyframes to interpolate the
field by.

The resulting interpolation from the keyframes is interpolated again for
smoothing.

```rust
fn interpolate_keyframes(
    element: Enum,
    field: &mut float,
    keyframes: Keyframe[],
    id: int,
    frame: int,
    smooth_frame: int,
) {
    let prev = get_prev_frame(...)
    let next = get_next_frame(...)

    // ensure both frames are pointing somewhere
    if prev == None {
        prev = next
    } else if next == None {
        next = prev
    }

    // if both are none, then the frame doesn't exist. Do nothing
    if prev == None && next == None {
        return
    }

    let total_frames = keyframes[next].frame - keyframes[prev].frame
    let current_frame = frame - keyframes[prev].frame

    let result = interpolate(
        current_frame, total_frames, keyframes[prev].value, keyframes[next].value
    )

    // result is smoothed
    *field = interpolate(current_frame, smooth_frame, *field, result)
}
```

## `IsAnimated()`

Returns true if a particular element is part of the provided animations.

```rust
fn is_animated(bone_id: int, element: int, animations: Animation[]) -> bool {
    for anim in anims {
        for kf in anim.keyframes {
            if kf.bone_id == bone_id && kf.element == element {
                return true
            }
        }
    }
    return false
}
```

## `Interpolate()`

Generic linear interpolation.

```rust
fn interpolate(current: int, max: int, start_val: float, end_val: float) -> float {
    if max == 0 || current >= max {
        return end_val
    }
    let interp = current as float / max as float
    let end = end_val - start_val
    let result = start_val + (end * interp)

    return result
}
```
