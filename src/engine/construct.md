# `Construct()` - Engine

Calls `Construct()` from the generic runtime, then modifies bones based on user
options and engine quirks.

```c
Construct(armature: Armature*, options: ConstructOptions): Bone[] {
    finalBones: Bone[] = GenericRuntime::Construct(armature)

    for bone in finalBones {
        // engine quirks like negative Y or reversed rotations can be applied here
        bone.pos.y = -bone.pos.y
        bone.rot   = -bone.rot

        // apply user options
        bone.scale *= options.scale
        bone.pos   *= options.scale
        bone.pos   += options.position

        GenericRuntime::checkFlip(bone)

        // engine quirks & user options applied to vertices
        for vert in bone.vertices {
            vert.pos.y = -vert.pos.y
            vert.pos   *= options.scale
            vert.pos   += options.position
        }
    }

    return finalBones
}
```
