# Interpolation

When handling [core animation logic](./core_anim_logic.md), Most bone fields
such as position, rotation, and scale must be interpolated across keyframes.

This page will cover best practices for how to handle this.

## Iterating Keyframes

The following section will use this keyframe as a reference:

```json
{
  "frame": 12,
  "bone_id": 4,
  "element_id": 2,
  "element": "PositionX",
  "value": 7.0,
  "transition": "Linear"
}
```

The keyframe structure of the `animations` object is designed to allow bone
fields to be easily identifiable via `element` or `element_id`. The above
keyframe animates the bone (of id 4)'s position on the X axis.

In relation to the element, the `value` field determines the actual value to
interpolate by. On frame 12, the bone's X position will be it's own plus 7.

## Meta Logic

Pseudo-code:

```rust
fn interpolate(
  keyframes: Keyframe[],
  frame: int,
  bone_id: int,
  element: Element,
  default_value: float,
) {
  let prev_kf;
  let next_kf;

  // get most recent frame
  for kf in keyframes {
    if kf.frame < frame && kf.bone_id == bone_id && kf.element == element {
      prev_kf = kf
    }
    break
  }

  // get next frame
  for kf in keyframes {
    if kf.frame > frame && kf.bone_id == bone_id && kf.element == element {
      next_kf = kf
      break
    }
  }

  // ensure both points are pointing somewhere
  if prev_kf == null {
    prev_kf == next_kf
  } else if next_kf == null {
    next_kf == null
  }

  // if both are null, return default value
  if prev_kf == null && next_kf == null {
    return default_value
  }

  let total_frames = next_kf.frame - prev_kf.frame;

  let current_frame = frame - prev_kf.frame;

  let result = interpolate_float(prev_kf.value, next_kf.value, current_frame, total_frames);

  return result
}
```
