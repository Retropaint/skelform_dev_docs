# Loading

Generally, all that is needed to load `.skf` files are zip, JSON, and PNG
readers.

For runtimes, the files that need to be loaded are:

- `armature.json`
- `textures.png`

The rest are only relevant to the editor.

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
