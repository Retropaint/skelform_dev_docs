# `Load()` - Engine

Reads a SkelForm file (`.skf`) and loads its armature and textures.

```rust
/// Load a SkelForm armature.
/// The file to load is the zip that is provided by SkelForm export.
pub fn load(zip_path: &str) -> (Armature, Vec<Texture2D>) {
    let file = std::fs::File::open(zip_path).unwrap();
    let mut zip = zip::ZipArchive::new(file).unwrap();
    let mut armature_json = String::new();
    zip.by_name("armature.json")
        .unwrap()
        .read_to_string(&mut armature_json)
        .unwrap();

    let armature: Armature = serde_json::from_str(&armature_json).unwrap();

    let mut texes = vec![];

    for atlas in &armature.atlases {
        let mut img = vec![];
        zip.by_name(&atlas.filename.to_string())
            .unwrap()
            .read_to_end(&mut img)
            .unwrap();
        texes.push(Texture2D::from_file_with_format(
            &img,
            Some(ImageFormat::Png),
        ));
    }

    (armature.clone(), texes)

```
