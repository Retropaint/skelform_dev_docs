# `Draw()`

Uses the bones from `Construct()` (usually `armature.constructed_bones`) to draw
the armature.

Propagated visibility is handled via a fixed bool array.

```typescript
function Draw(bones: Bone[], visuals: Visuals[], atlases: Texture2D[], styles: Style[]) {
    // bones with higher zindex should render first
    bones.sort((a, b) => {
        if(a.visuals_id == -1 || b.visuals_id == -1) {
            return -1;
        }
        let visualsA = visuals[a.visual_id];
        let visualsB = visuals[b.visual_id];
        return visualsA.zindex - visualsB.zindex;
    })

    // initialize a fixed array of false, for propagated visibility
    let hiddens = new Array(bones.length).fill(false);

    for(let bone of bones) {
        if(bone.visuals_id == -1) {
            continue;
        }
        let visual = visuals[bone.visual_id];

        // save this bone's hidden status so it can be propagated to its children,
        // and ignore rendering if it's hidden
        let hidden = bone.hidden || false
        if (bone.parent_id != -1 && hiddens[bone.parent_id]) {
          hidden = true;
        }
        hiddens[b] = hidden
        if (hidden) {
          continue;
        }

        // get active texture
        let tex = GenericRuntime.getBoneTexture(visual.tex, styles)
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
            drawMesh(bone, visual, tex, realTex)
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
