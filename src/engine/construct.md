# `Construct()` - Engine

Calls `Construct()` from the generic runtime, then modifies bones based on user
options and engine quirks.

**Parameters**:

- Armature
- User options

**Returns**:

- Constructed bones, to be used later for `Draw()`.

```rust
pub fn construct(armature: &Armature, options: ConstructOptions) -> Vec<Bone> {
    let mut final_bones = generic_runtime::construct(armature);

    for bone in &mut final_bones {
        // engine quirks like negative Y or reversed rotations can be applied here
        bone.pos.y = -bone.pos.y;
        bone.rot = -bone.rot;

        // apply user options
        bone.scale *= Vec2::new(options.scale.x, options.scale.y);
        bone.pos *= Vec2::new(options.scale.x, options.scale.y);
        bone.pos += Vec2::new(options.position.x, options.position.y);

        generic_runtime::check_flip(bone);

        // engine quirks & user options applied to vertices
        for vert in &mut bone.vertices {
            vert.pos.y = -vert.pos.y;
            vert.pos *= Vec2::new(options.scale.x, options.scale.y);
            vert.pos += Vec2::new(options.position.x, options.position.y);
        }
    }

    final_bones
}
```
