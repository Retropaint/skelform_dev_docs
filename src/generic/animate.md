## Table of Contents

- [`Animate()`](#animate)
    - [`interpolateKeyframes()`](#interpolatekeyframes)
    - [`resetBones()`](#resetbones)
    - [`interpolate()`](#interpolate)
        - [Bezier Explanation](#bezier-explanation)

# `Animate()`

Interpolates bone fields based on provided animation data, as well as initial
states for non-animated fields.

<pre> <code class="language-typescript hljs">function Animate(bones: Bone[], anims: Animation[], frames: int[], smoothFrames: int[]) {
    for (let a = 0; a < anims.length; a++) {
        for (k = 0; k < anims[a].keyframes.length; k++) {
            let kf = anims[a].keyframes[k];

            // only prev keyframes are considered
            if (kf.frame > frames[a]) {
                break;
            }

            // refer keyframe to itself, if there's no next
            if (kf.next_kf == -1) {
                kf.next_kf = k;
            }

            let nextKf = anims[a].keyframes[kf.next_kf];

            // skip keyframe if it's not the last, and would not be animated
            let isLast = kf.next_kf == k;
            let isBeforeFrame = nextKf.frame < frames[a];
            if (isBeforeFrame && !isLast) {
                continue;
            }

            // interpolate the relevant bone field, based on this and next keyframes' values
            let bone = bones[kf.bone_id]
            if (kf.element == "PositionX")
                interpolateKeyframes(bone.pos.x, kf, nextKf, frames[a], smoothFrames[a]);
            if (kf.element == "PositionY")
                interpolateKeyframes(bone.pos.y, kf, nextKf, frames[a], smoothFrames[a]);
            if (kf.element == "Rotation")
                interpolateKeyframes(bone.rot, kf, nextKf, frames[a], smoothFrames[a]);
            if (kf.element == "ScaleX")
                interpolateKeyframes(bone.scale.x, kf, nextKf, frames[a], smoothFrames[a]);
            if (kf.element == "ScaleY")
                interpolateKeyframes(bone.scale.y, kf, nextKf, frames[a], smoothFrames[a]);
            if (kf.element == "Hidden")
                bone.hidden = kf.value == 1;
        }
    }

    // bones that are not being animated should return to their initial states
    resetBones(bones, anims, frames[0], smoothFrames[0])
}</code> </pre>

## `resetBones()`

Animate bones back to their initial states, if they're not being animated.

This makes use of each bones' `init_` fields (`init_pos`, `init_rot`, etc).

**Example:** animation 1 was played and rotated the arm bone, but animation 1 is
not being played anymore. That arm bone must now return to its initial rotation.

<pre> <code class="language-typescript hljs">function resetBones(bones, animations, frame, smoothFrame) {
    elementMap = {}

    // add every element that's being animated for each bone
    anims.forEach(anim => {
        anim.keyframes.forEach(kf => {
            elementMap[kf.bone_id].push(kf.element)
        })
    jj

    // animate every element that's not in the map, back to its initial state
    bones.forEach(bone => {
        if (!elementMap[kf.bone_id]["PositionX"])
            interpolate(frame, smoothFrame, bone.pos.X, bone.init_pos.X, z, z)
        if (!elementMap[kf.bone_id]["PositionY"])
            interpolate(frame, smoothFrame, bone.pos.Y, bone.init_pos.Y, z, z)
        if (!elementMap[kf.bone_id]["Rotation"])
            interpolate(frame, smoothFrame, bone.rot, bone.init_rot, z, z)
        if (!elementMap[kf.bone_id]["ScaleX"])
            interpolate(frame, smoothFrame, bone.scale.X, bone.init_scale.X, z, z)
        if (!elementMap[kf.bone_id]["ScaleY"])
            interpolate(frame, smoothFrame, bone.scale.Y, bone.init_scale.X, z, z)
        if (!elementMap[kf.bone_id]["Hidden"])
            bone.hidden = bone.init_hidden
    })   
}
</code> </pre>

## `interpolateKeyframes()`

With the provided animation frame, determines the keyframes to interpolate the
field by.

The resulting interpolation from the keyframes is interpolated again for
smoothing.

<pre> <code class="language-typescript hljs">function interpolateKeyframes(
    field: float, prevKf: Keyframe, nextKf: Keyframe, frame: int, smoothFrame: int
): float {
    const totalFrames = nextKf.frame - prevKf.frame
    const currentFrame = frame - prevKf.frame

    // interpolate from current to next keyframe value
    const result = interpolate(
        currentFrame,
        totalFrames,
        prevKf.value,
        nextKf.value,
        nextKf.start_handle,
        nextKf.end_handle
    )

    // using the requested smoothing frames, interpolate the current field to the target value
    let z = { x: 0, y: 0 }
    interpolate(currentFrame, smoothFrame, field, result, z, z)
}
</code> </pre>

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
    // snapping behavior for Snap transition preset
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
