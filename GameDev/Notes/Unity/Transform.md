# Transform

## Overview

Every [[GameObject]] has a Transform component. It defines **where**, **how**, and **what size** the GameObject is in 3D space.

**The Transform is mandatory** — you cannot remove it. It's automatically added when you create a GameObject.

## What Transform Stores

| Property | Type | Purpose |
|---|---|---|
| **Position** | Vector3 | Location in 3D space (x, y, z) |
| **Rotation** | Quaternion | Orientation in 3D space |
| **Scale** | Vector3 | Size multiplier (1, 1, 1 = original size) |

## Local vs World Space

### World Space (Global)
- Position relative to the scene origin (0, 0, 0)
- `transform.position` — world position
- `transform.rotation` — world rotation

### Local Space (Relative to Parent)
- Position relative to parent GameObject
- `transform.localPosition` — position relative to parent
- `transform.localRotation` — rotation relative to parent

**Example:**
```
Actor (world position: 5, 0, 10)
└─ Camera (local position: 0, 18, -8)
   
Camera's world position = (5, 18, 2)
// Parent position + local offset
```

## Parent-Child Hierarchy

GameObjects can be **parented** to other GameObjects. Children inherit parent transformations.

### What Happens When You Move Parent:
- **Parent moves** → Children move with it (maintain relative offset)
- **Parent rotates** → Children rotate around parent's pivot
- **Parent scales** → Children scale proportionally

### Example: Camera Following Actor

```
Actor (moves around)
└─ LocalOnly
   └─ Camera (local position: 0, 18, -8)
```

When Actor moves to (10, 0, 5), Camera automatically moves to (10, 18, -3). **Zero scripts needed.**

## Common Properties

```csharp
// Position
transform.position = new Vector3(0, 1, 0);  // World space
transform.localPosition = new Vector3(0, 1, 0);  // Local space

// Rotation
transform.rotation = Quaternion.identity;  // No rotation
transform.eulerAngles = new Vector3(45, 0, 0);  // 45° pitch
transform.LookAt(target);  // Rotate to face target

// Scale
transform.localScale = new Vector3(2, 2, 2);  // Double size

// Direction vectors
Vector3 forward = transform.forward;  // Blue axis (Z+)
Vector3 right = transform.right;  // Red axis (X+)
Vector3 up = transform.up;  // Green axis (Y+)

// Hierarchy
transform.parent = parentTransform;  // Set parent
Transform child = transform.GetChild(0);  // Get first child
int childCount = transform.childCount;  // Number of children

// Find children by name
Transform eyeAnchor = transform.Find("EyeAnchor");
```

## Euler Angles vs Quaternions

### Euler Angles (Human-Readable)
- Three rotations: X (pitch), Y (yaw), Z (roll)
- Easy to understand: (45, 90, 0) = 45° pitch, 90° turn right
- **Problem:** Gimbal lock (certain rotations break)

```csharp
transform.eulerAngles = new Vector3(70, 0, 0);  // Pitch down 70°
```

### Quaternions (Math-Friendly)
- Four components: (x, y, z, w)
- Hard to read: (0, 0.7071, 0, 0.7071) = ???
- **Advantage:** No gimbal lock, smooth interpolation

```csharp
transform.rotation = Quaternion.Euler(70, 0, 0);  // Convert from Euler
transform.rotation = Quaternion.LookRotation(direction);  // Face direction
```

**Rule of thumb:** Use Euler angles for reading/setting simple rotations. Use Quaternions for math (interpolation, LookAt).

## Common Operations

### Move GameObject

```csharp
// Teleport
transform.position = new Vector3(10, 0, 5);

// Move over time (frame-rate independent)
transform.position += direction * speed * Time.deltaTime;

// Move via CharacterController (handles collision)
controller.Move(direction * speed * Time.deltaTime);
```

### Rotate GameObject

