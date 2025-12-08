# `Construct()` - Engine

Calls `Construct()` from the generic runtime, then modifies bones based on user
options and engine quirks.

**Parameters**:

- Armature
- User options

**Returns**:

- Constructed bones, to be used later for `Draw()`.

```c
Bone[] construct(Armature *armature, ConstructOptions options) {
    Bone[] finalBones = generic_runtime::construct(armature);

    for bone in finalBones {
        // engine quirks like negative Y or reversed rotations can be applied here
        bone.pos.y = -bone.pos.y
        bone.rot   = -bone.rot

        // apply user options
        bone.scale *= options.scale
        bone.pos   *= options.scale
        bone.pos   += options.position

        generic_runtime::check_flip(bone)

        // engine quirks & user options applied to vertices
        for vert in bone.vertices {
            vert.pos.y = -vert.pos.y
            vert.pos   *= options.scale
            vert.pos   += options.position
        }
    }

    finalBones
}
```
