# `Draw()`

Uses the bones from `Construct()` (usually `armature.constructed_bones`) to draw
the armature.

Propagated visibility is handled via a fixed Bool array.

**Note:** for simplicity reasons, meshes do _not_ need to account for pivots.

```typescript
function Draw(
    bones: Bone[],
    visuals: Visuals[],
    atlases: Texture2D[],
    styles: Style[],
) {
    // bones with higher zindex should render first
    bones.sort((a, b) => {
        if (a.visuals_id == -1 || b.visuals_id == -1) {
            return -1;
        }
        const visualsA = visuals[a.visual_id];
        const visualsB = visuals[b.visual_id];
        return visualsA.zindex - visualsB.zindex;
    });

    bone.forEach((bone) => {
        if (bone.hidden || bone.visuals_id == -1) {
            return;
        }
        const visual = visuals[bone.visual_id];

        // get active texture
        const tex: Texture = GenericRuntime.getBoneTexture(visual.tex, styles);
        if (!tex) {
            return;
        }

        // will be used to flip pivots if necessary
        let dir = isFacingLeft(bone.scale) ? -1 : 1;

        // setup pivot
        let pivotPos = visual.pivot_pos * tex.size;
        pivotPos = rotateVec2(pivotPos, bone.rot * dir);
        pivotPos *= bone.scale * visual.pivot_scale;
        // invert Y if engine is -Y
        pivotPos.y = -pivotPos.y;

        // use tex.atlasIdx to get the atlas that this texture is in
        const atlas = atlases[tex.atlasIdx];

        // crop the atlas to the texture
        // here, clip() is assumed to be a texture clipper that takes:
        // (image, top_left, bottom_right)
        // do what is best for the engine
        const realTex = clip(atlas, tex.offset, tex.size);

        // render bone as mesh.
        // note: meshes do not need to account for pivots.
        if (visual.vertices.length > 0) {
            drawMesh(bone, visual, tex, realTex);
            return;
        }

        // A lot of game engines have a non-center sprite origin.
        // In this case, the origin is top-left of the sprite.
        // SkelForm uses center origin, so it must be adjusted like so.
        const pushCenter: Vec2 = (tex.size / 2) * bone.scale;

        let finalRot: Float = bone.rot + visual.pivot_rot * dir;
        let finalScale: Vec2 = bone.scale * visual.pivot_scale;

        // render bone as regular rect.
        // Assume this is a draw function that takes (texture, pos, rot, scale)
        let pos = bone.pos + pivotPos - pushCenter;
        drawTexture(realTex, pos, finalRot, finalScale);
    });
}
```
