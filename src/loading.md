# Loading (API)

Runtimes need to unzip `.skf` files and extract the following:

- `armature.json`
- `textures.png`

## Table of Contents

- [Function `load()`](#loading)

## Function `load()`

Extract armature and texture data from a `.skf` file.

Required arguments:

- File path, or any kind of IO reader to pass file data to

```go
type SkelformRoot struct {
  Armature Armature
  Textures Image
}

func load(path String) SkelformRoot {
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
