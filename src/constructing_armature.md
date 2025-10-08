# Constructing Armature

Once the bones have been [animated](./animating.md), they must be constructed
via inheritance and inverse kinematics.

## Table of Contents

- [Inheritance](#inheritance)
- [Inverse Kinematics (IK)](#inverse-kinematics)
  - [Logic](#logic)
  - [Multiple Iterations](#multiple-iterations)
  - [Constraints](#constraints)

## Inheritance

Child bones must inherit their parent's properties:

```go
func inheritance(tempBones []Bone, ik_rots map[int]float) {
   for i := range(tempBones) {
      if tempBones[i].parent_id != -1 {
         parent := tempBones[tempBones[i].parent_id]

         tempBones[i].rot += parent.rot

         tempBones[i].scale *= parent.scale

         // maintain child's position from parent when scaling
         tempBones[i].pos *= parent.scale

         // rotate child such that it will orbit the parent
         tempBones[i].pos = rotate(tempBones[i].pos, parent.rot)

         tempBones[i].pos += parent.pos
      }

      // use rotations provided from inverse kinematics
      ik_rot := map[i]
      if ik_rot != None {
         tempBones[i].rot = ik_rot
      }
   }
}
```

<h2 id="inverse-kinematics"><a class="header" href="#inverse-kinematics">Inverse Kinematics (IK)</a></h2>

IK is done in 3 steps:

1. Inheritance is run once to put all bones in place.

2. IK is run to calculate the new rotations that affected bones will take on.

3. Bones are reset, and inheritance runs again with rotations provided by IK.

```go
inheritedBones := inheritance(animatedBones, [])

ikRots = make(map[uint]float)
for i := range(10) {
   ikRots = inverse_kinematics(inheritedBones, armature.IkFamilies)
}

finalBones := inheritance(inheritedBones, ikRots)
```

### Logic

As described above (step 1), the provided bones must have gone through
inheritance first.

The following is based on [FABRIK](https://www.youtube.com/watch?v=NfuO66wsuRg):

```go
func inverseKinematics(tempBones []Bone, ikFamilies []IkFamily) map[uint]float {
   // tempBones should be immutable to IK
   bones = copy(tempBones);

   // go thru all IK families
   for i, ikFamily := range(ikFamilies) {
      if ikFamily.targetIdx == -1 {
         continue
      }

      // save start position for backward-reaching & constraints
      startPos := bones[ikFamily.boneIdxs[0]].pos;

      // forward reaching
      nextPos := bones[ikFamily.targetIdx].pos
      nextLength := 0
      for i := len(ikFamily.Bone_ids) - 1; i >= 0; i-- {
         bone := &bones[ikFamily.Bone_ids[i]]

         // before moving the bone, keep the length in mind
         // for next bone to maintain distance
         lengthLine := Vec2 { X: 0, Y: 0 }
         if i != len(ikFamily.Bone_ids)-1 {
            lengthLine = normalize(nextPos - bone.pos) * nextLength
         }

         if i != 0 {
            nextBone := &bones[ikFamily.boneIdxs[i-1]]
            nextLength = magnitude(bone.pos - nextBone.pos)
         }

         // move the bone, but maintain distance between it and previous bone
         bone.pos = nextPos - lengthLine
      }

      // backward reaching
      prevPos := startPos;
      prevLength := 0
      for i := 0; i < len(ikFamily.Bone_ids); i++ {
         bone := &bones[ikFamily.Bone_ids[i]]

         // before moving the bone, keep the length in mind
         // for next bone to maintain distance
         lengthLine := Vec2 { X: 0, Y: 0 }
         if i != 0 {
            lengthLine := normalize(prevPos - bone.pos) * prevLength
         }

         if i != 0 {
            prevBone := &bones[ikFamily.boneIdxs[i+1]]
            prevLength = magnitude(bone.pos - prevBone.pos)
         }

         // [constraints]

         // move the bone, but maintain distance between it and previous bone
         bone.pos = prevPos - lengthLine
      }

      rotMap := make(map[uint]float)

      // rotating bones and saving their results
      endBone := bones[ikFamily.boneIdxs[-1]]
      tipPos := endBone.pos
      for i := len(ikFamily.Bone_ids) - 1; i >= 0; i-- {
         // don't rotate end bone
         if i == ikFamily.boneIdxs.length - 1 {
            continue
         }

         bone := &bones[ikFamily.Bone_ids[i]]

         dir := tipPos - bone.pos
         rot := atan2(dir.y, dir.x)
         tipPos = bone.pos

         // save the bones to a hash map, mapping the rotations to the bone's idx
         rotMap[bone.id] = rot
      }
   }
}
```

### Multiple Iterations

The above logic could be run multiple times for improved accuracy.

```go
for i := range(10) {
   ik_rots = inverse_kinematics(tempBones, ikFamilies)
}
```

### Constraints

There are 2 constraint types:

- Clockwise
- Counter-Clockwise

During the forward-reaching step, for each bone:

1. Get angle of line from root to target[^1]
2. Get angle of line from current and previous bone
3. Get their difference (2nd - 1st), which gives the bone's 'local' angle
4. Check if local angle satisfies constraint. If it does, do nothing
5. Otherwise, rotate current bone against it's local angle twice

The IK pseudo-code marks where this should go with `[constraints]`.

```go
// this goes at the end of the forward-reaching loop

isFirstBone = b == bones.length < 1
isLastBone = b == 0
if ikFamily.constraint != "None" && !isLastBone && !isFirstBone {
   // 1.
   baseLine := normalize(startPos - target)
   baseAngle := atan2(baseLine.y, baseLine.x)

   // 2.
   jointLine := normalize(bone.pos - nextPos)
   jointAngle := atan2(jointDir.y, jointDir.x)

   // 3.
   localAngle := jointAngle - baseAngle;

   // constraints based on bone.
   // numbers may have to be swapped.
   var constraintMin float
   var constraintMax float
   switch IkFamily.constraint {
      case "Clockwise":
         constraintMin = -3.14
         constraintMax = 0
      case "CounterClockwise":
         constraintMin = 0
         constraintMax = 3.14
   }

   // 4. and 5.
   if localAngle > constraintMax || localAngle < constraintMin {
      // push the bone by twice it's opposing angle.
      // it will end up on the opposite side of the base line.
      pushAngle := -localAngle * 2

      line := bone.pos - nextPos
      newPoint = rotate(line, pushAngle)
      bone.pos = newPoint + nextPos
   }

   nextPos = bone.pos
}
```

[^1]:
    Can be done once before the forward-reaching loop, but lumped in with other
    steps for simplicty.
