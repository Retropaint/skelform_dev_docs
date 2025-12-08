# `Animate()` - Generic

Interpolates bone fields based on provided animation data, as well as
interpolating initial states for non-animated fields.

```rust
pub fn animate(
    bones: &mut Vec<Bone>,
    anims: &Vec<&Animation>,
    frames: &Vec<i32>,
    blend_frames: &Vec<i32>,
) {
    for a in 0..anims.len() {
        for b in 0..bones.len() {
            let keyframes = &anims[a].keyframes;
            let b_id = bones[b].id;
            interpolate_bone(&mut bones[b], keyframes, b_id, frames[a], blend_frames[a]);
        }
    }

    for bone in bones {
        reset_bone(bone, frames[0], blend_frames[0], anims);
    }
}
```
