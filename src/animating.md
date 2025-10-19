# Animating

Bone fields are overridden via interpolation of values from animation keyframes.

All of the following logic should be render & engine agnostic.

## Table of Contents

- [Function `animate()`](#function-animate)
- [Function `interpolateKeyframes()`](#function-interpolatekeyframes)
  - [Blending](#blending)
- [Function `formatFrame()`](#function-formatframe)
- [Function `timeFrame()`](#function-timeframe)

## Function `animate()`

Bone fields are overidden by interpolating their respective values from
keyframes.

Keyframes only store float values, so each field is interpolated down to a float
value (eg; X and Y of a vec2).

```go
func Animate(bones []Bone, animation Animation, frame int) []Bone {
	boilerplate := {animation.Keyframes, frame, bone.Id, blendFrames}

	for b := range bones {
		bone := &bones[b]
		interpolateKeyframes(&bone.Rot,     "Rotation",  ..boilerplate)
		interpolateKeyframes(&bone.Scale.X, "ScaleX",    ..boilerplate)
		interpolateKeyframes(&bone.Scale.Y, "ScaleY",    ..boilerplate)
		interpolateKeyframes(&bone.Pos.X,   "PositionX", ..boilerplate)
		interpolateKeyframes(&bone.Pos.Y,   "PositionY", ..boilerplate)
	}

	return bones
}
```

## Function `interpolateKeyframes()`

Before interpolating, the proper keyframes must be fetched:

- 1: Get most recent keyframe of current frame
- 2: Get next-most keyframe of current frame
- 3: Provide safeguards in case either/both are invalid
- 4: Generate frame data from both keyframes

Once generated, the values from both keyframes are interpolated and
returned/assigned.

```go
func interpolateKeyframes(
	field *float32,
	element string,
	keyframes []Keyframe,
	frame int,
	boneId int,
	blendFrames int,
) {
	var prevKf Keyframe
	var nextKf Keyframe
	prevKf.Frame = -1
	nextKf.Frame = -1

	for _, kf := range keyframes {
		if kf.Frame < frame && kf.Bone_id == boneId && kf.Element == element {
			prevKf = kf
		}
	}

	for _, kf := range keyframes {
		if kf.Frame >= frame && kf.Bone_id == boneId && kf.Element == element {
			nextKf = kf
			break
		}
	}

	if prevKf.Frame == -1 {
		prevKf = nextKf
	} else if nextKf.Frame == -1 {
		nextKf = prevKf
	}

	if prevKf.Frame == -1 && nextKf.Frame == -1 {
		return
	}

	totalFrames := nextKf.Frame - prevKf.Frame
	currentFrame := frame - prevKf.Frame

	// assume a generic, linear interpolate() func that takes
	// (currentStep, maxStep, startValue, endValue)
	result := interpolate(currentFrame, totalFrames, prevKf.Value, nextKf.Value)
	blend := interpolate(currentFrame, blendFrames, *field, result)

	*field = blend
}
```

### Blending

As shown above, the resulting interpolation is 'blended'.

Blending allows for smoother movement & changes, often used for animation
transitions.

It only requires generic interpolation, from the starting value to the result.

## Function `formatFrame()`

Helper function to apply effects to an animation frame (looping, reverse, etc).

```go
func FormatFrame(frame int, animation Animation, reverse bool, loop bool) int {
	lastKf := len(animation.Keyframes) - 1
	lastFrame := animation.Keyframes[lastKf].Frame

	if loop {
		frame %= lastFrame
	}

	if reverse {
		frame = lastFrame - frame
	}

	return frame
}
```

## Function `timeFrame()`

Helper function to provide the appropriate animation frame based on time.

**Recommended**: include `FormatFrame()` and it's options, to reduce verbosity
in user code.

```go
func TimeFrame(time time.Duration, animation Animation, reverse bool, loop bool) int {
	fps := animation.Fps

	frametime := 1. / float32(fps)
	frame := int(float32(time.Milliseconds()) / frametime / 1000)

	frame = FormatFrame(animation, frame, reverse, loop)

	return frame
}
```
