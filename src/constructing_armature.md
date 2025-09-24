# Constructing Armature

Once the bones have been [animated](./animating.md), they must be constructed
via inheritance and/or inverse kinematics.

## Table of Contents

- [Inheritance](#inheritance)
- [Inverse Kinematics](#inverse-kinematics)
  - [Logic](#logic)

## Inheritance

The properties of bones as laid out in `armature.json` are only local to
themselves, and do not include the parent.

Child bones must inherit their parent's properties:

```go
func inheritance(tempBones []Bone) {
   for i := range(tempBones) {
      if tempBones[i].parent_idx == -1 {
         continue
      }

      parent := tempBones[tempBones[i].parent_idx]

      tempBones[i].rot += parent.rot;

      tempBones[i].scale *= parent.scale;

      // maintain child's position from parent when scaling
      tempBones[i].pos *= parent.scale;

      // rotate child such that it will orbit the parent
      tempBones[i].pos = Vec2.new(
         tempBones[i].pos.x * Cos(parent.rot) - tempBones[i].pos.y * Sin(parent.rot),
         tempBones[i].pos.x * Sin(parent.rot) - tempBones[i].pos.y * Cos(parent.rot),
      )

      tempBones[i].pos += parent.pos;
   }
}
```

## Inverse Kinematics

If the armature contains inverse kinematics, construction is done via 3 steps:

1. Inheritance
2. Inverse kinematics
3. Inheritance (with rotations from 2nd step)

**Reasoning**:

1. Inheritance is run once to put all bones in place. Inverse kinematics
   requires this to accurately calculate the final rotations that bones will
   take on.

2. Inverse kinematics is run to calculate the new rotations that the affected
   bones will use.

3. The bones are reset, and inheritance runs again with the rotations provided
   by inverse kinematics.

### Logic

Inverse kinematics is entirely non-mutable; it only serves to return the new
rotations for bones to use in the 2nd inheritance call.

As described in the steps above, the temp bones provided must have been
constructed from forward kinematics.

The following is based on the
[FABRIK](https://www.youtube.com/watch?v=NfuO66wsuRg) technique, and iterates
through all IK families:

```go
func inverseKinematics(tempBones []Bone, ikFamilies []IkFamily) map[uint]float {
   for ikFamily, i := range(ikFamilies) {
      root := tempBones[ikFamily.boneIdxs[0]]

      // forward reaching
      nextPos := tempBones[ikFamily.targetIdx].pos
      nextLength := 0
      for idx, i := reverse(range(ikFamily.boneIdxs)) {
         bone := &tempBones[idx]

         // before moving the bone, keep the length in mind for the next bone
         length := normalize(nextPos - bone.pos) * nextLength

         if i != 0 {
            nextBone := &tempBones[ikFamily.boneIdxs[i-1]]
            nextLength = magnitude(bone.pos - nextBone.pos)
         }

         // move the bone, but maintain distance between it and previous bone
         bone.pos = nextPos - length;
      }

      // backward reaching
      prevPos := tempBones[ikFamily.boneIdxs[0]].pos;
      prevLength := 0
      for idx, i := range(ikFamily.boneIdxs) {
         bone := &tempBones[idx]

         // before moving the bone, keep the length in mind for the next bone
         length := normalize(prevPos - bone.pos) * prevLength

         if i != 0 {
            prevBone := &tempBones[ikFamily.boneIdxs[i-1]]
            prevLength = magnitude(bone.pos - prevBone.pos)
         }

         // move the bone, but maintain distance between it and previous bone
         bone.pos = prevPos - length;
      }

      rotMap := make(map[uint]float)

      // rotating bones and saving their results
      endBone := tempBones[ikFamily.boneIdxs[-1]]
      tipPos := endBone.pos
      for idx, i := range(reverse(ikFamily.boneIdxs)) {
         // don't rotate end bone
         if i == 0 {
            continue
         }

         dir := tipPos - tempBones[idx].pos
         rot := atan2(dir.y, dir.x)
         tipPos = tempBones[idx].pos

         // save the bones to a hash map, mapping the rotations to the bone's idx
         rotMap[idx] = rot
      }
   }
}
```
