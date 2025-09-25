# Rendering

The final step of any runtime is rendering the animation, or simply providing
the bones to the consumer:

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

  // process core animation logic (via a generic runtime or such)
  animated_bones := skelform::animate(armature, anim_idx, frame, speed);

  // process user options
  for bone, _ := range animated_bones {
    bone.pos += options.position_offset;
    bone.rot += options.rotation_offset;
  }

  // if user chooses to, provide convenient and immediate rendering
  if options.render {
    engine.render(
      animated_bones,
      texture
    );
  }

  // always return animated bones
  return animated_bones;
}
```
