# Rendering

Rendering is best handled on a per-environment/engine basis with dedicated
runtimes (known as 'engine runtimes').

This page covers best practices and general advice that applies to any engine
runtime.

## Table of Contents

- [Loading Armatures](#loading-armatures)
- [General `animate()` Function](#general-animate-function)

## Loading Armatures

It is recommended for engine runtimes to provide convenient helpers for loading
`.skf` files directly. This would most likely require dependencies for reading
zip, JSON, and PNG files.

```rust,noplayground
fn load_skelform(zip_path: string): (SkelformArmature, TextureImage){
  let zip = load_zip(zip_path);
  let armature = load_json(zip.by_name("armature.json"));
  let texture_img = load_png(zip.by_name("texture.png"));

  return (armature, texture_img);
}
```

Additionally, an option to load the armature and texture image separately can be
helpful for debugging.

```rust,noplayground
fn load_skelform_scatted(
  armature_path: string,
  texture_path: string
): (SkelformArmature, TextureImage) {
  let armature = load_json(armature_path);
  let texture_img = load_png(texture_path);

  return (armature, texture_img);
}
```

## General `animate()` Function

In this example, we will assume the engine as a simple `engine` variable. The
complexity of the actual code will greatly vary.

```rust,noplayground
struct AnimationOptions {
  position_offset: Vec2,
  rotation_offset: Vec2,
  // ... and more

  speed: float,
  should_loop: bool,

  // an option can be provided to immediately render the animation,
  // or just provide the bones
  render: bool
}

fn animate(
  armature: SkelformArmature,
  texture: TextureImage
  anim_idx: int,
  frame: i32,
  time: Time
  options: AnimationOptions
): Bone[] {
  let animated_bones = [];

  // process core animation logic (via a generic runtime or such)
  animated_bones = skelform::animate(armature, anim_idx, frame, speed);

  // process user options
  for bone in animated_bones {
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
