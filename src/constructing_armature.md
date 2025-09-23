# Constructing Armature

Before animations can take place, it is best to nail down the construction
process of the armature.

## Table of Contents

- [Non-Mutability](#non-mutability)
- [Inheritance](#inheritance)
- [Inverse Kinematics](#inverse-kinematics)

## Non-Mutability

It is recommended that the original bones supplied to construction are not
modified.

Instead, construction should work with cloned bones and return them to the
consumer.

This does not apply to generic runtimes, as the supplied bones (coming from an
engine runtime) are expected to be cloned already.

### Reasoning:

Runtimes should ideally be as deterministic as possible. By mutating the
original bones, animations may behave differently depending on what played
before it.

## Inheritance

The properties of bones as laid out in `armature.json` is only local to
themselves, and do not include the parent.

Child bones must inherit their parent's properties:

## Inverse Kinematics

If the armature contains inverse kinematics, construction is done via 3 steps:

1. Inheritance
2. Inverse kinematics
3. Inheritance (with rotations from 2nd step)

### Reasoning:

1. Inheritance is run once to put all bones in place. This is needed as inverse
   kinematics will require bones with their world properties.

2. Inverse kinematics is run to calculate the new rotations that the affected
   bones will use.

3. The bones are reset, and inheritance runs again with the rotations provided
   by inverse kinematics.
