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

```typescript
function interpolateKeyframes(
    element: enum,
    field: float,
    keyframes: Keyframe[],
    id: int,
    frame: int,
    smoothFrame: int,
): float {
    prev = getPrevFrame(...)
    next = getNextFrame(...)

    // ensure both frames are pointing somewhere
    if(prev == -1) {
        prev = next
    } else if(next == -1) {
        next = prev
    }

    // if both are -1, then the frame doesn't exist. Do nothing
    if(prev == -1 && next == -1)
        return

    totalFrames = keyframes[next].frame - keyframes[prev].frame
    currentFrame = frame - keyframes[prev].frame

    result = interpolate(
        currentFrame, totalFrames, keyframes[prev].value, keyframes[next].value
    )

    // result is smoothed
    return interpolate(currentFrame, smoothFrame, field, result)
}
```

## `isAnimated()`

Returns true if a particular element is part of the provided animations.

```typescript
function isAnimated(boneId: int, element: enum, animations: Animation[]): bool {
    for (let anim of anims) {
        for (let kf of anim.keyframes) {
            if (kf.boneId == boneId && kf.element == element) {
                return true;
            }
        }
    }
    return false;
}
```

## `interpolate()`

Interpolation uses a modified bezier spline (explanation below).

Note that 2 helper functions are included below the main function.

```typescript
function interp(
    current: int,
    max: int,
    start_val: float,
    end_val: float,
    start_handle: Vec2,
    end_handle: Vec2,
): float {
    // snapping behavior for None transition preset
    if(start_handle.y == 999.0 && end_handle.y == 999.0) {
        return start_val;
    }
    if(max == 0 || current >= max) {
        return end_val;
    }

    // solve for t with Newton-Raphson
    let initial = current / max
    let t = initial
    for(let i = 0; i < 5; i++) {
        let x = cubic_bezier(t, start_handle.x, end_handle.x)
        let dx = cubic_bezier_derivative(t, start_handle.x, end_handle.x)
        if(abs(dx) < 1e-5 {
            break
        }
        t -= (x - initial) / dx
        t = clamp(t, 0.0, 1.0)
    }

    let progress = cubic_bezier(t, start_handle.y, end_handle.y)
    return start_val + (end_val - start_val) * progress
}

// for both functions below, p0 and p3 are always 0 and 1 respectively

function cubicBezier(t: float, p1: float, p2: float): float {
    let u = 1. - t
    return 3. * u * u * t * p1 + 3. * u * t * t * p2 + t * t * t
}

function cubicBezierDerivative(t: float, p1: float, p2: float): float {
    let u = 1. - t
    return 3. * u * u * p1 + 6. * u * t * (p2 - p1) + 3. * t * t * (1. - p2)
}
```

## Bezier Explanation

_Note: the following explanation is incomplete, as it doesn't include
Newton-Rapshon. However, understanding this is not required to implement the
code above._

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

In interpolation, `startVal` and `endVal` should be 0 and 1 respectively to
represent 0% to 100% of the end value. This allows the algorithm to have a
persistent curve regardless of the actual values being interpolated.

Simplified points:

|     | Formula             | Coefficient (b, c, d) |
| --- | ------------------- | --------------------- |
| h01 | 3 \* (1 - t)^2 \* t | startHandle           |
| h10 | 3 \* (1 -t) \* t^2  | endHandle             |
| h11 | t^3                 | 1                     |

Notice that `h00` is now gone, as its coefficient, `startVal`, is always 0 and
would have no effect on the algorithm.

The actual start and end values are applied at the end:

```
progress = h10 * startHandle + h01 * endHandle + h11
value = start + (end - start) * progress
```
