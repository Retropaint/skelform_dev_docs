## Table of Contents

- [`Construct()`](#construct)
    - [`inheritance()`](#inheritance)
    - [`resetInheritance()`](#resetinheritance)
    - [`rotate()`](#rotate)
    - [`inverseKinematics()`](#inversekinematics)
        - [`pointBones()`](#pointbones)
        - [`applyConstraints()`](#applyconstraints)
        - [`fabrik()`](#fabrik)
        - [`arcIk()`](#arcik)
    - [`simulatePhysics()`](#simulatephysics)
    - [`constructVerts()`](#constructverts)
        - [Pathing Explained](#pathing-explained)

# `Construct()`

Constructs the armature's bones with inheritance and inverse kinematics.

```typescript
function Construct(armature: Armature) {
    let constBones = armature.constructed_bones;

    // initialize constructed_bones
    if (constBones == undefined) {
        constBones = clone(armature.bones);
    } else {
        // constructed_bones may have been used later for drawing
        // which sorts them by zindex, so sort back by id
        constBones.sort((bone) => bone.id);
    }

    // 1st inheritance pass
    resetInheritance(constBones, armature.bones);
    inheritance(constBones, {}, []);

    // 2nd inheritance pass: inverse kinematics
    if (armature.inverse_kinematics.length > 0) {
        let ikRots = inverseKinematics(constBones, armature.inverse_kinematics);
        resetInheritance(constBones, armature.bones);
        inheritance(constBones, ikRots, []);
    }

    // 3rd inheritance pass: physics
    if (armature.physics.length > 0) {
        simulatePhysics(constBones, armature.physics);
        resetInheritance(aramture.constructed_bones, armature.bones);
        inheritance(constBones, ikRots, armature.physics);
    }

    // mesh deformation
    constructVerts(constBones, armature.visuals);
}
```

## `inheritance()`

Child bones need to inherit their parent.

```typescript
function inheritance(bones: Bone[], ikRots: Object, physics: Physics[]) {
    for (let b = 0; b < bones.length; b++) {
        if (bones[b].parent_id != -1) {
            const parent: Bone = bones[bones[b].parent_id];

            let orbitRot = bones[bones[b].parent_id as usize].rot;

            // apply orbital difference, if rotation resistance physics is active
            let phys = physics[bones[b].physics_id];
            if (phys != undefined && phys.sway > 0) {
                orbitRot -= phys.global_orbit_diff;
            }
            bones[b].rot += orbitRot;

            bones[b].scale *= parent.scale;

            // adjust child's distance from player as it gets bigger/smaller
            bones[b].pos *= parent.scale;

            // rotate child around parent as if it were orbitting
            bones[b].pos = rotate(bones[b].pos, parent.rot);

            bones[b].pos += parent.pos;
        }

        // override bone's rotation from inverse kinematics
        if (ikRots.get(bones[b].id)) {
            bones[b].rot = ikRots.get(bones[b].id);
        }

        // apply physics, if armature_bones is provided
        let phys = physics[bones[b].physics_id];
        if (phys != undefined) {
            if (phys.rot_damping > 0) {
                bones[b].rot = phys.global_rot;
            }
            if (phys.pos_damping > 0) {
                bones[b].pos = phys.global_pos;
            }
            if (phys.scale_damping > 0) {
                bones[b].scale = phys.global_scale;
            }
        }
    }
}
```

## `resetInheritance()`

Resets the provided `constructed_bones` to their original transforms.

Must always be called before `inheritance()`.

```typescript
resetInheritance(constructedBones: Bone[], bones: Bone[]) {
    for(let b = 0; b < bones.length; b++) {
        constructedBones[b].pos = bones[b].pos;
        constructedBones[b].rot = bones[b].rot;
        constructedBones[b].scale = bones[b].scale;
    }
}
```

## `rotate()`

Helper for rotating a Vec2.

```typescript
function rotate(point: Vec2, rot: float): Vec2 {
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
wth `inverse_kinematics`.

```typescript
function inverseKinematics(
    bones: Bone[],
    inverseKinematics: InverseKinematics[],
): Map<Int, Float> {
    let ikRots = new Map<Int, Float>();

    for (let family of inverseKinematics) {
        // get relevant bones from the same set
        if (family.target_id == -1) {
            continue;
        }
        const root: Vec2 = bones[family.bone_ids[0]].pos;
        const target: Vec2 = bones[family.target_id].pos;
        let familyBones: Bone[] = bones.filter((bone) =>
            family.bone_ids.contains(bone.id),
        );

        // determine which IK mode to use
        switch (family.mode) {
            case "FABRIK":
                for (let i = 0; i < 10; i++) {
                    fabrik(familyBones, root, target);
                }
            case "Arc":
                arcIk(familyBones, root, target);
        }

        pointBones(bones, family);
        applyConstraints(bones, family);

        // add rotations to ikRot, with bone ID being the key
        for (let b = 0; b < family.bone_ids.length; b++) {
            // last bone of IK should have free rotation
            if (b == family.bone_ids.length - 1) {
                continue;
            }
            ikRots.set(family.bone_ids[b].id, bones[family.bone_ids[b]].rot);
        }
    }

    return ikRots;
}
```

## `pointBones()`

Point each bone toward the next one.

Used by `inverseKinematics()` to get the final bone's rotations.

```typescript
function pointBones(bones: Bone[], family: InverseKinematics) {
    let endBone: Bone = bones[family.bone_ids[-1]];
    let tipPos: Vec2 = endBone.pos;
    for (let i = family.bone_ids.length - 1; i > 0; i--) {
        const bone = bones[family.bone_ids[i]];
        if (i == family.bone_ids.length - 1) {
            // end bone should follow target bone rotation, if mimic_target is true
            if (family.mimic_target) {
                bone.rot = bones[family.target_id].rot;
            }
            continue;
        }
        const dir: Vec2 = tipPos - bone.pos;
        bone.rot = atan2(dir.y, dir.x);
        tipPos = bone.pos;
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
function applyConstraints(bones: Bone[], family: InverseKinematics) {
    const jointDir: Vec2 = normalize(bones[family.bone_ids[1]].pos - root);
    const baseDir: Vec2 = normalize(target - root);
    const dir: Float = jointDir.x * baseDir.y - baseDir.x * jointDir.y;
    const baseAngle: Float = atan2(baseDir.y, baseDir.x);
    const cw: Bool = family.constraint == "Clockwise" && dir > 0;
    const ccw: Bool = family.constraint == "CounterClockwise" && dir < 0;
    if (ccw || cw) {
        family.bone_ids.forEach((id) => {
            bones[id].rot = -bones[id].rot + baseAngle * 2;
        });
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
    let nextPos: Vec2 = target;
    let nextLength: Float = 0.0;
    for (let b = bones.length - 1; b > 0; b--) {
        let length: Vec2 = normalize(nextPos - bones[b].pos) * nextLength;
        if (isNaN(length)) length = Vec2(0, 0);
        if (b != 0) nextLength = magnitude(bones[b].pos - bones[b - 1].pos);
        bones[b].pos = nextPos - length;
        nextPos = bones[b].pos;
    }

    // backward-reaching
    let prevPos: Vec2 = root;
    let prevLength: Float = 0.0;
    for (let b = 0; b < bones.length; b++) {
        let length: Vec2 = normalize(prevPos - bones[b].pos) * prevLength;
        if (isNaN(length)) length = Vec2(0, 0);
        if (b != bones.length - 1)
            prevLength = magnitude(bones[b].pos - bones[b + 1].pos);
        bones[b].pos = prevPos - length;
        prevPos = bones[b].pos;
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
    let dist: Float[] = [0];

    const maxLength: Vec2 = magnitude(bones[-1].pos - root);
    let currLength: Float = 0;
    for (let b = 1; b < bones.length; b++) {
        const length: Float = magnitude(bones[b].pos - bones[b - 1].pos);
        currLength += length;
        dist.push(currLength / maxLength);
    }

    const base: Vec2 = target - root;
    const baseAngle: Float = base.y.atan2(base.x);
    const baseMag: Float = magnitude(base).min(maxLength);
    const peak: Float = maxLength / baseMag;
    const valley: Float = baseMag / maxLength;
    for (let b = 1; b < bones.length; b++) {
        bones[b].pos = new Vec2(
            bones[b].pos.x * valley,
            root.y + (1.0 - peak) * sin(dist[b] * PI) * baseMag,
        );

        const rotated: Float = rotate(bones[b].pos - root, baseAngle);
        bones[b].pos = rotated + root;
    }
}
```

## `simulatePhysics()`

Processes all physics:

- Position (`physics.pos_damping`)
- Scale (`physics.scale_damping`)
- Rotation (`physics.rot_damping`)
- Sway (`physics.sway`)
- Bounce (`physics.rot_bounce`)

```typescript
function simulatePhysics(constructedBones: Bone[], physics: Physics[]) {
    for(let b = 0; b < constructed_bones.length; b++) {
        if constructedBones[b].physics_id == -1 {
            continue;
        }
        let physics = physics[constructedBones[b].physics_id as usize];

        const s = Vec2(0.3, 0.3);
        const e = Vec2(0.6, 0.6);
        const constBone = constructedBones[b];
        const prevPos = physics.global_pos;

        // interpolate position
        if(physics.pos_damping > 0 || physics.sway > 0) {
            let damping = Vec2(physics.pos_damping, physics.pos_damping)

            // ratio
            if(physics.pos_ratio < 0) {
                damping.y *= 1. - Math.abs(physics.pos_ratio);
            } else if(physics.pos_ratio > 0) {
                damping.x *= 1. - physics.pos_ratio;
            }

            let phys_pos = physics.global_pos;
            phys_pos.x = interpolate(2, damping.x, phys_pos.x, constBone.pos.x, s, e);
            phys_pos.y = interpolate(2, damping.y, phys_pos.y, constBone.pos.y, s, e);
        }

        // interpolate scale
        if(physics.scale_damping > 0) {
            let damping = Vec2(physics.scale_damping, physics.scale_damping);

            // ratio
            if(physics.scale_ratio < 0) {
                damping.y *= 1. - Math.abs(physics.scale_ratio)
            } else if(physics.pos_ratio > 0) {
                damping.x *= 1. - physics.scale_ratio
            }

            let physScale = physics.global_scale;
            physScale.x = interpolate(2, damping.x, physScale.x, constBone.scale.x, s, e);
            physScale.y = interpolate(2, damping.y, physScale.y, constBone.scale.y, s, e);
        }

        // interpolate rotation
        if(physics.rot_damping > 0) {
            const rot = shortest_angle_delta(physics.global_rot, constBone.rot)
            physics.global_rot += rot / physics.rot_damping
        }

        // interpolate parent orbit (rot res, bounce, etc)
        const parent = constructed_bones.find(b => b.id == constBone.parent_id)
        if(physics.sway > 0 && parent != undefined) {
            // 1. get the raw orbit angle between this bone and its parent
            const diff = normalize(constBone.pos - parent.pos)
            const diffAngle = Math.atan2(diff.y, diff.x)

            // 2. interpolate current orbit angle to raw angle
            let orbitBuffer = shortest_angle_delta(physics.global_orbit, diffAngle)

            // 3. apply bounce to orbit angle
            if(physics.rot_bounce > 0 && physics.rot_bounce <= 1) {
                orbitBuffer += physics.global_orbit_vel / (2 - physics.rot_bounce)
                physics.global_orbit_vel = orbitBuffer
            }

            // 4. apply orbit buffer
            physics.global_orbit += orbitBuffer / 10

            // 5. swing orbit based on position momentum
            const vel = normalize(physics.global_pos - prevPos)
            const angle = Math.atan2(-vel.y, -vel.x)
            const velRot = shortest_angle_delta(physics.global_orbit, angle)
            const strength = magnitude(physics.global_pos - prevPos) / 1000
            physics.global_orbit += velRot * strength * physics.sway

            // 6. apply difference in raw angle and orbit
            physics.global_orbit_diff = diffAngle - physics.global_orbit
        }
    }
}
```

## `constructVerts()`

Constructs vertices, for bones with mesh data.

Note: a helper function (`inheritVert()`) is included in the code block below

```typescript
function constructVerts(bones: Bone[], visuals: Visuals[]) {
    for(let b = 0; b < bones.length; b++) {
        if bones[b].visuals_id == -1 {
            continue;
        }
        let visual = visuals[bones[b].visuals_id];

        const bone: Bone = bones[b]

        // Move vertex to main bone.
        // This will be overridden if vertex has a bind.
        for(let v = 0; v < visual.vertices.length; v++) {
            visual.vertices[v].pos = visual.vertices[v].init_pos;
            visual.vertices[v] = inheritVert(visual.vertices[v].pos, bone)
        }

        for(let bi = 0; bi < visual.binds.length; bi++) {
            let boneId = visual.binds[bi].bone_id
            if boneId == -1 {
                continue;
            }
            bindBone: Bone = bones.find(bone => bone.id == bId))
            bind: Bind = visual.binds[bi]
            for(let v = 0; v < bind.verts.length; v++) {
                id: Int = bind.verts[v].id

                if !bind.isPath {
                    // weights
                    const vert: Vertex = visual.vertices[id]
                    const weight: Float = bind.verts[v].weight
                    const endPos: Vec2 = inheritVert(vert.init_pos, bindBone) - vert.pos
                    vert.pos += endPos * weight
                    continue
                }

                // pathing

                // Check out the 'Pathing Explained' section below for a
                // comprehensive explanation.

                // 1.
                // get previous and next bind
                const prev: Int = bi > 0 ? bi - 1 : bi;
                const next: Int = min(bi + 1, visual.binds.length - 1);
                const prevBone: Bone = bones.find(bone => bone.id == visual.binds[prev].bone_id);
                const nextBone: Bone = bones.find(bone => bone.id == visual.binds[next].bone_id);

                // 2.
                // get the average of normals between previous and next bind
                const prevDir: Vec2 = bindBone.pos - prevBone.pos;
                const nextDir: Vec2 = nextBone.pos - bindBone.pos;
                const prevNormal: Vec2 = normalize(Vec2.new(-prevDir.y, prevDir.x));
                const nextNormal: Vec2 = normalize(Vec2.new(-nextDir.y, nextDir.x));
                const average: Vec2 = prevNormal + nextNormal;
                const normalAngle: Float = atan2(average.y, average.x);

                // 3.
                // move vert to bind, then rotate it around bind by normalAngle
                const vert: Vertex = visual.vertices[id];
                vert.pos = vert.initPos + bindBone.pos;
                const rotated: Vec2 = rotate(vert.pos - bindBone.pos, normalAngle);
                vert.pos = bindBone.pos + (rotated * bind.verts[v].weight);
                visual.vertices[id] = vert;
            }
        }
    }
}

function inheritVert(pos: Vec2, bone: Bone): Vec2 {
    pos *= bone.scale
    pos = rotate(pos, bone.rot)
    pos += bone.pos
    return pos
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

1. Reset vertex position to its initial position + bind position
2. Rotate vertex around bind with angle from 2nd step