```csharp
// Face a target
transform.LookAt(target);

// Rotate smoothly over time
Quaternion targetRotation = Quaternion.LookRotation(direction);
transform.rotation = Quaternion.Lerp(
    transform.rotation, 
    targetRotation, 
    Time.deltaTime * rotationSpeed
);

// Rotate around axis
transform.Rotate(0, 90 * Time.deltaTime, 0);  // Turn 90°/sec around Y
```

### Scale GameObject

```csharp
// Make twice as big
transform.localScale = Vector3.one * 2;

// Pulse scale over time
float scale = 1 + Mathf.Sin(Time.time) * 0.2f;
transform.localScale = Vector3.one * scale;
```

## Direction Vectors Explained

Every Transform has three perpendicular direction vectors:

| Vector | Color | Axis | Meaning |
|---|---|---|---|
| `forward` | Blue | Z+ | "In front of" |
| `right` | Red | X+ | "To the right of" |
| `up` | Green | Y+ | "Above" |

**Example: Vision Check**

```csharp
// Get direction from observer to target
Vector3 toTarget = target.position - observer.position;
Vector3 direction = toTarget.normalized;

// Check if target is in front of observer
float angle = Vector3.Angle(observer.forward, direction);
if (angle < 45f) {
    // Target is within 45° cone in front
}
```

## Parent-Child Use Cases in POSSESSED

### Camera Following Actor
```
Actor
└─ LocalOnly
   └─ Camera (local: 0, 18, -8)
```
Camera automatically follows Actor. No CameraFollow script needed.

### Lights Following Actor
```
Actor
└─ LocalOnly
   ├─ FocusLight (local: 0, 1.5, 0)
   └─ PeriphLight (local: 0, 1.5, 0)
```
Lights move with Actor automatically.

### Eye-Level Anchor
```
Actor (ground level: y=0)
└─ EyeAnchor (local: 0, 1.5, 0)
```
Vision linecasts use `eyeAnchor.position` (world position = Actor.y + 1.5).

## Common Mistakes

### 1. Using transform.position for Vision Checks
```csharp
// WRONG: Uses ground level (y=0)
Vector3 eyePos = transform.position;

// RIGHT: Uses eye level (y=1.5)
Vector3 eyePos = eyeAnchor.position;
```

Ground-level linecasts hit the floor and always return "blocked".

### 2. Forgetting Time.deltaTime
```csharp
// WRONG: Speed depends on frame rate
transform.position += direction * speed;

// RIGHT: Frame-rate independent
transform.position += direction * speed * Time.deltaTime;
```

Without `Time.deltaTime`, movement is faster on high-fps machines.

### 3. Modifying World Scale Instead of Local
```csharp
// WRONG: Breaks if parent is scaled
transform.lossyScale = Vector3.one * 2;  // Read-only!

// RIGHT: Modify local scale
transform.localScale = Vector3.one * 2;
```

`lossyScale` is read-only (world scale after parent scaling).

### 4. Setting Rotation with Euler Angles Every Frame
```csharp
// INEFFICIENT: Creates garbage
transform.eulerAngles = new Vector3(pitch, yaw, 0);

// BETTER: Use Quaternion methods
transform.rotation = Quaternion.Euler(pitch, yaw, 0);
```

## Performance Tips

1. **Cache Transform references** (don't call `GetComponent<Transform>()` repeatedly)
2. **Avoid setting properties if unchanged** (Unity checks for changes internally)
3. **Use local space when possible** (faster than world space)

## Related Notes

- [[GameObject]] — What Transform is attached to
- [[Vector3]] — Position and scale type
- [[Component]] — Transform is a component
- [[Camera]] — Uses Transform for viewpoint
- [[Actor]] — Main game object using Transform

## Quick Reference

```csharp
// Movement
transform.position += Vector3.forward * speed * Time.deltaTime;

// Rotation
transform.LookAt(target);
transform.rotation = Quaternion.Lerp(current, target, t);

// Hierarchy
transform.parent = newParent;
Transform child = transform.Find("ChildName");

// Directions
Vector3 forward = transform.forward;
Vector3 right = transform.right;
Vector3 up = transform.up;

// Local vs World
Vector3 worldPos = transform.position;
Vector3 localPos = transform.localPosition;
```
