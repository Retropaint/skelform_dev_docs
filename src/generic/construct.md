# `Construct()`

Constructs the armature's bones with inheritance and inverse kinematics.

```typescript
function Construct(armature: Armature): Bone[] {
    inhBones: Bone[] = clone(armature.bones)
    // inheritance is run once to put bones in place,
    // for inverse kinematics to properly determine rotations
    inheritance(inhBones, {})

    // inverse kinematics will return which bones' rotations should be overridden
    ikRots: Object = inverseKinematics(inhBones, armature.ikRootIds)

    // inheritance is run again on a fresh clone of bones, this time with the IK rotations
    finalBones: Bone[] = clone(armature.bones)
    inheritance(finalBones, ikRots)

    constructVerts(finalBones)

    return finalBones
}
```

## `inheritance()`

Child bones need to inherit their parent.

```typescript
inheritance(bones: Bone[], ikRots: Object) {
    for(let b = 0; b < bones.length; b++) {
        if(bones[b].parentId != -1) {
            parent: Bone = clone(bones[bones[b].parentId]);

            bones[b].rot += parent.rot
            bones[b].scale *= parent.scale

            // adjust child's distance from player as it gets bigger/smaller
            bones[b].pos *= parent.scale

            // rotate child around parent as if it were orbitting
            bones[b].pos = rotate(&bones[b].pos, parent.rot)

            bones[b].pos += parent.pos
        }

        // override bone's rotation from inverse kinematics
        if ikRots[b] {
            bones[b].rot = ikRots[b]
        }
    }
}
```

## `rotate()`

Helper for rotating a Vec2.

```typescript
function rotate(point: Vec2, rot: f32): Vec2 {
    return Vec2 {
        x: point.x * rot.cos() - point.y * rot.sin(),
        y: point.x * rot.sin() + point.y * rot.cos(),
    }
}
```

## `inverseKinematics()`

Processes inverse kinematics and returns the final bones' rotations, which would
later be used by `inheritance()`.

IK data for each set of bones is stored in the root bone, which can be iterated
wth `ikRootIds`.

```typescript
function inverseKinematics(bones: Bone[], ikRootIds: int[]): Object {
    ikRot: Object = {}

    for(let rootId of ikRootIds) {
        family: Bone[] = clone(bones[rootId])

        // get relevant bones from the same set
        if(family.ikTargetId == - 1) {
            continue
        }
        root: Vec2 = bones[family.ikBoneIds[0]].pos
        target: Vec2 = bones[family.ikTargetId].pos
        familyBones: Bone[] = bones.filter(|bone|
            family.ikBoneIds.contains(bone.id)
        )

        // determine which IK mode to use
        switch(family.ikMode) {
            case 0:
                for range(10) {
                    fabrik(*familyBones, root, target)
                }
            case 1:
                arcIk(*familyBones, root, target)
        }

        pointBones(*bones, family)

        applyConstraints(*bones, family)

        // add rotations to ikRot, with bone ID being the key
        for(let b = 0; b < family.ikBoneIds.length; b++) {
            // last bone of IK should have free rotation
            if(b == family.ikBoneIds.len() - 1) {
                continue
            }
            ikRot[family.ikBoneIds[b]] = bones[family.ikBoneIds[b]].rot
        }
    }

    return ikRot
}
```

## `pointBones()`

Point each bone toward the next one.

Used by `inverseKinematics()` to get the final bone's rotations.

```typescript
function pointBones(bones: Bone[]*, family: Bone) {
    endBone: Bone = bones[family.ikBoneIds[-1]]
    tipPos: Vec2 = endBone.pos
    for(let i = family.ikBoneIds.length - 1; i > 0; i--) {
        bone = *bones[family.ikBoneIds[i]]
        dir: Vec2 = tipPos - bone.pos
        bone.rot = atan2(dir.y, dir.x)
        tipPos = bone.pos
    }
}
```

## `applyConstraints()`

Applies constraints to bone rotations (clockwise or counter-clockwise).

1. Get angle of first joint
2. Get angle from root to target
3. Compare against the 2 based on the constraint
4. If the constraint is satisfied, apply `rot + baseAngle * 2` to bone rotation

```typescript
function applyConstraints(bones: Bone[], family: Bone) {
    jointDir:  Vec2 = normalize(bones[family.ikBoneIds[1]].pos - root)
    baseDir:   Vec2 = normalize(target - root)
    dir:       float = jointDir.x * baseDir.y - baseDir.x * jointDir.y
    baseAngle: float = atan2(baseDir.y, baseDir.x)
    cw:        bool = family.ikConstraint == 1 && dir > 0.
    ccw:       bool = family.ikConstraint == 2 && dir < 0.
    if(cww || cw) {
        for(let id of family.ikBoneIds) {
            bones[id].rot = -bones[id].rot + baseAngle * 2.
        }
    }
}
```

## `fabrik()`

The FABRIK mode (Forward And Backward Reaching Inverse Kinematics).

Note that this should be run multiple times for higher accuracy (usually 10
times).

