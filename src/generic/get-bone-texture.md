# `GetBoneTexture()`

Helper function to provide the final `Texture` that a bone will use, based on
the provided tex name and styles.

```typescript
GetBoneTexture(boneTex: string, styles: Style[]): Texture {
    for style in styles {
        tex: Texture = style.textures.find(|tex| tex.name == bone.tex)
        if(tex != None) {
            return tex
        }
    }
    return undefined
}
```
