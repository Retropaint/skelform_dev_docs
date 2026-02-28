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
| id        | uint   | Bone ID                                            |
| name      | string | Name of bone                                       |
| pos       | Vec2   | Position of bone                                   |
| rot       | float  | Rotation of bone                                   |
| scale     | Vec2   | Scale of bone                                      |
| parent_id | int    | Bone parent ID (-1 if none)                        |
| tex       | string | Name of texture to use                             |
| zindex    | int    | Z-index of bone (higher index renders above lower) |
| hidden    | bool   | Whether this bone is hidden                        |
| tint      | Vec4   | Color ting                                         |

### Initial Fields

During animation, armature bones need to be modified directly for `smoothing` to
work.

If a bone field is not being animated, it needs to go back to its initial state
with initial fields (starting with `init_`).

`bool` fields use `int` initial fields, as animations cannot store boolean
values (but can still represent them as `0` and `1`)

_The following is **not** an exhaustive list._

| Key        | Type |
| ---------- | ---- |
| init_pos   | Vec2 |
| init_rot   | Vec2 |
| init_scale | Vec2 |
| ...        | ...  |

### Inverse Kinematics

Inverse kinematics is stored in the root (first) bone of each set of IK bones.

Other bones will only have `ik_family_id`, which is -1 by default.

| Key               | Type   | Data                                             |
| ----------------- | ------ | ------------------------------------------------ |
| ik_family_id      | uint   | The ID of family this bone is in (-1 by default) |
| ik_constraint[^1] | string | This family's constraint                         |
| ik_mode[^1]       | string | This family's mode (0 = FABRIK, 1 = Arc)         |
| ik_target_id      | uint   | This set's target bone ID                        |
| ik_bone_ids       | uint[] | This set's ID of bones                           |

## Bone Meshes

Bones with texture meshes have quite a bit of data:

| Key      | Type     | Data                                                                |
| -------- | -------- | ------------------------------------------------------------------- |
| indices  | uint[]   | Array of indices pointing to a vert. Every 3 indices is 1 triangle. |
| vertices | Vertex[] | Array of vertices                                                   |
| binds    | Bind[]   | Array of bone binds                                                 |

### Vertex

| Key      | Type | Data                               |
| -------- | ---- | ---------------------------------- |
| id       | uint | ID of vertex                       |
| pos      | Vec2 | Position of vertex                 |
| uv       | Vec2 | UV of vertex                       |
| init_pos | int  | Helper for initial vertex position |

### Bind

| Key     | Type       | Data                                         |
| ------- | ---------- | -------------------------------------------- |
| id      | int        | ID of bind                                   |
| is_path | bool       | Should this bind behave like a path?         |
| verts   | BindVert[] | Array of vertex data associated to this bind |

### BindVert

| Key    | Type  | Data                           |
| ------ | ----- | ------------------------------ |
| id     | uint  | ID of vertex                   |
| weight | float | Weight assigned to this vertex |

## Animations

| Key       | Type      | Data                               |
| --------- | --------- | ---------------------------------- |
| id        | string    | ID of animation                    |
| name      | string    | Name of animation                  |
| fps       | uint      | Frames per second of animation     |
| keyframes | see below | Data of all keyframes of animation |

### Keyframes

| Key          | Type       | Data                                     |
| ------------ | ---------- | ---------------------------------------- |
| frame        | uint       | frame of keyframe                        |
| bone_id      | uint       | ID of bone that keyframe refers to       |
| element      | string[^1] | Element to be animated by this keyframe  |
| value        | float      | Value to set `element` of bone to        |
| value_str    | string     | String variant of value                  |
| start_handle | float      | Handle to use for start of interpolation |
| end_handle   | float      | Handle to use for end of interpolation   |

## Atlases

| Key      | Type   | Data                        |
| -------- | ------ | --------------------------- |
| filename | string | Name of file for this atlas |
| size     | Vec2   | Size of image (in pixels)   |

## Styles

| Key      | Type    | Data              |
| -------- | ------- | ----------------- |
| id       | uint    | ID of style       |
| name     | string  | Name of style     |
| textures | Texture | Array of textures |

### Textures

Note: Coordinates are in pixels.

| Key       | Type   | Data                                                     |
| --------- | ------ | -------------------------------------------------------- |
| name      | string | Name of texture                                          |
| offset    | Vec2   | Top-left corner of texture in the atlas                  |
| size      | Vec2   | Append to `offset` to get bottom-right corner of texture |
| atlas_idx | uint   | Index of atlas that this texture lives in                |

[^1]: Can be treated as enum
