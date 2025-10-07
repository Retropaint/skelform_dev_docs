# Rendering (API)

Engine runtimes handle the final rendering process, but are also responsible for
the user-facing API.

## Table of Contents

- [Function `animate()`](#function-animate)

## Function `animate()`

The main function where bones will be processed.

Required arguments:

- Armature data
- Texture image data
- User-defined options
- Animation frame options (specific & time-based)

Required logic:

- Handle all engine-specific quirks (eg; negative Y, reversed rotations, etc)
- If animation is invalid, copy armature bones
- If scale of either axis is negative, reverse rotations

```go
type AnimationOptions struct {
  position_offset Vec2,
  rotation_offset Vec2,
  // ... and more

  speed float,
  should_loop bool,

  // an option can be provided to immediately render the animation,
  // or just provide the bones
  render bool
}

func animate(
  armature Armature,
  texture TextureImage
  anim_idx int,
  frame int,
  time Time
  options AnimationOptions
) Bone[] {
  animated_bones := [];

  // process bones
  animated_bones := skelform::animate(armature, anim_idx, frame, speed);
  animated_bones = skelform::inheritance(animated_bones, armature.ikFamilies, []);
  ikRot := skelform::inverseKinematics(animated_bones, armature.ikFamilies);
  animated_bones = skelform::inheritance(animated_bones, armature.ikFamilies, ikRot);

  // process user options
  for bone, _ := range animated_bones {
    bone.pos += options.position_offset;
    bone.rot += options.rotation_offset;
  }

  // if user chooses to, provide convenient and immediate rendering
  if options.render {
    render(
      animated_bones,
      texture
    );
  }

  // always return animated bones
  return animated_bones;
}

func render(bones []Bone, styles []Style, activeStyles []int, texture TextureImage) {
  for bone, _ ;= bones {
    var texture Texture

    // get texture of first active style
    for styleIdx, _ := bone.style_idxs {
      for activeIdx, _ := activeStyles {
        if activeIdx == styleIdx {
          texture = style.textures[bone.tex_idx]
          break
        }
      }
      if texture != None {
        break
      }
    }

    engine.render(bone, texture)
  }
}
```
