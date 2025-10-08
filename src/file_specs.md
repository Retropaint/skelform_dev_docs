# File Structure

The editor exports a unique `.skf` file, which can be unzipped to reveal:

- `armature.json` - Armature data (bones, animations, etc)
- `textures.png` - Texture atlas / sprite sheet
- `editor.json` - Editor-only data
- `thumbnail.png` - Armature preview image

## Table of Contents

- [Sample Files](#sample-files)
- [`armature.json` Structure](#armaturejson-structure)
- [Bones](#bones)
- [Animations](#animations)
  - [Keyframes](#keyframes)
- [Textures](#Textures)

## Sample Files

The following can be downloaded for reference:

- [`.skf` file](https://github.com/Retropaint/skelform_dev_docs/raw/refs/heads/main/skellington.skf)
- <a href="https://raw.githubusercontent.com/Retropaint/skelform_dev_docs/refs/heads/main/armature.json" target="_blank">armature.json</a>
- <a href="https://raw.githubusercontent.com/Retropaint/skelform_dev_docs/refs/heads/main/textures.png" target="_blank">textures.png</a>

## `armature.json` Structure

Primarily consists of an `armature` object with the following data:

- `bones` - Data of all bones
- `animations` - Array of all animation data, including keyframes
- `styles` - Array of style data, including texture coordinates

## Bones

| Key        | Type   | Data                                                  |
| ---------- | ------ | ----------------------------------------------------- |
| id         | int    | bone ID                                               |
| \_name[^1] | string | Name of bone                                          |
| parent_id  | int    | bone parent ID                                        |
| style_idxs | int[]  | Indexes of this bone's assigned styles                |
| tex_idx    | int    | Bone will use this texture index in styles            |
| rot        | Vec2   | Rotation of bone                                      |
| scale      | Vec2   | Scale of bone                                         |
| pos        | Vec2   | Position of bone                                      |
| zindex     | int    | Z-index of bone<br>(higher index renders above lower) |

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
| \_element  | string[^2] | Element to be animated by this keyframe |
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
