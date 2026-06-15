# `Load()` - Engine

Reads a SkelForm file (`.skf`) and loads its armature and textures.

The below example assumes `Texture2D` is the engine-specific texture object.

```typescript
function Load(zipPath: String): (Armature, Texture2D[]) {
    const zip: Zip = ZipLib.open(zipPath);
    const armatureJson: String = zip.byName("armature.json");

    const armature: Armature = Json.new(armatureJson);

    let textures: Texture2D[];
    armature.atlases.forEach((atlas) => {
        Image img = zip.byName(atlas.filename);
        textures.push(Texture2D(img));
    })

    return (armature, textures);
}
```
