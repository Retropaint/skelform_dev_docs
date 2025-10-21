# Animating

Bone fields are overridden via interpolation of values from animation keyframes.

All of the following logic should be render & engine agnostic.

## Table of Contents

- [Function `animate()`](#function-animate)
- [Function `interpolateKeyframes()`](#function-interpolatekeyframes)
  - [Blending](#blending)
- [Function `resetBones()`](#function-resetbones)

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

## Function `resetBones()`

Reset bone elements to their default values if they haven't been animated.

Must take blending into account.

```go
func ResetBones(bones []Bone, anims []Animation, frame int, blendFrames int) {
	boilerplate = {bone.Id, frame, blendFrames, anims}

	for b := range bones {
		bone := &bones[b]
		resetBoneElement(&bone.Pos.X,   bone.Init_pos.X,   0, ...boilerplate)
		resetBoneElement(&bone.Pos.Y,   bone.Init_pos.Y,   1, ...boilerplate)
		resetBoneElement(&bone.Rot,     bone.Init_rot,     2, ...boilerplate)
		resetBoneElement(&bone.Scale.X, bone.Init_scale.X, 3, ...boilerplate)
		resetBoneElement(&bone.Scale.Y, bone.Init_scale.Y, 4, ...boilerplate)
	}
}

func resetBoneElement(
	value *float32,
	init float32,
	el int,
	bone_id int,
	frame int,
	blendFrames int,
	anims []Animation)
{
	shouldReset := true
	for a := range anims {
		anim := &anims[a]
		for _, kf := range anim.Keyframes {
			if kf.Bone_id == bone_id && kf.Element_id == el {
				shouldReset = false
				break
			}
		}
		if !shouldReset {
			break
		}
	}
	if shouldReset {
		*value = interpolate(frame, blendFrames, *value, init)
	}
}
```
