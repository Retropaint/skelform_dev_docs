# `Load()` - Engine

Reads a SkelForm file (`.skf`) and loads its armature and textures.

The below example assumes `Texture2D` is the engine-specific texture object.

```c
(Armature, Texture2D[]) load(string zip_path) {
    let file = file::open(zip_path).unwrap()
    let mut zip = zip::new(file).unwrap()
    let mut armature_json = String::new()
    armature_json = zip.byName("armature.json")

    Armature armature = json::new(&armature_json)

    Texture2D textures[]
    for atlas in armature.atlases {
        Image img = []
        img = zip.byName(atlas.filename)
        textures.push(Texture2D(img))
    }

    return (armature, textures)
}
```
