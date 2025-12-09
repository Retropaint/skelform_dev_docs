# `Load()` - Engine

Reads a SkelForm file (`.skf`) and loads its armature and textures.

The below example assumes `Texture2D` is the engine-specific texture object.

```c
Load(zipPath: string): (Armature, Texture2D[]) {
    zip: Zip = ZipLib::open(zipPath)
    armatureJson: string = zip.byName("armature.json")

    armature: Armature = json::new(&armatureJson)

    textures: Texture2D[]
    for atlas in armature.atlases {
        Image img = zip.byName(atlas.filename)
        textures.push(Texture2D(img))
    }

    return (armature, textures)
}
```
