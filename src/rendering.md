# Rendering

Engine runtimes handle the final rendering process, but are also responsible for
the user-facing API.

## Table of Contents

- [Function `animate()`](#function-animate)
- [Function `draw()`](#function-draw)

## Function `animate()`

The main function where bones will be processed, but _not_ rendered.

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
  positionOffset Vec2,
  rotationOffset Vec2,
  // ... and more

  speed float,
  shouldLoop bool,

  // an option can be provided to immediately render the animation,
  // or just provide the bones
  render bool
}

func animate(
  armature *skelform_go.Armature,
  animations []skelform_go.Animation,
  frames []int,
  animOptions AnimOptions,
) Bone[] {
  animatedBones := skelform::animate(armature, animIdx, frame, speed);

  inheritedBones = skelform::inheritance(animated_bones, armature.ikFamilies, []);
  ikRots = make(map[uint]float)
  for i := range(10) {
     ikRots = skelform::inverseKinematics(inheritedBones, armature.IkFamilies)
  }
  finalBones = skelform::inheritance(animatedBones, armature.ikFamilies, ikRot);

  skelform::resetBones(armature.Bones, animations, frames[0], animOptions.BlendFrames);

  // process user options
  for bone, _ := range finalBones {
    bone.pos += options.positionOffset;
    bone.rot += options.rotationOffset;
  }

  // if user chooses to, provide convenient and immediate rendering.
  // see function `draw()` below.
  if options.render {
    draw(finalBones, texture);
  }

  // always return final bones
  return finalBones;
}
```

## Function `draw()`

Draws the provided bones within the engine.

Required arguments:

- Bones to draw
- All styles
- Indexes of all active styles
- Texture image

```go
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

    if texture == None {
      continue
    }

    engine.render(bone, texture)
  }
}
```
