# File Structure

The editor exports a unique `.skf` file, which can be unzipped to reveal:

- `armature.json` - Armature data (bones, animations, etc)
- `atlasX.png` - Texture atlas(es), starting from 0
- `editor.json` - Editor-only data
- `thumbnail.png` - Armature preview image
- `readme.md` - Little note for runtime devs

This section will only cover the content in `armature.json`.

## Table of Contents

- [`armature.json`](#armaturejson)
- [Bones](#bones)
    - [Initial Fields](#initial-fields)
    - [Inverse Kinematics](#inverse-kinematics)
    - [Meshes](#meshes)
        - [Vertex](#vertex)
        - [Bind](#bind)
        - [BindVert](#bindvert)
- [Animations](#animations)
    - [Keyframes](#keyframes)
- [Atlases](#atlases)
- [Styles](#styles)
    - [Textures](#Textures)
- [Cached Bones](#cached-bones)

## `armature.json`

| Key         | Type        | Default                                     | Description                                  |
| ----------- | ----------- | ------------------------------------------- | -------------------------------------------- |
| version     | string      | `""`                                        | Editor version that exported this file       |
| ik_root_ids | int[]       | `[]`                                        | ID of every inverse kinematics root bone     |
| baked_ik    | bool        | `false`                                     | Was this file exported with baked IK frames? |
| img_format  | string      | `"PNG"`                                     | Exported atlas image format (PNG, JPG, etc)  |
| clear_color | Color[^1]   | <span class="color">`(0, 0, 0, 0)`</span> | Exported clear color of atlas images         |
| bones       | Bone[]      | `[]`                                        | Array of all bones                           |
| animations  | Animation[] | `[]`                                        | Array of all animations                      |
| atlases     | Atlas[]     | `[]`                                        | Array of all atlases                         |
| styles      | Style[]     | `[]`                                        | Array of all styles                          |

## Bones

| Key       | Type      | Default                                           | Description                                        |
| --------- | --------- | ------------------------------------------------- | -------------------------------------------------- |
| id        | uint      | `0`                                               | Bone ID                                            |
| name      | string    | `""`                                              | Name of bone                                       |
| pos       | Vec2      | `(0, 0)`                                          | Position of bone                                   |
| rot       | float     | `0`                                               | Rotation of bone                                   |
| scale     | Vec2      | `(1, 1)`                                          | Scale of bone                                      |
| parent_id | int       | `-1`                                              | Bone parent ID (-1 if none)                        |
| tex       | string    | `""`                                              | Name of texture to use                             |
| zindex    | int       | `0`                                               | Z-index of bone (higher index renders above lower) |
| hidden    | bool      | `false`                                           | Whether this bone is hidden                        |
| tint      | Color[^1] | <span class="color">`(255, 255, 255, 255)`</span> | Color tint                                         |

### Initial Fields

During animation, armature bones need to be modified directly for `smoothing` to
work.

If a bone field is not being animated, it needs to go back to its initial state
with initial fields (starting with `init_`).

`bool` fields use `int` initial fields, as animations cannot store boolean
values (but can still represent them as `0` and `1`)

_The following is **not** an exhaustive list._

| Key        | Type  | Default      |
| ---------- | ----- | ------------ |
| init_pos   | Vec2  | `bone.pos`   |
| init_rot   | float | `bone.rot`   |
| init_scale | Vec2  | `bone.scale` |
| ...        | ...   | ...          |

### Inverse Kinematics

Inverse kinematics is stored in the root (first) bone of each set of IK bones.

Other bones will only have `ik_family_id`, which is -1 by default.

| Key           | Type   | Default    | Description                                      |
| ------------- | ------ | ---------- | ------------------------------------------------ |
| ik_family_id  | uint   | `-1`       | The ID of family this bone is in (-1 by default) |
| ik_constraint | string | `"None"`   | This family's constraint                         |
| ik_mode       | string | `"FABRIK"` | This family's mode (FABRIK, Arc)                 |
| ik_target_id  | uint   | `-1`       | This set's target bone ID                        |
| ik_bone_ids   | uint[] | `[]`       | This set's ID of bones                           |

### Meshes

Only bones that explicitly contain a mesh, will have building data on it.

Bones with a regular texture rect will omit this, as the building data can be
inferred through `Texture` instead.

| Key      | Type     | Default | Description                                            |
| -------- | -------- | ------- | ------------------------------------------------------ |
| vertices | Vertex[] | `[]`    | Array of vertices                                      |
| indices  | uint[]   | `[]`    | Each index is vertex ID. Every 3 IDs forms 1 triangle. |
| binds    | Bind[]   | `[]`    | Array of bone binds                                    |

#### Vertex

A mesh is defined by its vertices, which describe how each point is positioned,
as well as how the texture is mapped (UV).

| Key      | Type | Default  | Description                        |
| -------- | ---- | -------- | ---------------------------------- |
| id       | uint | `0`      | ID of vertex                       |
| pos      | Vec2 | `(0, 0)` | Position of vertex                 |
| uv       | Vec2 | `(0, 0)` | UV of vertex                       |
| init_pos | int  | `pos`    | Helper for initial vertex position |

#### Bind

Meshes can have 'binding' bones to influence a set of vertices. These are the
primary method of animating vertices.

| Key     | Type       | Default | Description                                  |
| ------- | ---------- | ------- | -------------------------------------------- |
| id      | int        | `-1`    | ID of bind                                   |
| is_path | bool       | `false` | Should this bind behave like a path?         |
| verts   | BindVert[] | `[]`    | Array of vertex data associated to this bind |

#### BindVert

Vertices assigned to a bind.

| Key    | Type  | Default | Description                    |
| ------ | ----- | ------- | ------------------------------ |
| id     | uint  | `0`     | ID of vertex                   |
| weight | float | `1`     | Weight assigned to this vertex |

## Animations

| Key       | Type       | Default | Description                        |
| --------- | ---------- | ------- | ---------------------------------- |
| id        | string     | `0`     | ID of animation                    |
| name      | string     | `""`    | Name of animation                  |
| fps       | uint       | `0`     | Frames per second of animation     |
| keyframes | Keyframe[] | `[]`    | Data of all keyframes of animation |

### Keyframes

Keyframes are defined by their `element` (what's animated), as well as either
`value` or `value_str` (what value to animate `element` to)

Eg: `element: PosX` with `value: 20` means 'Position X = 20 at `frame`'

| Key          | Type   | Default | Description                              |
| ------------ | ------ | ------- | ---------------------------------------- |
| frame        | uint   | `0`     | frame of keyframe                        |
| bone_id      | uint   | `0`     | ID of bone that keyframe refers to       |
| element      | string | `""`    | Element to be animated by this keyframe  |
| value        | float  | `0`     | Value to set `element` of bone to        |
| value_str    | string | `""`    | String variant of value                  |
| start_handle | float  | `0.333` | Handle to use for start of interpolation |
| end_handle   | float  | `0.666` | Handle to use for end of interpolation   |

## Atlases

Easily-accessible information about texture atlas files.

| Key      | Type   | Default  | Description                 |
| -------- | ------ | -------- | --------------------------- |
| filename | string | `""`     | Name of file for this atlas |
| size     | Vec2   | `(0, 0)` | Size of image (in pixels)   |

## Styles

Groups of textures.

| Key      | Type    | Default | Description       |
| -------- | ------- | ------- | ----------------- |
| id       | uint    | `0`     | ID of style       |
| name     | string  | `""`    | Name of style     |
| textures | Texture | `[]`    | Array of textures |

### Textures

Note: Coordinates are in pixels.

| Key       | Type   | Default  | Description                                              |
| --------- | ------ | -------- | -------------------------------------------------------- |
| name      | string | `""`     | Name of texture                                          |
| offset    | Vec2   | `(0, 0)` | Top-left corner of texture in the atlas                  |
| size      | Vec2   | `(0, 0)` | Append to `offset` to get bottom-right corner of texture |
| atlas_idx | uint   | `0`      | Index of atlas that this texture lives in                |

## Cached Bones

An extra set of bones is recommended for optimization in the `Construct()`
generic function. This is essentially a clone of the original bone array.

[^1]: A variant of Vec4: `(r, g, b, a)`
