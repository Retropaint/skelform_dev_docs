# `Draw()`

Uses the bones from `Construct()` to render the armature.

**Parameters**:

- Bones
- Textures
- Styles

**Returns**:

- Void

```rust
/// Draw the provided bones with Macroquad.
pub fn draw(bones: &mut Vec<Bone>, textures: &Vec<Texture2D>, styles: &Vec<&Style>) {
    // bones with higher zindex should render first
    bones.sort_by(|a, b| a.zindex.total_cmp(&b.zindex));

    setup_bone_texes(bones, styles);

    let col = Color::from_rgba(255, 255, 255, 255);
    for bone in bones {
        if bone.tex == "" {
            continue;
        }

        // render bone as mesh
        if bone.vertices.len() > 0 {
            let atlas_idx = bone.final_tex.atlas_idx as usize;
            draw_mesh(&create_mesh(&bone, &bone.final_tex, &texes[atlas_idx]));
            continue;
        }

        // Macroquad's sprite origin is top-left, so this will align them to center origin
        let push_center = bone.final_tex.size / 2. * bone.scale;

        // render bone as regular rect
        draw_texture_ex(
            &texes[bone.final_tex.atlas_idx as usize],
            bone.pos.x - push_center.x,
            bone.pos.y - push_center.y,
            col,
            DrawTextureParams {
                source: Some(Rect {
                    x: bone.final_tex.offset.x,
                    y: bone.final_tex.offset.y,
                    w: bone.final_tex.size.x,
                    h: bone.final_tex.size.y,
                }),
                dest_size: Some(macroquad::prelude::Vec2::new(
                    bone.final_tex.size.x * bone.scale.x,
                    bone.final_tex.size.y * bone.scale.y,
                )),
                rotation: bone.rot,
                ..Default::default()
            },
        );
    }
}
```
