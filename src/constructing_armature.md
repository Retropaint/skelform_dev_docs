# Constructing Armature

Before animating can take place, it is best to nail down the construction
process of the armature.

## Forward Kinematics / Inheritance

The properties of bones as laid out in `armature.json` is only local to
themselves, and do not include the parent.

Child bones must inherit their parent's properties:

## Inverse Kinematics

If the armature contains inverse kinematics, construction is done via 3 steps:

1. Forward kinematics
2. Inverse kinematics
3. Forward kinematics (with rotations from 2nd step)

Reasoning:

1. Forward kinematics is run once to put all bones in place. This is needed as
   inverse kinematics will require bones with their world properties.

2. Inverse kinematics is run to calculate the new rotations that the affected
   bones will use.

3. The bones are reset, and forward kinematics runs again with the rotations
   provided by inverse kinematics.
