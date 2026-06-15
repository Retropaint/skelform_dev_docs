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

<pre> <code class="language-typescript hljs">function Construct(armature: Armature): Bone[] {
    // initialize constructed_bones
    if (armature.constructed_bones == undefined) {
        armature.constructed_bones = clone(armature.bones);
    } else {
        // constructed_bones may have been used later for drawing
        // which sorts them by zindex, so sort back by id
        armature.constructed_bones.sort((bone) => bone.id);
    }

    // 1st inheritance pass
    <a href="#resetinheritance">resetInheritance</a>(armature.constructed_bones, armature.bones);
    <a href="#inheritance">inheritance</a>(armature.constructed_bones, {}, []);

    // 2nd inheritance pass: inverse kinematics
    if (armature.inverse_kinematics.length > 0) {
        ikRots: Object = <a href="#inversekinematics">inverseKinematics</a>(
           armature.constructed_bones, armature.inverse_kinematics
        );
        <a href="#resetinheritance">resetInheritance</a>(armature.constructed_bones, armature.bones);
        <a href="#inheritance">inheritance</a>(armature.constructed_bones, ikRots, []);
    }
    
    // 3rd inheritance pass: physics
    if (armature.physics.length > 0) {
        <a href="#simulatePhysics">simulatePhysics</a>(armature.constructed_bones, armature.physics);
        <a href="#resetinheritance">resetInheritance</a>(aramture.constructed_bones, armature.bones);
        <a href="#inheritance">inheritance</a>(armature.constructed_bones, ikRots, armature.physics);
    }

    // mesh deformation
    <a href="#constructverts">constructVerts</a>((armature.constructed_bones, armature.visuals);
}
</code> </pre>

## `inheritance()`

Child bones need to inherit their parent.

```typescript
inheritance(bones: Bone[], ikRots: Object, physics: Physics[]) {
    for(let b = 0; b < bones.length; b++) {
        if(bones[b].parentId != -1) {
            parent: Bone = bones[bones[b].parentId];

            let orbit_rot = bones[bones[b].parent_id as usize].rot

            // apply orbital difference, if rotation resistance physics is active
            let phys = physics[bones[b].physics_id];
            if(phys != undefined && phys.sway > 0) {
                orbit_rot -= phys.global_orbit_diff;
            }
            bones[b].rot += orbit_rot;

            bones[b].scale *= parent.scale;

            // adjust child's distance from player as it gets bigger/smaller
            bones[b].pos *= parent.scale;

            // rotate child around parent as if it were orbitting
            bones[b].pos = rotate(&bones[b].pos, parent.rot);

            bones[b].pos += parent.pos;
        }

        // override bone's rotation from inverse kinematics
        if(ikRots[b]) {
            bones[b].rot = ikRots[b];
        }

        // apply physics, if armature_bones is provided
        let phys = physics[bones[b].physics_id];
        if phys != undefined {
            if(bones[b].phys_rot_damping > 0.) {
                bones[b].rot = phys.global_rot;
            }
            if(bones[b].phys_pos_damping > 0.) {
                bones[b].pos = phys.global_pos;
            }
            if(bones[b].phys_scale_damping > 0.) {
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
resetInheritance(constructed_bones: Bone[], bones: Bone[]) {
    for(let b = 0; b < bones.length; b++) {
        constructed_bones[b].pos = bones[b].pos;
        constructed_bones[b].rot = bones[b].rot;
        constructed_bones[b].scale = bones[b].scale;
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
wth `inverse_kinematics`.

<pre> <code class="language-typescript hljs">function inverseKinematics(
    bones: Bone[], inverse_kinematics: InverseKinematics[]
): Object {
    ikRot: Object = {}

    for(let family of inverse_kinematics) {
        // get relevant bones from the same set
        if(family.target_id == - 1) {
            continue
        }
        root: Vec2 = bones[family.bone_ids[0]].pos
        target: Vec2 = bones[family.target_id].pos
        familyBones: Bone[] = bones.filter(|bone|
            family.bone_ids.contains(bone.id)
        )

        // determine which IK mode to use
        switch(family.mode) {
            case "FABRIK":
                for range(10) {
                    <a href="#fabrik">fabrik</a>(*familyBones, root, target)
                }
            case "Arc":
                <a href="#arcik">arcIk</a>(*familyBones, root, target)
        }

        <a href="#pointbones">pointBones</a>(*bones, family)
        <a href="#applyconstraints">applyConstraints</a>(*bones, family)

        // add rotations to ikRot, with bone ID being the key
        for(let b = 0; b < family.ikBoneIds.length; b++) {
            // last bone of IK should have free rotation
            if(b == family.bone_ids.len() - 1) {
                continue
            }
            ikRot[family.bone_ids[b]] = bones[family.bone_ids[b]].rot
        }
    }

    return ikRot
}
</code> </pre>

## `pointBones()`

Point each bone toward the next one.

Used by `inverseKinematics()` to get the final bone's rotations.

```typescript
function pointBones(bones: Bone[]*, family: Bone) {
    endBone: Bone = bones[family.ik_bone_ids[-1]]
    tipPos: Vec2 = endBone.pos
    for(let i = family.ik_bone_ids.length; i > 0; i--) {
        if(i == family.ik_bone_ids.length - 1) {
            // end bone should follow target bone rotation, if miimc_target is true
            if(family.mimic_target) {
                endBone.rot = bones[family.target_id]
            }
            break;
        }
        bone = *bones[family.ik_bone_ids[i]]
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
  let jointDir: Vec2 = normalize(bones[family.ikBoneIds[1]].pos - root);
  let baseDir: Vec2 = normalize(target - root);
  let dir: Float = jointDir.x * baseDir.y - baseDir.x * jointDir.y;
  let baseAngle: Float = atan2(baseDir.y, baseDir.x);
  let cw: Bool = family.ikConstraint == "Clockwise" && dir > 0;
  let ccw: Bool = family.ikConstraint == "CounterClockwise" && dir < 0;
  if (ccw || cw) {
    for (let id of family.ikBoneIds) {
      bones[id].rot = -bones[id].rot + baseAngle * 2;
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
  nextPos: Vec2 = target;
  nextLength: Float = 0.0;
  for (let b = bones.length - 1; b > 0; b--) {
    length: Vec2 = normalize(nextPos - bones[b].pos) * nextLength;
    if (isNaN(length)) length = new Vec2(0, 0);
    if (b != 0) nextLength = magnitude(bones[b].pos - bones[b - 1].pos);
    bones[b].pos = nextPos - length;
    nextPos = bones[b].pos;
  }

  // backward-reaching
  prevPos: Vec2 = root;
  prevLength: Float = 0.0;
  for (let b = 0; b < bones.length; b++) {
    length: Vec2 = normalize(prevPos - bones[b].pos) * prevLength;
    if (isNaN(length)) length = new Vec2(0, 0);
    if (b != bones.len() - 1)
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
    dist: Float[] = [0.]

    maxLength: Vec2 = magnitude(bones.last().pos - root)
    currLength: Float = 0.
    for(let b = 1; b < bones.length; b++) {
        length: Float = magnitude(bones[b].pos - bones[b - 1].pos)
        currLength += length;
        dist.push(currLength / maxLength)
    }

    base: Vec2 = target - root
    baseAngle: Float = base.y.atan2(base.x)
    baseMag: Float = magnitude(base).min(maxLength)
    peak: Float = maxLength / baseMag
    valley: Float = baseMag / maxLength
    for(let b = 1; b < bones.length; b++) {
        bones[b].pos = new Vec2(
            bones[b].pos.x * valley,
            root.y + (1.0 - peak) * sin(dist[b] * PI) * baseMag,
        )

        rotated: Float = rotate(bones[b].pos - root, baseAngle)
        bones[b].pos = rotated + root
    }
}
```

## `simulatePhysics()`

Processes all physics:

- Position (`phys_pos_damping`)
- Scale (`phys_scale_damping`)
- Rotation (`phys_rot_damping`)
- Sway (`phys_sway`)
- Bounce (`phys_rot_bounce`)

<pre> <code class="language-typescript hljs">function simulatePhysics(constructedBones: Bone[], physics: Physics[]) {
    for(let b = 0; b < constructed_bones.length; b++) {
        if constructedBones[b].physics_id == -1 {
            continue;
        }
        let physics = &mut physics[constructedBones[b].physics_id as usize];

        let s = Vec2(0.3, 0.3)
        let e = Vec2(0.6, 0.6)
        let const_bone = &constructedBones[b]
        let prev_pos = physics.phys_global_pos

        // interpolate position
        if(physics.pos_damping > 0 || physics.sway > 0) {
            let damping = Vec2(physics.pos_damping, physics.pos_damping)

            // ratio
            if(physics.pos_ratio < 0) {
                damping.y *= 1. - Math.abs(physics.pos_ratio)
            } else if(physics.pos_ratio > 0) {
                damping.x *= 1. - physics.pos_ratio
            }

            physics.global_pos.x = interpolate(2, damping.x, phys_pos.x, const_bone.pos.x, s, e)
            physics.global_pos.y = interpolate(2, damping.y, phys_pos.y, const_bone.pos.y, s, e)
        }

        // interpolate scale
        if(physics.scale_damping > 0) {
            let phys_scale = &physics.global_scale
            let damping = Vec2(physics.scale_damping, physics.scale_damping)

            // ratio
            if(physics.scale_ratio < 0) {
                damping.y *= 1. - Math.abs(physics.scale_ratio)
            } else if(physics.pos_ratio > 0) {
                damping.x *= 1. - physics.scale_ratio
            }

            cb_scale = const_bone.scale
            phys_scale.x = interpolate(2, damping.x, phys_scale.x, cb.scale.x, s, e)
            phys_scale.y = interpolate(2, damping.y, phys_scale.y, cb.scale.y, s, e)
        }

        // interpolate rotation
        if(physics.rot_damping > 0) {
            let rot = shortest_angle_delta(physics.global_rot, const_bone.rot)
            physics.global_rot += rot / physics.rot_damping
        }

        // interpolate parent orbit (rot res, bounce, etc)
        let parent = constructed_bones.find((b) => b.id == const_bone.parent_id)
        if(physics.sway > 0 && parent != None) {
            // 1. get the raw orbit angle between this bone and its parent
            let diff = normalize(const_bone.pos - parent.pos)
            let diff_angle = Math.atan2(diff.y, diff.x)

            // 2. interpolate current orbit angle to raw angle
            let orbit_buffer = shortest_angle_delta(physics.global_orbit, diff_angle)

            // 3. apply bounce to orbit angle
            if(physics.rot_bounce > 0. && physics.rot_bounce <= 1) {
                orbit_buffer += physics.global_orbit_vel / (2 - physics.rot_bounce)
                physics.global_orbit_vel = orbit_buffer
            }

            // 4. apply orbit buffer
            physics.global_orbit += orbit_buffer / 10

            // 5. swing orbit based on position momentum
            let vel = normalize(physics.global_pos - prev_pos)
            let angle = Math.atan2(-vel.y, -vel.x)
            let vel_rot = shortest_angle_delta(physics.global_orbit, angle)
            let strength = magnitude(physics.global_pos - prev_pos) / 1000
            physics.global_orbit += vel_rot * strength * physics.sway

            // 6. apply difference in raw angle and orbit
            physics.global_orbit_diff = diff_angle - physics.global_orbit
        }
    }
}
</code> </pre>

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

        bone: Bone = bones[b]

        // Move vertex to main bone.
        // This will be overridden if vertex has a bind.
        for(let v = 0; v < visual.vertices.length; v++) {
            visual.vertices[v].pos = visual.vertices[v].init_pos;
            visual.vertices[v] = inheritVert(visual.vertices[v].pos, bone)
        }

        for(let bi = 0; bi < visual.binds.length; bi++) {
            let boneId = visual.binds[bi].boneId
            if boneId == -1 {
                continue
            }
            bindBone: Bone = bones.find(|bone| bone.id == bId))
            bind: Bind = visual.binds[bi]
            for(let v = 0; v < bind.verts.length; v++) {
                id: Int = bind.verts[v].id

                if !bind.isPath {
                    // weights
                    vert: Vertex = visual.vertices[id]
                    weight: Float = bind.verts[v].weight
                    endpos: Vec2 = inheritVert(vert.initPos, bindBone) - vert.pos
                    vert.pos += endPos * weight
                    continue
                }

                // pathing

                // Check out the 'Pathing Explained' section below for a
                // comprehensive explanation.

                // 1.
                // get previous and next bind
                binds: Bind[] = visual.binds
                prev: Int = bi > 0 ? bi - 1 : bi
                next: Int = min((bi + 1, binds.length - 1)
                prevBone: Bone = bones.find(|bone| bone.id == binds[prev].boneId)
                nextBone: Bone = bones.find(|bone| bone.id == binds[next].boneId)

                // 2.
                // get the average of normals between previous and next bind
                prevDir: Vec2 = bindBone.pos - prevBone.pos
                nextDir: Vec2 = nextBone.pos - bindBone.pos
                prevNormal: Vec2 = normalize(Vec2.new(-prevDir.y, prevDir.x))
                nextNormal: Vec2 = normalize(Vec2.new(-nextDir.y, nextDir.x))
                average: Vec2 = prevNormal + nextNormal
                normalAngle: Float = atan2(average.y, average.x)

                // 3.
                // move vert to bind, then rotate it around bind by normalAngle
                vert: Vertex = visual.vertices[id]
                vert.pos = vert.initPos + bindBone.pos
                rotated: Vec2 = rotate(vert.pos - bindBone.pos, normalAngle)
                vert.pos = bindBone.pos + (rotated * bind.verts[v].weight)
                visual.vertices[id] = vert
            }
        }
    }
}

function inheritVert(pos: Vec2, bone: Bone): Vec2 {
    pos *= bone.scale
    pos = utils.rotate(&pos, bone.rot)
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
