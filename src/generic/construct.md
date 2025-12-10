# `Construct()`

Constructs the armature's bones with inheritance and inverse kinematics.

```c
Construct(armature: Armature): Bone[] {
    inhBones: Bone[] = clone(armature.bones)
    // inheritance is run once to put bones in place,
    // for inverse kinematics to properly determine rotations
    inheritance(*inhBones, HashMap::new())

    // inverse kinematics will return which bones' rotations should be overridden
    ikRots: Map<int, float> = inverseKinematics(*inhBones, armature.ikRootIds)

    // inheritance is run again on a fresh clone of bones, this time with the IK rotations
    finalBones: Bone[] = clone(armature.bones)
    inheritance(*finalBones, ikRots)

    constructVerts(*finalBones)

    return finalBones
}
```

## `inheritance()`

Child bones need to inherit their parent.

```c
inheritance(bones: Bone[]*, ikRots: HashMap<i32, f32>) {
    for b in range(bones) {
        if bones[b].parentId != -1 {
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
        if ikRots.get(b) != None {
            bones[b].rot = ikRots.get(b)
        }
    }
}
```

## `rotate()`

Helper for rotating a Vec2.

```c
rotate(point: Vec2, rot: f32): Vec2 {
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

```c
inverseKinematics(bones: Bone[]*, ikRootIds: int[]): HashMap<int, float> {
    ikRot: HashMap<i32, f32> = HashMap::new()

    for rootId in ikRootIds {
        family: Bone[] = clone(bones[rootId])

        // get relevant bones from the same set
        if family.ikTargetId == -1 {
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
        for b in range(family.ikBoneIds) {
            // last bone of IK should have free rotation
            if b == family.ikBoneIds.len() - 1 {
                continue
            }
            ikRot.push(<family.ikBoneIds[b], bones[family.ikBoneIds[b]].rot>)
        }
    }

    return ikRot
}
```

## `pointBones()`

Point each bone toward the next one.

Used by `inverseKinematics()` to get the final bone's rotations.

```c
pointBones(bones: Bone[]*, family: Bone) {
    endBone: Bone = bones[*family.ikBoneIds.last()]
    tipPos: Vec2 = endBone.pos
    for i in range(family.ikBoneIds).reverse() {
        if i == family.ikBoneIds.len() - 1 {
            continue;
        }
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

```c
applyConstraints(bones: Bone[]*, family: Bone) {
    jointDir:  Vec2 = normalize(bones[family.ikBoneIds[1]].pos - root)
    baseDir:   Vec2 = normalize(target - root)
    dir:       float = jointDir.x * baseDir.y - baseDir.x * jointDir.y
    baseAngle: float = atan2(baseDir.y, baseDir.x)
    cw:        bool = family.ikConstraint == 1 && dir > 0.
    ccw:       bool = family.ikConstraint == 2 && dir < 0.
    if ccw || cw {
        for i in family.ikBoneIds {
            bones[i].rot = -bones[i].rot + baseAngle * 2.
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

```c
fabrik(bones: Bone[]*, root: Vec2, target: Vec2) {
    // forward-reaching
    nextPos: Vec2 = target
    nextLength: float = 0.
    for b in range(bones).reverse() {
        length: Vec2 = normalize(nextPos - bones[b].pos) * nextLength
        if isNaN(length) {
            length = Vec2::new(0., 0.)
        }
        if b != 0 {
            nextLength = magnitude(bones[b].pos - bones[b - 1].pos)
        }
        bones[b].pos = nextPos - length
        nextPos = bones[b].pos
    }

    // backward-reaching
    prevPos: Vec2 = root
    prevLength: float = 0.
    for b in range(bones) {
        length: Vec2 = normalize(prevPos - bones[b].pos) * prevLength;
        if isNaN(length) {
            length = Vec2::new(0., 0.)
        }
        if b != bones.len() - 1 {
            prevLength = magnitude(bones[b].pos - bones[b + 1].pos)
        }
        bones[b].pos = prevPos - length
        prevPos = bones[b].pos
    }
}
```

## `arcIk()`

Arcing IK mode.

Bones are positioned like a bending arch, with the max length being the combined
distance of each bone after the other.

```c
arcIk(bones: Bone[]*, root: Vec2, target: Vec2) {
    // determine where bones will be on the arc line (ranging from 0 to 1)
    dist: float[] = [0.]

    maxLength: Vec2 = magnitude(bones.last().pos - root)
    currLength: float = 0.
    for b in 1..bones.len() {
        length: float = magnitude(bones[b].pos - bones[b - 1].pos)
        currLength += length;
        dist.push(currLength / maxLength)
    }

    base: Vec2 = target - root
    baseAngle: float = base.y.atan2(base.x)
    baseMag: float = magnitude(base).min(maxLength)
    peak: float = maxLength / baseMag
    valley: float = baseMag / maxLength

    for b in 1..bones.len() {
        bones[b].pos = Vec2::new(
            bones[b].pos.x * valley,
            root.y + (1. - peak) * sin(dist[b] * 3.14) * baseMag,
        );

        rotated: float = rotate(bones[b].pos - root, baseAngle)
        bones[b].pos = rotated + root
    }
}
```

## `ConstructVerts()`

Constructs vertices, for bones with mesh data.

```c
constructVerts(bones: *Bone[]) {
    for b in range(bones) {
        bone: Bone = clone(bones[b])

        // Move vertex to main bone.
        // This will be overridden if vertex has a bind.
        for vert in bones.vertices {
            vert.pos = inheritVert(vert.pos, bone)
        }

        for bi in range(bones[b].binds) {
            let boneId = bones[b].binds[bi].boneId
            if boneId == -1 {
                continue
            }
            bindBone: Bone = clone(bones.find(|bone| bone.id == bId)))
            bind: Bind = clone(bones[b].binds[bi])
            for v in 0..bind.verts.len() {
                id: int = bind.verts[v].id

                if !bind.isPath {
                    // weights
                    vert: Vertex = bones[b].vertices[id]
                    weight: float = bind.verts[v].weight
                    endpos: Vec2 = inheritVert(vert.initPos, bindBone) - vert.pos
                    vert.pos += endPos * weight
                    continue
                }

                // pathing:
                // Bone binds are treated as one continuous line.
                // Vertices will follow along this path.

                // get previous and next bone
                binds: Bind[] = bones[b].binds
                prev: int = if bi > 0 { bi - 1 } else { bi }
                next: int = (bi + 1).min(binds.len() - 1)
                prevBone: Bone = bones.find(|bone| bone.id == binds[prev].boneId)
                nextBone: Bone = bones.find(|bone| bone.id == binds[next].boneId)

                // get the average of normals between previous bone,
                // this bone, and next bone
                prevDir: Vec2 = bindBone.pos - prevBone.pos
                nextDir: Vec2 = nextBone.pos - bindBone.pos
                prevNormal: Vec2 = normalize(Vec2::new(-prevDir.y, prevDir.x))
                nextNormal: Vec2 = normalize(Vec2::new(-nextDir.y, nextDir.x))
                average: Vec2 = prevNormal + nextNormal
                normalAngle: float = atan2(average.y, average.x)

                // move vertex to bind bone, then just adjust it to
                // 'bounce' off the line's surface
                vert: Vertex = *bones[b].vertices[id]
                vert.pos = vert.initPos + bindBone.pos
                rotated: Vec2 = rotate(vert.pos - bindBone.pos, normalAngle)
                vert.pos = bindBone.pos + (rotated * bind.verts[v].weight)
            }
        }
    }
}
```
