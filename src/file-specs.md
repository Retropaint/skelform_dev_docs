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
- [Inverse Kinematics](#inverse-kinematics)
- [Visuals](#visuals)
    - [Meshes](#meshes)
        - [Vertex](#vertex)
        - [Bind](#bind)
        - [BindVert](#bindvert)
- [Animations](#animations)
    - [Keyframes](#keyframes)
- [Atlases](#atlases)
- [Styles](#styles)
    - [Textures](#Textures)
- [Initial Fields](#initial-fields)
- [Constructed Bones](#constructed-bones)

## `armature.json`

| Key                | Type                                       | Default                                   | Description                                  |
| ------------------ | ------------------------------------------ | ----------------------------------------- | -------------------------------------------- |
| version            | string                                     | `""`                                      | Editor version that exported this file       |
| baked_ik           | bool                                       | `false`                                   | Was this file exported with baked IK frames? |
| img_format         | string                                     | `"PNG"`                                   | Exported atlas image format (PNG, JPG, etc)  |
| clear_color        | Color[^1]                                  | <span class="color">`(0, 0, 0, 0)`</span> | Exported clear color of atlas images         |
| bones              | [Bone](#bone)[]                            | `[]`                                      | Array of all bones                           |
| animations         | [Animation](#animation)[]                  | `[]`                                      | Array of all animations                      |
| atlases            | [Atlas](#atlas)[]                          | `[]`                                      | Array of all atlases                         |
| styles             | [Styles](#styles)[]                        | `[]`                                      | Array of all styles                          |
| inverse_kinematics | [InverseKinematics](#inverse-kinematics)[] | `[]`                                      | Array of all bone inverse kinematics data    |
| visuals            | [Visuals](#visuals)[]                      | `[]`                                      | Array of all bone visuals (texture, mesh)    |
| physics            | [Physics](#physics)[]                      | `[]`                                      | Array of all bone physics data               |

## Bones

| Key                   | Type      | Default                                           | Description                                        |
| --------------------- | --------- | ------------------------------------------------- | -------------------------------------------------- |
| id                    | uint      | `0`                                               | Bone ID                                            |
| name                  | string    | `""`                                              | Name of bone                                       |
| pos                   | Vec2      | `(0, 0)`                                          | Position of bone                                   |
| rot                   | float     | `0`                                               | Rotation of bone                                   |
| scale                 | Vec2      | `(1, 1)`                                          | Scale of bone                                      |
| parent_id             | int       | `-1`                                              | Bone parent ID (-1 if none)                        |
| tex                   | string    | `""`                                              | Name of texture to use                             |
| zindex                | int       | `0`                                               | Z-index of bone (higher index renders above lower) |
| hidden                | bool      | `false`                                           | Whether this bone is hidden                        |
| tint                  | Color[^1] | <span class="color">`(255, 255, 255, 255)`</span> | Color tint                                         |
| inverse_kinematics_id | int       | `-1`                                              | [Inverse Kinematics](#inverse-kinematics) ID       |
| visuals_id            | int       | `-1`                                              | [Visuals](#visuals) ID                             |
| physics_id            | int       | `-1`                                              | [Physics](#physics) ID                             |

## Inverse Kinematics

Inverse kinematics is stored in the root (first) bone of each set of IK bones.

| Key               | Type   | Default        | Description                                      |
| ----------------- | ------ | -------------- | ------------------------------------------------ |
| family_id         | uint   | `-1`           | The ID of family this bone is in (-1 by default) |
| constraint        | string | `"None"`       | Constraint (Clockwise, CounterClockwise)         |
| mode              | string | `"FABRIK"`     | Mode (FABRIK, Arc)                               |
| target_id         | uint   | `-1`           | Target bone ID                                   |
| bone_ids          | uint[] | `[]`           | ID of all bones in this family                   |
| mimic_target      | bool   | `false`        | Should the last bone follow target's rotation?   |
| init_constraint   | string | `constraint`   |
| init_mode         | string | `mode`         |
| init_mimic_target | bool   | `mimic_target` |

## Visuals

Visual data of each bone (texture, zindex )

| Key         | Type                | Default                                           | Description                                            |
| ----------- | ------------------- | ------------------------------------------------- | ------------------------------------------------------ |
| tex         | string              | `""`                                              | Name of texture to use                                 |
| zindex      | int                 | `0`                                               | Z-index of bone (higher index renders above lower)     |
| tint        | Color[^1]           | <span class="color">`(255, 255, 255, 255)`</span> | Color tint                                             |
| vertices    | [Vertex](#vertex)[] | `[]`                                              | Array of vertices                                      |
| indices     | uint[]              | `[]`                                              | Each index is vertex ID. Every 3 IDs forms 1 triangle. |
| binds       | [Bind](#bind)[]     | `[]`                                              | Array of bone binds                                    |
| init_tex    | Color[^1]           | <span class="color">`(255, 255, 255, 255)`</span> | Color tint                                             |
| init_zindex | Color[^1]           | <span class="color">`(255, 255, 255, 255)`</span> | Color tint                                             |

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
| next_kf      | int    | `-1`    | Index of the next associated keyframe    |
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

## Initial Fields

Some fields in Bones, Visuals, Inverse Kinematics, and Physics have `init_`
counterparts. Their value is duplicated from the original field.

While the original field is modified during animation, `init_` fields are used
to return the original field back to its initial state (see
[ResetBones()](/generic/animate.html#resetbones)).

## Constructed Bones

An extra set of bones is recommended for optimization in the `Construct()`
generic function. This is a clone of the bones array, but with construction
applied to it for use later with `Draw()`.

[^1]: A variant of Vec4: `(r, g, b, a)`
