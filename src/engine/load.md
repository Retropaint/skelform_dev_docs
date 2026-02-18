# `Load()` - Engine

Reads a SkelForm file (`.skf`) and loads its armature and textures.

The below example assumes `Texture2D` is the engine-specific texture object.

```typescript
function Load(zipPath: string): (Armature, Texture2D[]) {
    zip: Zip = ZipLib.open(zipPath)
    armatureJson: string = zip.byName("armature.json")

    armature: Armature = Json.new(&armatureJson)

    textures: Texture2D[]
    for(let atlas of armature.atlases) {
        Image img = zip.byName(atlas.filename)
        textures.push(Texture2D(img))
    }

    return (armature, textures)
}
```
