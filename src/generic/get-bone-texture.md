# `GetBoneTexture()`

Helper function to provide the final `Texture` that a bone will use, based on
the provided tex name and styles.

```typescript
function GetBoneTexture(boneTex: String, styles: Style[]): Texture {
    for style in styles {
        const tex: Texture = style.textures.find((tex) => tex.name == boneTex)
        if(tex != undefined) {
            return tex;
        }
    }
    return undefined;
}
```
