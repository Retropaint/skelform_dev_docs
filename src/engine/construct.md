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
    GenericRuntime::Construct(armature)

    for(let b = 0; b < armature.bones.length; b++) {
        let constructed_bone = &armature.constructed_bones[b];

        // engine quirks like negative Y or reversed rotations can be applied here
        constructed_bone.pos.y = -constructed_bone.pos.y
        constructed_bone.rot   = -constructed_bone.rot

        // apply user options
        constructed_bone.scale *= options.scale
        constructed_bone.pos   *= options.scale
        constructed_bone.pos   += options.position

        // bones must be flipped if scale is in the negatives
        GenericRuntime.CheckBoneFlip(bone, options.scale)

        // velocity only affects position for physics
        armature.bones[b].phys_global_pos -= options.velocity

        // engine quirks & user options applied to vertices
        for(let vert of constructed_bone.vertices) {
            vert.pos.y = -vert.pos.y
            vert.pos   *= options.scale
            vert.pos   += options.position
        }
    }
}
```
