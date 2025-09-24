# Constructing Armature

Before animations can take place, it is best to nail down the construction
process of the armature.

## Table of Contents

- [Non-Mutability](#non-mutability)
- [Inheritance](#inheritance)
- [Inverse Kinematics](#inverse-kinematics)

## Non-Mutability

It is recommended that the original bones supplied to construction are not
modified. Instead, construction should work with cloned bones and return them to
the consumer.

This does not apply to generic runtimes, as the supplied bones (coming from an
engine runtime) are expected to be cloned already.

### Reasoning:

Runtimes should ideally be deterministic. By mutating the original bones,
animations may behave differently depending on what played before it.

## Inheritance

The properties of bones as laid out in `armature.json` is only local to
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

### Reasoning:

1. Inheritance is run once to put all bones in place. This is needed as inverse
   kinematics will require bones with their world properties.

2. Inverse kinematics is run to calculate the new rotations that the affected
   bones will use.

3. The bones are reset, and inheritance runs again with the rotations provided
   by inverse kinematics.

### Logic

Inverse kinematics is entirely non-mutable; it only serves to return the new
rotations for bones to use in the 2nd inheritance call.

As described in the steps above, the temp bones provided must have been constructed
from forward kinematics.

The following is based on the
[FABRIK](https://www.youtube.com/watch?v=NfuO66wsuRg) technique, and iterates
through all IK families:

```go
func inverseKinematics(tempBones []Bone, ikFamilies []IkFamily) {
   for i := range(ikFamilies) {
      ik_family := ikFamilies[i]

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
            prevtLength = magnitude(bone.pos - prevBone.pos)
         }

         // move the bone, but maintain distance between it and previous bone
         bone.pos = prevPos - length;
      }
   }
}
```
