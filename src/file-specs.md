# File Structure

The editor exports a unique `.skf` file, which can be unzipped to reveal:

- `armature.json` - Armature data (bones, animations, etc)
- `atlasX.png` - Texture atlas(es), X being index (0, 1, etc)
- `editor.json` - Editor-only data
- `thumbnail.png` - Armature preview image
- `readme.md` - Little note for runtime devs

This section will only cover the content in `armature.json`.

## Table of Contents

- [`armature.json`](#armaturejson)
- [Bones](#bones)
  - [Initial Fields](#initial-fields)
  - [Inverse Kinematics](#inverse-kinematics)
- [Bone Meshes](#bone-meshes)
  - [Vertex](#vertex)
  - [Bind](#bind)
  - [BindVert](#bindvert)
- [Animations](#animations)
  - [Keyframes](#keyframes)
- [Textures](#Textures)

## `armature.json`

Primarily consists of an `armature` object with the following data:

- `version` - SkelForm editor version used to export this file
- `bones` - Data of all bones
- `ik_root_ids` - Array of bone IDs that contain inverse kinematics data
- `animations` - Array of all animation data, including keyframes
- `atlases` - Array of texture atlases
- `styles` - Array of style data, including texture coordinates

All fields below can be parsed and used as needed for runtime APIs. However, it
is not mandatory to parse _all_ fields.

## Bones

| Key       | Type   | Data                                               |
| --------- | ------ | -------------------------------------------------- |
| id        | int    | Bone ID                                            |
| name      | string | Name of bone                                       |
| parent_id | int    | Bone parent ID (-1 if none)                        |
| tex       | string | Name of texture to use                             |
| rot       | Vec2   | Rotation of bone                                   |
| scale     | Vec2   | Scale of bone                                      |
| pos       | Vec2   | Position of bone                                   |
| zindex    | int    | Z-index of bone (higher index renders above lower) |

### Initial Fields

During animation, armature bones need to be modified directly for `smoothing` to
work.

If a bone field is not being animated, it needs to go back to its initial state
with initial fields (starting with `init_`).

_The following is **not** an exhaustive list._

| Key        | Type |
| ---------- | ---- |
| init_rot   | Vec2 |
| init_scale | Vec2 |
| init_pos   | Vec2 |
| ...        | ...  |

### Inverse Kinematics

Inverse kinematics is stored in the root (first) bone of each set of IK bones.

Other bones will only have `ik_family_id`, which is -1 by default.

| Key               | Type   | Data                                             |
| ----------------- | ------ | ------------------------------------------------ |
| ik_family_id      | int    | The ID of family this bone is in (-1 by default) |
| ik_constraint     | int    | This family's constraint                         |
| ik_constraint_str | string | This family's constraint (as string)             |
| ik_mode           | int    | This family's mode (0 = FABRIK, 1 = Arc)         |
| ik_mode_str       | string | This family's mode (as string)                   |
| ik_target_id      | int    | This set's target bone ID                        |
| ik_bone_ids       | int[]  | This set's ID of bones                           |

## Bone Meshes

Bones with texture meshes have quite a bit of data:

| Key      | Type     | Data                                                                |
| -------- | -------- | ------------------------------------------------------------------- |
| indices  | int[]    | Array of indices pointing to a vert. Every 3 indices is 1 triangle. |
| vertices | Vertex[] | Array of vertices                                                   |
| binds    | Bind[]   | Array of bone binds                                                 |

### Vertex

| Key      | Type | Data                               |
| -------- | ---- | ---------------------------------- |
| id       | int  | ID of vertex                       |
| pos      | Vec2 | Position of vertex                 |
| uv       | Vec2 | UV of vertex                       |
| init_pos | int  | Helper for initial vertex position |

### Bind

| Key     | Type       | Data                                         |
| ------- | ---------- | -------------------------------------------- |
| id      | int        | ID of bind                                   |
| is_path | int        | Should this bind behave like a path?         |
| verts   | BindVert[] | Array of vertex data associated to this bind |

### BindVert

| Key    | Type | Data                           |
| ------ | ---- | ------------------------------ |
| id     | int  | ID of vertex                   |
| weight | int  | Weight assigned to this vertex |

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

## Atlases

| Key      | Type   | Data                        |
| -------- | ------ | --------------------------- |
| filename | string | Name of file for this atlas |
| size     | Vec2   | Size of image (in pixels)   |

## Styles

| Key      | Type    | Data              |
| -------- | ------- | ----------------- |
| id       | int     | ID of style       |
| name     | string  | Name of style     |
| textures | Texture | Array of textures |

### Textures

Note: Coordinates are in pixels.

| Key       | Type   | Data                                                     |
| --------- | ------ | -------------------------------------------------------- |
| name      | string | Name of texture                                          |
| offset    | Vec2   | Top-left corner of texture in the atlas                  |
| size      | Vec2   | Append to `offset` to get bottom-right corner of texture |
| atlas_idx | int    | Index of atlas that this texture lives in                |

[^2]: Can be treated as enum
