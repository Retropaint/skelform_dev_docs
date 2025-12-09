# `Draw()`

Uses the bones from `Construct()` to render the armature.

```c
Draw(bones: Bone[]*, textures: Texture2D[], styles: Style[]) {
    // bones with higher zindex should render first
    sort(&bones, zindex)

    SetupBoneTextures(&bones, styles)

    for bone in bones {
        if bone.tex == "" {
            continue
        }

        // render bone as mesh
        if bone.vertices.len() > 0 {
            drawMesh(createMesh(bone, bone.finalTex, texes[bone.finalTex.atlasIdx))
            continue
        }

        // A lot of game engines have a non-center sprite origin.
        // In this case, the origin is top-left of the sprite.
        // SkelForm uses center origin, so it must be adjusted like so.
        pushCenter: Vec2 = bone.finalTex.size / 2. * bone.scale

        // render bone as regular rect
        drawTexture(bone.finalTex, bone.pos - pushCenter);
    }
}
```
