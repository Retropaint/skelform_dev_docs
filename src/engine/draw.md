# `Draw()`

Uses the bones from `Construct()` to draw the armature.

```typescript
function Draw(bones: Bone[], atlases: Texture2D[], styles: Style[]) {
    // bones with higher zindex should render first
    sort(&bones, zindex)

    for(let bone of bones) {
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
