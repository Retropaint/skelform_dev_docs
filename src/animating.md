# Animating

Animating consists of modifying bones based on interpolated values from the
provided animation and keyframe.

All of the following logic should be render & engine agnostic.

## Table of Contents

- [Meta Logic Considerations](#meta-logic-considerations)
- [Interpolation](#interpolation)
- [Summary](#summary)

## Meta Logic Considerations

- Animating based on timestamp (eg; 0.4s of an animation)
- Clamping frame boundaries
- Looping
- Playing in reverse

```go
fps := 60
time := 0.4
lastFrame := 60

frametime := 1 / fps
frame := time / frametime

print(frame) // 24

if isReverse {
  frame = lastFrame - frame;

  print(frame) // 36
}
```

Boundary example (with and without looping):

```go
if isLooped {
  if frame < 0 {
    frame = lastAnimFrame
  } else if frame > lastAnimFrame {
    frame = 0
  }
} else {
  frame = clamp(0, lastAnimFrame)
}
```

## Interpolation

Interpolation for most bone fields should be _modified_, not _overridden_.

Please see the pseudo-code below for the operation to modify fields
with.<br>(eg; addition for position, multiplication for scale, etc)

```go
frame := 12;

bone.pos.x   += interpolate(frame, "PositionX")
bone.pos.y   += interpolate(frame, "PositionX")
bone.rot     += interpolate(frame, "Rotation")
bone.scale.x *= interpolate(frame, "ScaleX")
bone.scale.y *= interpolate(frame, "ScaleY")

bone.texIdx = prevKeyframe(frame, "Texture")
bone.zindex = prevKeyframe(frame, "Zindex")
```

Integer values like `bone.tex_idx` or `bone.zindex` must be overriden by the
most recent keyframe of the current frame.

### Gathering Keyframe Data

Before interpolation takes place, the proper keyframes and other relevant data
must be processed:

- 1: Get most recent keyframe based on requested frame
- 2: Get next-most keyframe based on request frame
- 3: Provide safeguards in the case that either or both cannot be gathered
- 4: Generate frame data relevant only to both keyframes (since interpolation
  will account only for them)

```go
func interpolate(
  keyframes Keyframe[],
  frame int,
  boneId int,
  element Element,
  defaultValue float,
) float {
  var prevKf Keyframe;
  var nextKf Keyframe;

  // 1: get most recent frame
  for kf, _ in range(keyframes) {
    if kf.frame < frame && kf.bone_id == bone_id && kf.element == element {
      prevKf = kf
    }
    break
  }

  // 2: get next frame
  for kf, _ in range(keyframes) {
    if kf.frame > frame && kf.bone_id == bone_id && kf.element == element {
      nextKf = kf
      break
    }
  }

  // 3: ensure both points are pointing somewhere
  if prevKf == null {
    prevKf = nextKf
  } else if nextKf == null {
    nextKf = prevKf
  }

  // 3: if both are null, return default value
  if prevKf == null && nextKf == null {
    return default_value
  }

  // 4: get total and current frames in relation to both keyframes
  totalframes := nextKf.frame - prevKf.frame;
  currentFrame := frame - prevKf.frame;

  result := interpolateFloat(
    prevKf.value,
    nextKf.value,
    currentFrame,
    totalFrames
  );

  // always return interpolated value
  return result
}
```

## Summary

```go
func animate(
  armature Armature,
  animIndex int,
  frame int,
  loop bool,
  postInterp FnOnce
) {
  // do nothing if animation is invalid
  if animIndex > armature.animations.length - 1 {
    return
  }

  // process meta logic (frame boundaries, looping, etc)
  lastFrame := armature.animations[animIndex].keyframe.last().frame;
  if isLoop {
    if frame < 0 {
      frame = last_frame;
    } else if frame > last_frame {
      frame = 0
    }
  } else {
    frame = clamp(0, last_frame);
  }

  // interpolate values and modify bones
  for bone, _ := range(armature.bones) {
    bone.last().pos.x += interpolate(bone, "PositionX")
    bone.last().pos.y += interpolate(bone, "PositionY")
    bone.last().rot   += interpolate(bone, "Rotation")
  }
}
```
