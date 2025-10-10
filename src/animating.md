# Animating

Bone fields are modified via interpolation of values from animation keyframes.

All of the following logic should be render & engine agnostic.

## Table of Contents

- [Function `animate()`](#function-animate)
- [Function `interpolate()`](#function-interpolate)
- [Function `formatFrame()`](#function-formatframe)
- [Function `timeFrame()`](#function-timeframe)

## Function `animate()`

Interpolation for most bone fields should be _modified_, not _overridden_:

```go
func Animate(bones []Bone, animation Animation, frame int) []Bone {
	for b := range bones {
		bone := &bones[b]
		bone.Rot += interpolate(animation.Keyframes, frame, bone.Id, "Rotation", 0)
		bone.Scale.X *= interpolate(animation.Keyframes, frame, bone.Id, "ScaleX", 1)
		bone.Scale.Y *= interpolate(animation.Keyframes, frame, bone.Id, "ScaleY", 1)
		bone.Pos.X += interpolate(animation.Keyframes, frame, bone.Id, "PositionX", 0)
		bone.Pos.Y += interpolate(animation.Keyframes, frame, bone.Id, "PositionY", 0)
	}

	return bones
}
```

## Function `interpolate()`

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

## Function `formatFrame()`

Helper function to apply effects (looping, reverse, etc) to an animation frame.

```go
func FormatFrame(frame int, animation Animation, reverse bool, loop bool) int {
	lastKf := len(animation.Keyframes) - 1
	lastFrame := animation.Keyframes[lastKf].Frame

	if loop {
		frame %= animation.Keyframes[lastKf].Frame
	}

	if reverse {
		frame = lastFrame - frame
	}

	return frame
}
```

## Function `timeFrame()`

Helper function to provide the appropriate animation frame based on time.

```go
func TimeFrame(time time.Duration, animation Animation, reverse bool, loop bool) int {
	fps := animation.Fps

	var frametime float32 = 1 / float32(fps)
	frame := int(float32(time.Milliseconds()) / frametime / 1000)

	frame = FormatFrame(animation, frame, reverse, loop)

	return frame
}
```
