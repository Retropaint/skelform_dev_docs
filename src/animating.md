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

```rust,noplayground
let fps = 60
let time = 0.4
let last_frame = 60

let frametime = 1 / fps
let frame = time / frametime

print(frame) // 24

if reverse {
  frame = last_frame - frame;

  print(frame) // 36
}
```

Boundary example (with and without looping):

```rust,noplayground
if loop {
  if frame < 0 {
    frame = last_anim_frame
  } else if frame > last_anim_frame {
    frame = 0
  }
} else {
  frame = clamp(0, last_anim_frame)
}
```

## Interpolation

Interpolation for most bone fields should be _modified_, not _overridden_.

Please see the pseudo-code below for the operation to modify fields
with.<br>(eg; addition for position, multiplication for scale, etc)

```rust,noplayground
let frame = 12;

bone.pos.x   += interpolate(frame, "PositionX");
bone.pos.y   += interpolate(frame, "PositionX");
bone.rot     += interpolate(frame, "Rotation");
bone.scale.x *= interpolate(frame, "ScaleX");
bone.scale.y *= interpolate(frame, "ScaleY");
bone.tex_idx =  prev_keyframe(frame, "Texture");
bone.zindex  =  prev_keyframe(frame, "Zindex");
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

```rust,noplayground
fn interpolate(
  keyframes: Keyframe[],
  frame: int,
  bone_id: int,
  element: Element,
  default_value: float,
) {
  let prev_kf;
  let next_kf;

  // 1: get most recent frame
  for kf in keyframes {
    if kf.frame < frame && kf.bone_id == bone_id && kf.element == element {
      prev_kf = kf
    }
    break
  }

  // 2: get next frame
  for kf in keyframes {
    if kf.frame > frame && kf.bone_id == bone_id && kf.element == element {
      next_kf = kf
      break
    }
  }

  // 3: ensure both points are pointing somewhere
  if prev_kf == null {
    prev_kf = next_kf
  } else if next_kf == null {
    next_kf = prev_kf
  }

  // 3: if both are null, return default value
  if prev_kf == null && next_kf == null {
    return default_value
  }

  // 4: get total and current frames in relation to both keyframes
  let total_frames = next_kf.frame - prev_kf.frame;
  let current_frame = frame - prev_kf.frame;

  let result = interpolate_float(
    prev_kf.value,
    next_kf.value,
    current_frame,
    total_frames
  );

  // the function should at least return the interpolated value,
  // but other data is always welcome (keyframes, frame data, etc)
  return result
}
```

## Summary

Assuming all 3 phases are in a single function, it should look something like
this:

```rust,noplayground
fn animate(
  armature: Armature,
  anim_index: int,
  frame: int,
  loop: bool,
  post_interp: FnOnce
) {
  // do nothing if animation is invalid
  if anim_index > armature.animations.length - 1 {
    return;
  }

  // process meta logic (frame boundaries, looping, etc)
  let last_frame = armature.animations[anim_index].keyframe.last().frame;
  if loop {
    if frame < 0 {
      frame = last_frame;
    } else if frame > last_frame {
      frame = 0
    }
  } else {
    frame = clamp(0, last_frame);
  }

  // interpolate values and modify bones
  for bone in armature.bones {
    bone.last().pos.x += interpolate(bone, "PositionX")
    bone.last().pos.y += interpolate(bone, "PositionY")
    bone.last().rot   += interpolate(bone, "Rotation")
  }

  return animated_bones
}
```
