# File Structure

The editor exports a unique `.skf` file, which can be unzipped to reveal:

- `armature.json` - Armature data (bones, animations, etc)
- `atlasX.png` - Texture atlas(es), starting from 0
- `editor.json` - Editor-only data
- `thumbnail.png` - Armature preview image
- `readme.md` - Little note for runtime devs

This section will only cover the content in `armature.json`.

## Table of Contents

- [Armature](#armature)
- [Bones](#bones)
- [Animations](#animations)
    - [Keyframess](#keyframes)
- [Atlases](#atlases)
- [Styles](#styles)
    - [Textures](#textures)
- [Visuals](#visuals)
    - [Vertex](#vertex)
    - [Bind](#bind)
    - [BindVert](#bindvert)
- [Inverse Kinematics](#inverse-kinematics)
- [Physics](#physics)
- [Initial Fields](#initial-fields)
- [Constructed Bones](#constructed-bones)

## Armature

| Key                | Type                                       | Default                                   | Description                                  |
| ------------------ | ------------------------------------------ | ----------------------------------------- | -------------------------------------------- |
| version            | String                                     | `""`                                      | Editor version that exported this file       |
| baked_ik           | Bool                                       | `false`                                   | Was this file exported with baked IK frames? |
| img_format         | String                                     | `"PNG"`                                   | Exported atlas image format (PNG, JPG, etc)  |
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
| id                    | Uint      | `0`                                               | Bone ID                                            |
| name                  | String    | `""`                                              | Name of bone                                       |
| pos                   | Vec2      | `(0, 0)`                                          | Position of bone                                   |
| rot                   | Float     | `0`                                               | Rotation of bone                                   |
| scale                 | Vec2      | `(1, 1)`                                          | Scale of bone                                      |
| parent_id             | Int       | `-1`                                              | Bone parent ID (-1 if none)                        |
| tex                   | String    | `""`                                              | Name of texture to use                             |
| zindex                | Int       | `0`                                               | Z-index of bone (higher index renders above lower) |
| hidden                | Bool      | `false`                                           | Whether this bone is hidden                        |
| tint                  | Color[^1] | <span class="color">`(255, 255, 255, 255)`</span> | Color tint                                         |
| inverse_kinematics_id | Int       | `-1`                                              | [Inverse Kinematics](#inverse-kinematics) ID       |
| visuals_id            | Int       | `-1`                                              | [Visuals](#visuals) ID                             |
| physics_id            | Int       | `-1`                                              | [Physics](#physics) ID                             |

## Inverse Kinematics

Inverse kinematics is stored in the root (first) bone of each set of IK bones.

| Key               | Type   | Default        | Description                                      |
| ----------------- | ------ | -------------- | ------------------------------------------------ |
| family_id         | Uint   | `-1`           | The ID of family this bone is in (-1 by default) |
| constraint        | String | `"None"`       | Constraint (Clockwise, CounterClockwise)         |
| mode              | String | `"FABRIK"`     | Mode (FABRIK, Arc)                               |
| target_id         | Uint   | `-1`           | Target bone ID                                   |
| bone_ids          | Uint[] | `[]`           | ID of all bones in this family                   |
| mimic_target      | Bool   | `false`        | Should the last bone follow target's rotation?   |
| init_constraint   | String | `constraint`   | Initial field for `constraint`[^2]               |
| init_mode         | String | `mode`         | Initial field for `mode`[^2]                     |
| init_mimic_target | Bool   | `mimic_target` | Initial field for `mimic_target`[^2]             |

## Visuals

Visual data of each bone (texture & mesh)

| Key         | Type                | Default                                           | Description                                            |
| ----------- | ------------------- | ------------------------------------------------- | ------------------------------------------------------ |
| tex         | String              | `""`                                              | Name of texture to use                                 |
| zindex      | Int                 | `0`                                               | Z-index of bone (higher index renders above lower)     |
| tint        | Color[^1]           | <span class="color">`(255, 255, 255, 255)`</span> | Multiplicative color tint for texture                  |
| vertices    | [Vertex](#vertex)[] | `[]`                                              | Array of vertices                                      |
| indices     | Uint[]              | `[]`                                              | Each index is vertex ID. Every 3 IDs forms 1 triangle. |
| binds       | [Bind](#bind)[]     | `[]`                                              | Array of bone binds                                    |
| init_tex    | String              | `""`                                              | Initial field for `tex`[^2]                            |
| init_tint   | Color[^1]           | <span class="color">`(255, 255, 255, 255)`</span> | Initial field for `tint`[^2]                           |
| init_zindex | Int                 | `0`                                               | Initial field for `zindex`[^2]                         |

#### Vertex

A mesh is defined by its vertices, which describe how each point is positioned,
as well as how the texture is mapped (UV).

| Key      | Type | Default  | Description                            |
| -------- | ---- | -------- | -------------------------------------- |
| id       | Uint | `0`      | ID of vertex                           |
| pos      | Vec2 | `(0, 0)` | Position of vertex                     |
| uv       | Vec2 | `(0, 0)` | UV of vertex                           |
| init_pos | Int  | `pos`    | Helper for initial vertex position[^2] |

#### Bind

Meshes can have 'binding' bones to influence a set of vertices. These are the
primary method of animating vertices.

| Key     | Type                    | Default | Description                                  |
| ------- | ----------------------- | ------- | -------------------------------------------- |
| id      | Int                     | `-1`    | ID of bind                                   |
| is_path | Bool                    | `false` | Should this bind behave like a path?         |
| verts   | [BindVert](#bindvert)[] | `[]`    | Array of vertex data associated to this bind |

#### BindVert

Vertices assigned to a bind.

| Key    | Type  | Default | Description                    |
| ------ | ----- | ------- | ------------------------------ |
| id     | Uint  | `0`     | ID of vertex                   |
| weight | Float | `1`     | Weight assigned to this vertex |

## Physics

| Key               | Type  | Default  | Description                                                                                                           |
| ----------------- | ----- | -------- | --------------------------------------------------------------------------------------------------------------------- |
| global_pos        | Vec2  | `(0, 0)` | Bone's position based on physics. Overrides `bone.position` in [inheritance()](/generic/construct.html#inheritance)   |
| global_scale      | Vec2  | `(0, 0)` | Bone's scale based on physics. Overrides `bone.scale` in [inheritance()](/generic/construct.html#inheritance)         |
| global_rot        | Vec2  | `(0, 0)` | Bone's rotation based on physics. Overrides `bone.rotation` in [inheritance()](/generic/construct.html#inheritance)   |
| pos_damping       | Float | `0`      | Higher damping makes position interpolate slower towards `bone.position`                                              |
| scale_damping     | Float | `0`      | Higher damping makes scale interpolate slower towards `bone.scale`                                                    |
| rot_damping       | Float | `0`      | Higher damping makes rotation interpolate slower towards `bone.rotation`                                              |
| pos_ratio         | Float | `0`      | Ranges from -1 to 1. -1-0: less Y, 0-1: less X                                                                        |
| scale_ratio       | Float | `0`      | Ranges from -1 to 1. -1-0: less Y, 0-1: less X                                                                        |
| global_orbit      | Float | `0`      | Bone's parental orbit based on physics. Overrides `orbit_rot` in [inheritance()](/generic/construct.html#inheritance) |
| global_orbit_diff | Float | `0`      | Used to offset `global_orbit` by this much when swaying                                                               |
| global_orbit_vel  | Float | `0`      | Used to calculate velocity when `rot_bounce` is more than 0                                                           |
| sway              | Float | `0`      | When bone's parent is moved, how much should this bone sway?                                                          |
| rot_bounce        | Float | `0`      | Along with sway, makes bone bouncy/wiggly                                                                             |

## Animations

| Key       | Type                      | Default | Description                        |
| --------- | ------------------------- | ------- | ---------------------------------- |
| id        | String                    | `0`     | ID of animation                    |
| name      | String                    | `""`    | Name of animation                  |
| fps       | Uint                      | `0`     | Frames per second of animation     |
| keyframes | [Keyframes](#keyframes)[] | `[]`    | Data of all keyframes of animation |

### Keyframes

Keyframes are defined by their `element` (what's animated), as well as either
`value` or `value_str` (what value to animate `element` to)

Eg: `element: PosX` with `value: 20` means 'Position X = 20 at `frame`'

| Key          | Type   | Default | Description                              |
| ------------ | ------ | ------- | ---------------------------------------- |
| frame        | Uint   | `0`     | frame of keyframe                        |
| bone_id      | Uint   | `0`     | ID of bone that keyframe refers to       |
| element      | String | `""`    | Element to be animated by this keyframe  |
| value        | Float  | `0`     | Value to set `element` of bone to        |
| value_str    | String | `""`    | String variant of value                  |
| next_kf      | Int    | `-1`    | Index of the next associated keyframe    |
| start_handle | Float  | `0.333` | Handle to use for start of interpolation |
| end_handle   | Float  | `0.666` | Handle to use for end of interpolation   |

## Atlases

Easily-accessible information about texture atlas files.

| Key      | Type   | Default  | Description                 |
| -------- | ------ | -------- | --------------------------- |
| filename | String | `""`     | Name of file for this atlas |
| size     | Vec2   | `(0, 0)` | Size of image (in pixels)   |

## Styles

Groups of textures.

| Key      | Type                | Default | Description       |
| -------- | ------------------- | ------- | ----------------- |
| id       | Uint                | `0`     | ID of style       |
| name     | String              | `""`    | Name of style     |
| textures | [Texture](#texture) | `[]`    | Array of textures |

### Textures

Note: Coordinates are in pixels.

| Key       | Type   | Default  | Description                                              |
| --------- | ------ | -------- | -------------------------------------------------------- |
| name      | String | `""`     | Name of texture                                          |
| offset    | Vec2   | `(0, 0)` | Top-left corner of texture in the atlas                  |
| size      | Vec2   | `(0, 0)` | Append to `offset` to get bottom-right corner of texture |
| atlas_idx | Uint   | `0`      | Index of [atlas](#atlases) that this texture lives in    |

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

[^2]: See [Initial Fields](#initialfields)
