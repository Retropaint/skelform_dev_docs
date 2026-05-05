# `Draw()`

Uses the bones from `Construct()` (usually `armature.constructed_bones`) to draw
the armature.

Propagated visibility is handled via a fixed bool array.

```typescript
function Draw(bones: Bone[], atlases: Texture2D[], styles: Style[]) {
    // bones with higher zindex should render first
    sort(&bones, zindex)

    // initialize a fixed array of false, for propagated visibility
    let hiddens = new Array(bones.length).fill(false);

    for(let bone of bones) {
        let hidden = bone.hidden || false

        // if this bone's parent is hidden, so is this
        if (bone.parent_id != -1 && hiddens[bone.parent_id]) {
          hidden = true;
        }

        // add this bone's visibility to the array
        hiddens[b] = hidden

        if (hidden) {
          continue;
        }

        let tex = GenericRuntime.getBoneTexture(bone.tex, styles)
        if !tex {
            continue
        }

        // use tex.atlasIdx to get the atlas that this texture is in
        atlas = atlases[tex.atlasIdx]

        // crop the atlas to the texture
        // here, clip() is assumed to be a texture clipper that takes:
        // (image, top_left, bottom_right)
        // do what is best for the engine
        let realTex = clip(atlas, tex.offset, tex.size)

        // render bone as mesh
        if(bone.vertices.len() > 0) {
            drawMesh(bone, tex, realTex)
            continue
        }

        // A lot of game engines have a non-center sprite origin.
        // In this case, the origin is top-left of the sprite.
        // SkelForm uses center origin, so it must be adjusted like so.
        pushCenter: Vec2 = tex.size / 2. * bone.scale

        // render bone as regular rect
        drawTexture(realTex, bone.pos - pushCenter)
    }
}
```
