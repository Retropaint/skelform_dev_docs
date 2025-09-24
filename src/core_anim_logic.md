# Core Animation Logic

This page covers logic that should normally be handled in generic runtimes.

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

Integer values like `bone.tex_idx` or `bone.zindex` must be overriden by the
most recent keyframe of the current frame.

```rust,noplayground
let frame = 12;

bone.pos.x   += interpolate(frame, "PositionX");
bone.pos.y   += interpolate(frame, "PositionX");
bone.rot     += interpolate(frame, "Rotation");
bone.scale.x *= interpolate(frame, "ScaleX");
bone.scale.y *= interpolate(frame, "ScaleY");
bone.tex_idx = last_keyframe_of(frame, "Texture");
bone.zindex  = last_keyframe_of(frame, "Zindex");
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
  // do nothing without an animation
  if armature.animations.length == 0 {
    return;
  }

  let anim = armature.animations[anim_index];

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

  // run bone animation logic
  let animated_bones = [];
  for bone in armature.bones {
    animated_bones.push(bone)

    animated_bones.last().pos.x += interpolate(bone, "PositionX")
    animated_bones.last().pos.y += interpolate(bone, "PositionY")
    animated_bones.last().rot   += interpolate(bone, "Rotation")
  }

  return animated_bones
}
```
