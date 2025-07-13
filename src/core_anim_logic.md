# Core Animation Logic

This page covers logic that should normally be handled in generic runtimes.

All of the following logic should be render & engine agnostic.

## Table of Contents

- [Dependency Considerations](#dependency-considerations)
- [Meta Logic Considerations](#meta-logic-considerations)
- [Animating Bones](#animating-bones)
- [Static Armatures](#static-armatures)

## Dependency Considerations

Ideally, generic runtimes should have as few dependencies as possible.

Optional features such as loading the `.skf` file should be made optional if it
requires a dependency to function. Otherwise, dependencies are fine for
mandatory functionality.

Additionally, the texture image (`textures.png`) should not be necessary for
generic runtimes to process. All relevant texture data is already included in
`armature.json`.

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

## Animating Bones

Bone animation logic should clone the provided armature's bones, then animate
them via 3 phases:

- Interpolation
- Post-interpolation (customizable)
- Parent inheritance

Once all phases are completed, the animated bones should be returned to the
consumer.

### Interpolation

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

### Post-interpolation

This phase does not have hardcoded logic. Instead, the API must allow custom
code injection for the consumer (eg; closures & anonymous functions).

```rust,noplayground
fn animate(post_interp: Fn) {
  let animated_bones = []

  // run interpolations, populating the bones list

  post_interp(animated_bones);

  // proceed with other phases
}
```

### Parent inheritance

Runs immediately after post-interpolation.

```rust,noplayground
fn inherit_parent(bone: Bone) {
  if bone.parent_id == -1 {
    return
  }

  let parent = find_bone(bone.parent_id);

  bone.rot += parent.rot;
  bone.scale *= parent.scale;
  // account for parent's scale in bone's position
  bone.pos *= parent.scale;

  // rotate bone 'around' parent
  bone.pos = Vec2::new(
    bone.pos.x * cos(parent.rot) - bone.pos.y * sin(parent.rot)
    bone.pos.x * sin(parent.rot) + bone.pos.y * cos(parent.rot)
  );

  bone.pos += parent.pos;

  return bone
}
```

### Overview

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
  // safeguard for no animations; just create an empty one
  let anim: Animation;
  if armature.animations.length == 0 {
    anim = Animation::default();
  } else {
    anim = armature.animations[anim_index];
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

  // run bone animation logic
  let animated_bones = [];
  for bone in armature.bones {
    animated_bones.push(bone)

    // phase 1: interpolation
    animated_bones.last().pos.x += interpolate(bone, "PositionX")
    animated_bones.last().pos.y += interpolate(bone, "PositionY")
    animated_bones.last().rot   += interpolate(bone, "Rotation")

    // phase 2: post-interpolation
    post_interp(animated_bones)

    // phase 3: parent inheritance
    animated_bones = inherit_parent(animated_bones)
  }

  return animated_bones
}
```

## Static Armatures

All of the above logic assumes, and is designed for static armatures (ie. it
has no animation). This is because the [ideal interpolation logic](./interpolation.md) already
handles such cases by default, and the only safeguard required is an animation
length check.

At the very least, it is mandatory in some way to support static armatures. In
this case, only phase 3 (parent inheritance) is required.
