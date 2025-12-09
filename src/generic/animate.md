# `Animate()` - Generic

Interpolates bone fields based on provided animation data, as well as initial
states for non-animated fields.

```c
Animate(
    bones: *Bone[],
    anims: Animation[],
    frames: int[],
    smoothFrames: int[]
) {
    // interpolate bone fields
    for a in range(anims) {
        for bone in bones {
            interpolateBone(
                &bone, anims[a].keyframes, bone.id, frames[a], smooth_frames[a]
            )
        }
    }

    // interpolate bone resets
    for bone in bones {
        resetBone(bone, ...)
    }
}
```

## `interpolateBone()`

Interpolates one bone's fields based on provided animation data.

```c
interpolateBone(
    bone: *Bone,
    keyframes: Keyframe[],
    boneId: int,
    frame: int,
    blendFrame: int,
) {
    interpolateKeyframes("PositionX", &bone.pos.x,   ...)
    interpolateKeyframes("PositionY", &bone.pos.y,   ...)
    interpolateKeyframes("Rotation",  &bone.rot,     ...)
    interpolateKeyframes("ScaleX",    &bone.scale.x, ...)
    interpolateKeyframes("ScaleX",    &bone.scale.y, ...)

    bone.tex = getPrevFrame("Texture" ...)
    bone.ikConstraint = getPrevFrame("IkConstraint", ...)
}
```

## `resetBone()`

Interpolates one bone's fields to their initial values if not being animated.

```c
resetBone(bone: Bone*, frame: int, smoothFrame: int, anims: Animation[]) {
    if !isAnimated("PositionX", ...)
        interpolate(bone.pos.x, bone.initPos.x, ...)

    if !isAnimated("PositionY", ...)
        interpolate(bone.pos.y, bone.initPos.y, ...)

    if !isAnimated("Rotation", ...)
        interpolate(bone.rot, bone.initRot, ...)

    if !isAnimated("ScaleX", ...)
        interpolate(bone.scale.x, bone.initScale.x, ...)

    if !isAnimated("ScaleY", ...)
        interpolate(bone.scale.y, bone.initScale.y, ...)

    // non-interpolated fields are set immediately
    if !isAnimated("IkConstraint", ...)
        bone.ikConstraint = bone.initIkConstraint
}
```

## `interpolateKeyframes()`

With the provided animation frame, determines the keyframes to interpolate the
field by.

The resulting interpolation from the keyframes is interpolated again for
smoothing.

```c
interpolateKeyframes(
    element: enum,
    field: *float,
    keyframes: Keyframe[],
    id: int,
    frame: int,
    smoothFrame: int,
) {
    prev = getPrevFrame(...)
    next = getNextFrame(...)

    // ensure both frames are pointing somewhere
    if prev == -1 {
        prev = next
    } else if next == -1 {
        next = prev
    }

    // if both are -1, then the frame doesn't exist. Do nothing
    if prev == -1 && next == -1
        return

    totalFrames = keyframes[next].frame - keyframes[prev].frame
    currentFrame = frame - keyframes[prev].frame

    result = interpolate(
        currentFrame, totalFrames, keyframes[prev].value, keyframes[next].value
    )

    // result is smoothed
    *field = interpolate(currentFrame, smoothFrame, *field, result)
}
```

## `isAnimated()`

Returns true if a particular element is part of the provided animations.

```c-like
isAnimated(bone_id: int, element: enum, animations: Animation[]): bool {
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
interpolate(current: int, max: int, start_val: float, end_val: float): float {
    if max == 0 || current >= max {
        return end_val
    }
    interp = current / max
    end = end_val - start_val
    result = start_val + (end * interp)

    return result
}
```
