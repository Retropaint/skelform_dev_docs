# `Draw()`

Uses the bones from `Construct()` to draw the armature.

```c
Draw(bones: Bone[]*, textures: Texture2D[], styles: Style[]) {
    // bones with higher zindex should render first
    sort(&bones, zindex)

    finalTextures = SetupBoneTextures(bones, styles)

    for bone in bones {
        if !finalTextures.has(bone.id) {
            continue
        }

        // retrieve final textures based on bone id
        tex = finalTextures.get(bone.id)

        // render bone as mesh
        if bone.vertices.len() > 0 {
            drawMesh(bone, tex, texes[bone.finalTex.atlasIdx)
            continue
        }

        // A lot of game engines have a non-center sprite origin.
        // In this case, the origin is top-left of the sprite.
        // SkelForm uses center origin, so it must be adjusted like so.
        pushCenter: Vec2 = tex.size / 2. * bone.scale

        // render bone as regular rect
        drawTexture(tex, bone.pos - pushCenter);
    }
}
```