Source for algorithm:
[Programming Chaos' FABRIK video](https://www.youtube.com/watch?v=NfuO66wsuRg)

```typescript
function fabrik(bones: Bone[], root: Vec2, target: Vec2) {
    // forward-reaching
    nextPos: Vec2 = target
    nextLength: float = 0.0
    for(let b = bones.length - 1; b > 0; b--) {
        length: Vec2 = normalize(nextPos - bones[b].pos) * nextLength
        if(isNaN(length))
            length = new Vec2(0, 0)
        if(b != 0)
            nextLength = magnitude(bones[b].pos - bones[b - 1].pos)
        bones[b].pos = nextPos - length
        nextPos = bones[b].pos
    }

    // backward-reaching
    prevPos: Vec2 = root
    prevLength: float = 0.0
    for(let b = 0; b < bones.length; b++) {
        length: Vec2 = normalize(prevPos - bones[b].pos) * prevLength
        if(isNaN(length))
            length = new Vec2(0, 0)
        if(b != bones.len() - 1)
            prevLength = magnitude(bones[b].pos - bones[b + 1].pos)
        bones[b].pos = prevPos - length
        prevPos = bones[b].pos
    }
}
```

## `arcIk()`

Arcing IK mode.

Bones are positioned like a bending arch, with the max length being the combined
distance of each bone after the other.

```typescript
function arcIk(bones: Bone[], root: Vec2, target: Vec2) {
    // determine where bones will be on the arc line (ranging from 0 to 1)
    dist: float[] = [0.]

    maxLength: Vec2 = magnitude(bones.last().pos - root)
    currLength: float = 0.
    for(let b = 1; b < bones.length; b++) {
        length: float = magnitude(bones[b].pos - bones[b - 1].pos)
        currLength += length;
        dist.push(currLength / maxLength)
    }

    base: Vec2 = target - root
    baseAngle: float = base.y.atan2(base.x)
    baseMag: float = magnitude(base).min(maxLength)
    peak: float = maxLength / baseMag
    valley: float = baseMag / maxLength
    for(let b = 1; b < bones.length; b++) {
        bones[b].pos = new Vec2(
            bones[b].pos.x * valley,
            root.y + (1.0 - peak) * sin(dist[b] * PI*2) * baseMag,
        )

        rotated: float = rotate(bones[b].pos - root, baseAngle)
        bones[b].pos = rotated + root
    }
}
```

## `ConstructVerts()`

Constructs vertices, for bones with mesh data.

```typescript
function constructVerts(bones: Bone[]) {
    for(let b = 0; b < bones.length; b++) {
        bone: Bone = clone(bones[b])

        // Move vertex to main bone.
        // This will be overridden if vertex has a bind.
        for(let v = 0; v < bone.vertices.length; v++) {
            bone.vertices[v] = inheritVert(bone.vertices[v].pos, bone)
        }

        for(let bi = 0; bi < bones[b].binds.length; bi++) {
            let boneId = bones[b].binds[bi].boneId
            if boneId == -1 {
                continue
            }
            bindBone: Bone = clone(bones.find(|bone| bone.id == bId)))
            bind: Bind = clone(bones[b].binds[bi])
            for(let v = 0; v < bind.verts.length; v++) {
                id: int = bind.verts[v].id

                if !bind.isPath {
                    // weights
                    vert: Vertex = bones[b].vertices[id]
                    weight: float = bind.verts[v].weight
                    endpos: Vec2 = inheritVert(vert.initPos, bindBone) - vert.pos
                    vert.pos += endPos * weight
                    continue
                }

                // pathing

                // Check out the 'Pathing Explained' section below for a
                // comprehensive explanation.

                // 1.
                // get previous and next bind
                binds: Bind[] = bones[b].binds
                prev: int = bi > 0 ? bi - 1 : bi
                next: int = (bi + 1).min(binds.len() - 1)
                prevBone: Bone = bones.find(|bone| bone.id == binds[prev].boneId)
                nextBone: Bone = bones.find(|bone| bone.id == binds[next].boneId)

                // 2.
                // get the average of normals between previous and next bind
                prevDir: Vec2 = bindBone.pos - prevBone.pos
                nextDir: Vec2 = nextBone.pos - bindBone.pos
                prevNormal: Vec2 = normalize(Vec2::new(-prevDir.y, prevDir.x))
                nextNormal: Vec2 = normalize(Vec2::new(-nextDir.y, nextDir.x))
                average: Vec2 = prevNormal + nextNormal
                normalAngle: float = atan2(average.y, average.x)

                // 3.
                // move vert to bind, then rotate it around bind by normalAngle
                vert: Vertex = bones[b].vertices[id]
                vert.pos = vert.initPos + bindBone.pos
                rotated: Vec2 = rotate(vert.pos - bindBone.pos, normalAngle)
                vert.pos = bindBone.pos + (rotated * bind.verts[v].weight)
                bones[b].vertices[id] = vert
            }
        }
    }
}
```

## Pathing Explained

Instead of inheriting binds directly, vertices can be set to follow its bind
like a line forming a path:

<img src="pathing.png" alt="pathing example" width="300"/>

- Green - bind bone
- Orange - vertices
- Red - imaginary line from bind to bind
- Blue - Normal surface of imaginary line

Vertices will follow the path, distancing from the bind based on its surface
angle and initial position from vertex to bind.

The following steps can be iterated per bind:

### 1. Get Adjacent Binds

To form the imaginary line, get the adjacent binds just before and just after
the current bind. In particular:

- If current bind is first: get only next bind
- If current bind is last: get only previous bind
- If current bind is neither: get both previous and next bind

### 2. Get Average Normal Angle

Notice that in the diagram, the middle bind's surface is at a 45° angle.

To do so:

1. Get line from previous to current bind
2. Get line from current to next bind
3. Add up both lines
4. Get angle of combined line

### 3. Rotate Vertices

1. Reset vertex position to it's initial position + bind position
2. Rotate vertex around bind with angle from 2nd step
