# Loading

Runtimes need unzip to the `.skf` file and load 2 files:

- `armature.json`
- `textures.png`

Other files are only relevant to the editor.

```go
type SkelformRoot struct {
  Armature Armature
  Textures Image
}

func load_armature(path String) SkelformRoot {
  var root SkelformRoot
  zip := zip.open(path)
  for file, _ := range(zip) {
    switch file.name {
      case "armature.json":
        root.armature = json.Load(file)
      case "textures.png":
        root.textures = png.Load(file)
    }
  }

  return root
}
```

The `root` can then be used to pass around the armature and texture(s).
