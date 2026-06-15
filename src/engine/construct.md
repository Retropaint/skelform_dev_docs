# `Construct()` - Engine

Calls `Construct()` from the generic runtime, then modifies constructed bones
based on user options and engine quirks.

```typescript
ConstructOptions {
    position: Vec2,
    scale: Vec2,
    velocity: Vec2,
}

function Construct(armature: Armature*, options: ConstructOptions): Bone[] {
    // this will modify armature.constructed_bones, as well as armature.bones for physics fields
    GenericRuntime.construct(armature)

    for(let b = 0; b < armature.bones.length; b++) {
        let const_bone = armature.constructed_bones[b];

        // engine quirks like negative Y or reversed rotations can be applied here
        const_bone.pos.y = -const_bone.pos.y;
        const_bone.rot   = -const_bone.rot;

        // apply user options
        const_bone.scale *= options.scale;
        const_bone.pos   *= options.scale;
        const_bone.pos   += options.position;

        // bones must be flipped if scale is in the negatives
        GenericRuntime.checkBoneFlip(bone, options.scale)

        // velocity only affects position for physics
        if(const_bone.physics_id != -1) {
            let physics = armature.physics[const_bone.physics_id];
            physics.global_pos -= options.velocity;
        }

        // engine quirks & user options applied to vertices
        if(const_bone.visuals_id != -1) {
            let visuals = armature.physics[const_bone.visuals_id];
            visuals.vertices.forEach(vert => {
                vert.pos.y = -vert.pos.y;
                vert.pos   *= options.scale;
                vert.pos   += options.position;
            })
        }
    }
}
```
