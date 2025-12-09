# `SetupBoneTextures()`

Helper function to provide the final `Texture` that the bones will use, based on
the provided styles.

```c
SetupBoneTextures(bones: Bone[]*, styles: Style[]) {
    for bone in bones {
        for style in styles {
            tex: Texture = style.textures.find(|tex| tex.name == bone.tex)
            if tex != None {
                bone.finalTex = tex
                break
            }
        }
    }
}
```

## `bone.finalTex`

For the above helper to work, `Bone` must have a `finalTex: Texture` field.

This is not included in file spec to prevent cluttering `Bone` fields.
