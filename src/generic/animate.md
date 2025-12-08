# `Animate()` - Generic

Interpolates bone fields based on provided animation data, as well as initial
states for non-animated fields.

```c
void Animate(Bone *bones[], Animation anims[], int frames[], int smoothFrames[]) {
    for a in range(anims) {
        for bone in bones {
            interpolateBone(
                &bone, anims[a].keyframes, bone.id, frames[a], smooth_frames[a]
            )
        }
    }

    for bone in bones {
        resetBone(bone, ...)
    }
}
```

## `interpolateBone()`

Interpolates one bone's fields based on provided animation data.

```c
void interpolateBone(
    Bone *bone,
    Keyframe keyframes[],
    int bone_id,
    int frame,
    int blend_frame,
) {
    interpolate_keyframes("PositionX", &bone.pos.x,   ...)
    interpolate_keyframes("PositionY", &bone.pos.y,   ...)
    interpolate_keyframes("Rotation",  &bone.rot,     ...)
    interpolate_keyframes("ScaleX",    &bone.scale.x, ...)
    interpolate_keyframes("ScaleX",    &bone.scale.y, ...)

    bone.tex = get_prev_frame("Texture" ...)
    bone.ik_constraint = get_prev_frame("IkConstraint", ...)
}
```

## `resetBone()`

Interpolates one bone's fields to their initial values if not being animated.

```c
void resetBone(Bone *bone, int frame, int smooth_frame, Animation anims[]) {
    if !isAnimated("PositionX", ...)
        interpolate(bone.pos.x, bone.init_pos.x, ...)

    if !isAnimated("PositionY", ...)
        interpolate(bone.pos.y, bone.init_pos.y, ...)

    if !isAnimated("Rotation", ...)
        interpolate(bone.rot, bone.init_rot, ...)

    if !isAnimated("ScaleX", ...)
        interpolate(bone.scale.x, bone.init_scale.x, ...)

    if !isAnimated("ScaleY", ...)
        interpolate(bone.scale.y, bone.init_scale.y, ...)

    // non-interpolated fields are set immediately
    if !isAnimated("IkConstraint", ...)
        bone.ik_constraint = bone.init_ik_constraint
}
```

## `interpolateKeyframes()`

With the provided animation frame, determines the keyframes to interpolate the
field by.

The resulting interpolation from the keyframes is interpolated again for
smoothing.

```c
void interpolateKeyframes(
    Enum element,
    float *field,
    Keyframe keyframes[],
    int id,
    int frame,
    int smooth_frame,
) {
    int prev = getPrevFrame(...)
    int next = getNextFrame(...)

    // ensure both frames are pointing somewhere
    if prev == -1 {
        prev = next
    } else if next == -1 {
        next = prev
    }

    // if both are -1, then the frame doesn't exist. Do nothing
    if prev == -1 && next == -1
        return

    int total_frames = keyframes[next].frame - keyframes[prev].frame
    int current_frame = frame - keyframes[prev].frame

    float result = interpolate(
        current_frame, total_frames, keyframes[prev].value, keyframes[next].value
    )

    // result is smoothed
    *field = interpolate(current_frame, smooth_frame, *field, result)
}
```

## `isAnimated()`

Returns true if a particular element is part of the provided animations.

```c
bool isAnimated(int bone_id, Enum element, Animation animations[]) {
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

## `interpolate()`

Generic linear interpolation.

```c
float interpolate(int current, int max, float start_val, float end_val) {
    if max == 0 || current >= max {
        return end_val
    }
    float interp = current as float / max as float
    float end = end_val - start_val
    float result = start_val + (end * interp)

    return result
}
```
