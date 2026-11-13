# Changelog (v0.8 -> v0.9)

## Name-based animations

Animation keyframes now store `bone_name` instead of `bone_id`.

- `Animate()` now identifies bones by name (`kf.bone_name`) instead of ID
- `resetBones()` now identifies bones by name instead of ID

## Rotation-based Inheritance Scaling

Parent scaling in inheritance used to not take rotations into account, which
would lead to scenarios where children are being stretched in directions they
shouldn't be.

- Added `inheritedScale` to `Construct()` (Generic)
- `inheritedScale` is explicitly multiplied if bone's scale is negative

## Proper Pivot Positions

- Adjusted `pivotPos` so only `bone.scale` is multiplied _after_ rotation, in
  `Draw()` (Engine)

## No pivots for meshes

For simplicity reasons, meshes do not need to account for the bone's pivot.
Handling pivots for meshes is essentially undcoumentable due to it being
different from engine to engine. As of v0.9, the editor will reset and prevent
the user from editing pivot data on mesh bones.
