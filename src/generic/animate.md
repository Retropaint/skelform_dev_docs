# `Animate()` - Generic

Interpolates bone fields based on provided animation data, as well as initial
states for non-animated fields.

```typescript
function animate(
    bones: Bone[], anims: Animation[], frames: int[], smoothFrames: int[]
) {
    for (let a = 0; a < anims.length; a++) {
        for (let b = 0; b < bones.length; b++) {
            interpolateBone(
                bones[b], anims[a].keyframes, bones[b].id, frames[a], smoothFrames[a]
            )
        }
    }

    for (let b = 0; b < bones.length; b++) {
        resetBone(bones[b], ...)
    }
}
```

## `interpolateBone()`

Interpolates one bone's fields based on provided animation data.

```typescript
interpolateBone(
    bone: Bone, keyframes: Keyframe[], boneId: int, frame: int, smoothFrame: int
) {
    interpolateKeyframes("PositionX", bone.pos.x,   ...)
    interpolateKeyframes("PositionY", bone.pos.y,   ...)
    interpolateKeyframes("Rotation",  bone.rot,     ...)
    interpolateKeyframes("ScaleX",    bone.scale.x, ...)
    interpolateKeyframes("ScaleX",    bone.scale.y, ...)

    bone.tex = getPrevFrame("Texture" ...)
    bone.ikConstraint = getPrevFrame("IkConstraint", ...)
}
```

## `resetBone()`

Interpolates one bone's fields to their initial values if not being animated.

```typescript
function resetBone(bone: Bone, frame: int, smoothFrame: int, anims: Animation[]) {
    if(!isAnimated("PositionX", ...))
        interpolate(bone.pos.x, bone.initPos.x, ...)

    if(!isAnimated("PositionY", ...))
        interpolate(bone.pos.y, bone.initPos.y, ...)

    if(!isAnimated("Rotation", ...))
        interpolate(bone.rot, bone.initRot, ...)

    if(!isAnimated("ScaleX", ...))
        interpolate(bone.scale.x, bone.initScale.x, ...)

    if(!isAnimated("ScaleY", ...))
        interpolate(bone.scale.y, bone.initScale.y, ...)

    // non-interpolated fields are set immediately
    if(!isAnimated("IkConstraint", ...))
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
isAnimated(boneId: int, element: enum, animations: Animation[]): bool {
    for anim in anims {
        for kf in anim.keyframes {
            if kf.boneId == boneId && kf.element == element {
                return true
            }
        }
    }
    return false
}
```

## `interpolate()`

Interpolation uses a modified bezier spline (explanation below):

```c
interpolate(
    current: int,
    max: int,
    startVal: float,
    endVal: float,
    startHandle: float,
    endHandle: float
): float {
    // snapping behavior for None transition preset
    if startHandle == 999.0 && endHandle == 999.0 {
        return startVal;
    }
    if max == 0 || current >= max {
        return endVal
    }

    t: float = current / max
    h10: float = 3 * (1 - t)^2 * t
    h01: float = 3 * (1 - t) * t^2
    h11: float = t^3
    progress: float = h10 * startHandle + h01 * endHandle + h11

    return startVal + (endVal - endVal) * progress
}
```

## Bezier Explanation

[A Primer on Bezier Curves](https://pomax.github.io/bezierinfo/#explanation)

The bezier spline uses the following polynomial:

```
value =
    a * (1 - t)^3 +
    b * 3 * (1 - t)^2 * t +
    c * 3 * (1 - t) * t^2 +
    d * t^3
```

This can be simplified into 4 points:

|     | Formula             | Coefficient (a, b, c, d) |
| --- | ------------------- | ------------------------ |
| h00 | (1 - t)^3           | startVal                 |
| h01 | 3 \* (1 - t)^2 \* t | startHandle              |
| h10 | 3 \* (1 - t) \* t^2 | endHandle                |
| h11 | t^3                 | endVal                   |

The above is for a generic bezier spline, however.

In interpolation, it’s best to treat startVal and endVal as 0 and 1 respectively
to represent going from 0% to 100% of the end value. This allows the algorithm
to have a persistent curve, regardless of the actual values being interpolated.

Simplified points:

|     | Formula           | Coefficient (b, c, d) |
| --- | ----------------- | --------------------- |
| h01 | 3 _ (1 - t)^2 _ t | startHandle           |
| h10 | 3 _ (1 -t) _ t^2  | endHandle             |
| h11 | t^3               | 1                     |

Notice that h00 is now gone, as its coefficient (startVal) is always 0 and would
have no effect on the algorithm.

The actual start and end values are applied at the end:

```
progress = h10 * startHandle + h01 * endHandle + h11
value = start + (end - start) * progress
```
