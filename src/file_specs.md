# File Structure

SkelForm utilizes it's own file structure (with the extension `.skf`) upon
exporting from the editor.

`.skf` files are simply zip files, and can be unzipped to reveal:

- `armature.json` - JSON file containing all relevant data (bones, animations,
  etc)
- `textures.png` - Texture atlas / Sprite sheet

## Sample files

- `.skf` file

## `armature.json` Structure

The `armature.json` file primarily consists of an `armature` object with the
following data:

- `bones` - Contains bone data
- `animations` - Array containing all animation data, including keyframes
- `textures` - Array containing individual texture data, in relation to to the
  texture atlas

## Bones

| Key        | Type   | Data                                                  |
| ---------- | ------ | ----------------------------------------------------- |
| id         | int    | ID of bone                                            |
| Name       | string | Name of bone                                          |
| parent_id  | int    | ID of bone's parent                                   |
| tex_idx    | int    | Index of texture in `textures` object                 |
| rot        | float  | Rotation of bone                                      |
| scale      | float  | Scale of bone                                         |
| pos        | float  | Position of bone                                      |
| pivot[^1]  | float  | Pivot of bone                                         |
| zindex[^2] | int    | Z-index of bone<br>(higher index renders above lower) |

[^1]: Currently unused in editor; should not be acknowledged in runtimes

[^2]: Exported as float for compatibility reasons, but should be treated as int

### Bones Structure

The bone array uses a flat structure, with parent-child relationships defined by
`parent_id` on all bones (`-1` if it doesn't have a parent).

**Note**: A child bone is always _after_ it's parent in the array.

## Animations

| Key       | Type      | Data                               |
| --------- | --------- | ---------------------------------- |
| id        | string    | ID of animation                    |
| name      | string    | Name of animation                  |
| fps       | int       | Frames per second of animation     |
| keyframes | see below | Data of all keyframes of animation |

### Keyframes

| Key        | Type       | Data                                    |
| ---------- | ---------- | --------------------------------------- |
| frame      | int        | frame of keyframe                       |
| bone_id    | int        | ID of bone that keyframe refers to      |
| element    | string[^3] | Element to be animated by this keyframe |
| element_id | int        | Same as `element`, but as int           |
| value      | string[^3] | Value to append `element` of bone by    |
| transition | string[^3] | Transition type (linear, sine, etc)     |

## Textures

Note: Coordinates are in pixels.

| Key    | Type   | Data                                                     |
| ------ | ------ | -------------------------------------------------------- |
| name   | string | Name of texture                                          |
| offset | Vec2   | Top-left corner of texture in the atlas                  |
| size   | Vec2   | Append to `offset` to get bottom-right corner of texture |

[^3]: Can be treated as enum
