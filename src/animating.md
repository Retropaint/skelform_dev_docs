# Animating

Bone fields are modified via interpolation of values from animation keyframes.

All of the following logic should be render & engine agnostic.

## Table of Contents

- [Interpolation](#interpolation)
- [Summary](#summary)

## Interpolation

Interpolation for most bone fields should be _modified_, not _overridden_:

```go
frame := 12;

bone.pos.x   += interpolate(frame, "PositionX")
bone.pos.y   += interpolate(frame, "PositionX")
bone.rot     += interpolate(frame, "Rotation")
bone.scale.x *= interpolate(frame, "ScaleX")
bone.scale.y *= interpolate(frame, "ScaleY")

bone.texIdx  =  prevKeyframe(frame, "Texture")
bone.zindex  =  prevKeyframe(frame, "Zindex")
```

### Gathering Keyframe Data

Before interpolating, the proper keyframes must be fetched:

- 1: Get most recent keyframe of current frame
- 2: Get next-most keyframe of current frame
- 3: Provide safeguards in case either/both are invalid
- 4: Generate frame data from both keyframes

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

  // 1.
  for kf, _ in range(keyframes) {
    if kf.frame < frame && kf.bone_id == bone_id && kf.element == element {
      prevKf = kf
    }
    break
  }

  // 2.
  for kf, _ in range(keyframes) {
    if kf.frame > frame && kf.bone_id == bone_id && kf.element == element {
      nextKf = kf
      break
    }
  }

  // 3.
  if prevKf == null {
    prevKf = nextKf
  } else if nextKf == null {
    nextKf = prevKf
  }
  if prevKf == null && nextKf == null {
    return default_value
  }

  // 4.
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
