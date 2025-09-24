# File Structure

SkelForm utilizes it's own file format (with the extension `.skf`) upon
exporting from the editor.

`.skf` files are simply zip files, and can be unzipped to reveal:

- `armature.json` - JSON file containing all armature data (bones, animations,
  etc)
- `textures.png` - Texture atlas / sprite sheet
- `editor.json` - (Editor only) JSON file containing data relevant to editors
- `thumbnail.png` - (Editor only) preview image of the armature

## Table of Contents

- [Sample Files](#sample-files)
- [`armature.json` Structure](#armaturejson-structure)
- [Bones](#bones)
  - [Bones Structure](#bones-structure)
- [Animations](#animations)
  - [Keyframes](#keyframes)
- [Textures](#Textures)

## Sample Files

Feel free to download the following to dissect, or just follow along with the
documentation:

- [`.skf` file](https://github.com/Retropaint/skelform_dev_docs/raw/refs/heads/main/skellington.skf)
- <a href="https://raw.githubusercontent.com/Retropaint/skelform_dev_docs/refs/heads/main/armature.json" target="_blank">armature.json</a>
- <a href="https://raw.githubusercontent.com/Retropaint/skelform_dev_docs/refs/heads/main/textures.png" target="_blank">textures.png</a>

## `armature.json` Structure

The `armature.json` file primarily consists of an `armature` object with the
following data:

- `bones` - Contains bone data
- `animations` - Array containing all animation data, including keyframes
- `textures` - Array containing individual texture data, in relation to to the
  texture atlas

## Bones

| Key        | Type   | Data                                                          |
| ---------- | ------ | ------------------------------------------------------------- |
| \_idx[^1]  | int    | index of bone in the array                                    |
| \_name[^1] | string | Name of bone                                                  |
| parent_idx | int    | index of bone's parent                                        |
| style_idxs | int[]  | Indexes of this bone's assigned styles                        |
| tex_idx    | int    | The texture (by index) that this bone will use in it's styles |
| rot        | Vec2   | Rotation of bone                                              |
| scale      | Vec2   | Scale of bone                                                 |
| pos        | Vec2   | Position of bone                                              |
| zindex     | int    | Z-index of bone<br>(higher index renders above lower)         |

[^1]: Debug field & development aid, not required to be parsed

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
| element    | string[^2] | Element to be animated by this keyframe |
| element_id | int        | Same as `element`, but as int           |
| value      | float      | Value to append `element` of bone by    |
| transition | string[^2] | Transition type (linear, sine, etc)     |

## Textures

Note: Coordinates are in pixels.

| Key    | Type   | Data                                                     |
| ------ | ------ | -------------------------------------------------------- |
| \_name | string | Name of texture                                          |
| offset | Vec2   | Top-left corner of texture in the atlas                  |
| size   | Vec2   | Append to `offset` to get bottom-right corner of texture |

[^2]: Can be treated as enum
