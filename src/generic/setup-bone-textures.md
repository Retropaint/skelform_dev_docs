# `SetupBoneTextures()`

Helper function to provide the final `Texture` that the bones will use, based on
the provided styles.

```c
SetupBoneTextures(bones: *Bone[], styles: Style[]): Map<int, Texture> {
    finalTextures: Map<int, Texture>
    for bone in bones {
        for style in styles {
            tex: Texture = style.textures.find(|tex| tex.name == bone.tex)
            if tex != None {
                finalTextures.push(<bone.id, bone.finalTex>)
                break
            }
        }
    }
    return finalTextures
}
```
