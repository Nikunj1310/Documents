# Camera

## Overview

A **Camera** is a [[Component]] that renders the game world from a specific viewpoint. Every Unity scene needs at least one camera to display anything on screen.

**Think of it as:** A virtual movie camera. What the camera sees is what the player sees.

## How Cameras Work

1. Camera has a position and rotation ([[Transform]])
2. Camera defines a viewing frustum (cone-shaped volume)
3. Everything inside frustum is rendered to the screen
4. Everything outside frustum is culled (not rendered)

## Camera Component Properties

| Property | Purpose | Common Values |
|---|---|---|
| **Clear Flags** | What to render behind geometry | Skybox, Solid Color |
| **Culling Mask** | Which layers to render | Everything, or exclude UI |
| **Projection** | Perspective or Orthographic | Perspective (3D games) |
| **Field of View** | How wide the camera sees (degrees) | 60° (default), 90° (wide) |
| **Clipping Planes** | Near/far render distances | Near: 0.3, Far: 1000 |
| **Depth** | Render order (multiple cameras) | 0 (main), 1 (overlay) |
| **Target Texture** | Render to texture instead of screen | Null (render to screen) |

## Camera Setup in POSSESSED

### Actor Camera (Top-Down View)

```
Actor (moves around game world)
└─ LocalOnly (enabled only for photonView.IsMine)
   └─ Camera (local position: 0, 18, -8)
```

**Position:** `(0, 18, -8)` relative to Actor
- 18 units above Actor (bird's eye view)
- 8 units behind Actor (isometric angle)

**Rotation:** `(70, 0, 0)` — Pitch down 70°
- Points camera downward at Actor
- Creates top-down perspective

**Settings:**
- Clear Flags: Skybox
- Field of View: 60°
- Tag: **Untagged** (NOT MainCamera)

### Why Child of Actor?

**Parent-child hierarchy = automatic following**

```csharp
// NO SCRIPT NEEDED
// Camera follows Actor because it's a child
// Actor moves → Camera moves with it
```

**Alternative (DON'T DO THIS):**
```csharp
// Old way: Separate CameraRig + CameraFollow script
public class CameraFollow : MonoBehaviour {
    public Transform target;
    public Vector3 offset;
    void LateUpdate() {
        transform.position = target.position + offset;
        transform.LookAt(target);
    }
}
```

This is **unnecessary complexity**. Use parent-child hierarchy instead.

## Camera Tag (MainCamera)

Unity uses the `MainCamera` tag to identify the primary camera.

```csharp
Camera mainCam = Camera.main;  // Finds camera tagged "MainCamera"
```

### In Multiplayer (POSSESSED):

**Problem:** Each player spawns an Actor with a Camera. 8 players = 8 cameras. Which is "MainCamera"?

**Solution:** Tag all Actor cameras as **Untagged**. Only enable the local player's camera.

```csharp
// In Actor.cs Start()
bool isLocal = photonView.IsMine;
localOnlyRoot.SetActive(isLocal);  // Only enable YOUR camera

if (isLocal) {
    // Disable scene MainCamera (if it exists)
    var mainCam = Camera.main;
    if (mainCam != null) mainCam.gameObject.SetActive(false);
}
```

## Common Camera Operations

### Get Main Camera
```csharp
// Find camera tagged "MainCamera"
Camera mainCam = Camera.main;

// PERFORMANCE WARNING: Camera.main searches scene every time
// Cache it in Start() if you need it often
private Camera cam;
void Start() { cam = Camera.main; }
```

### Screen to World Position
```csharp
// Convert mouse position to world position
Ray ray = cam.ScreenPointToRay(Input.mousePosition);
if (Physics.Raycast(ray, out RaycastHit hit, 100f)) {
    Vector3 worldPos = hit.point;
}
```

**Use case in POSSESSED:** Click on ground to find look direction.

```csharp
// TopDownController: Face mouse cursor
Ray ray = cam.ScreenPointToRay(Input.mousePosition);
if (Physics.Raycast(ray, out RaycastHit hit, 100f, LayerMask.GetMask("Ground"))) {
    Vector3 lookDir = (hit.point - transform.position).normalized;
    transform.rotation = Quaternion.LookRotation(lookDir);
}
```

### World to Screen Position
```csharp
// Convert world position to screen position (for UI placement)
Vector3 screenPos = cam.WorldToScreenPoint(worldPos);
```

**Use case:** Place health bar above character's head.

### Check if Position is Visible
```csharp
// Is a world position inside camera frustum?
Plane[] planes = GeometryUtility.CalculateFrustumPlanes(cam);
bool isVisible = GeometryUtility.TestPlanesAABB(planes, bounds);
```

## Field of View (FOV)

FOV controls how much the camera sees horizontally.

| FOV | Effect | Use Case |
|---|---|---|
| 30-50° | Narrow, zoomed in | Sniper scope, cinematic |
| 60-70° | Natural, human-like | Default, most games |
| 80-100° | Wide, fish-eye | Racing games, action |

**In POSSESSED:** 60° (natural perspective for top-down view)

### Dynamic FOV (Speed Effect)
```csharp
// Increase FOV when moving fast
float targetFOV = isMovingFast ? 70f : 60f;
cam.fieldOfView = Mathf.Lerp(cam.fieldOfView, targetFOV, Time.deltaTime * 2f);
```

## Projection Types

### Perspective (3D)
- Objects farther away appear smaller
- Depth perception (realistic)
- **POSSESSED uses this**

### Orthographic (2D)
- Objects same size regardless of distance
- No depth perception (flat)
- Use for 2D games, UI cameras

## Multiple Cameras

You can have multiple cameras rendering to the same screen.

**Camera Depth** determines render order:
- Depth -1 renders first (background)
- Depth 0 renders second (main game)
- Depth 1 renders last (UI overlay)

**Example: Minimap**
```csharp
// Main camera: Depth 0
// Minimap camera: Depth 1, Target Texture = minimap render texture
```

## Culling Mask (Selective Rendering)

Control which layers the camera renders.

```csharp
// Don't render "LocalOnly" objects on other players' cameras
cam.cullingMask = ~(1 << LayerMask.NameToLayer("LocalOnly"));

// Only render UI layer
cam.cullingMask = LayerMask.GetMask("UI");
```

## Camera in POSSESSED Architecture

### Why One Camera Per Actor?

**Each player has their own camera under LocalOnly:**

```
Player A's Actor
└─ LocalOnly (enabled on Player A's machine only)
   └─ Camera (renders on Player A's machine)

Player B's Actor
└─ LocalOnly (enabled on Player B's machine only)
   └─ Camera (renders on Player B's machine)
```

**Result:**
- Player A only sees through their own camera
- Player B only sees through their own camera
- No network sync needed (cameras are local-only)
- Performance: Each client renders 1 camera, not 8

## Common Mistakes

### 1. Camera.main Every Frame
```csharp
// SLOW: Searches scene every frame
void Update() {
    Camera cam = Camera.main;
    Ray ray = cam.ScreenPointToRay(Input.mousePosition);
}

// FAST: Cache in Start()
private Camera cam;
void Start() { cam = Camera.main; }
void Update() {
    Ray ray = cam.ScreenPointToRay(Input.mousePosition);
}
```

See [[Common-Mistake-Camera-Main-Every-Frame]]

### 2. Multiple MainCameras in Multiplayer
```csharp
// WRONG: 8 players = 8 cameras tagged "MainCamera"
// Camera.main returns random one

// RIGHT: Tag Actor cameras as Untagged
// Only enable local player's camera
```

### 3. Separate CameraRig GameObject
```csharp
// WRONG: Separate CameraRig + CameraFollow script (unnecessary)

// RIGHT: Camera as child of Actor (automatic following)
```

See [[Common-Mistake-Camera-Separate-GameObject]]

## Performance Tips

1. **Reduce far clipping plane** — Don't render distant objects (use fog to hide cutoff)
2. **Use culling masks** — Don't render invisible layers
3. **Occlusion culling** — Don't render objects behind walls (bake in Unity)
4. **One shadow light per camera** — Multiple shadow lights = expensive

## Related Notes

- [[Transform]] — Camera position/rotation
- [[Light]] — Illuminates what camera sees
- [[GameObject]] — Camera is a component on GameObject
- [[Component]] — Camera is a component type
- [[Actor]] — Has camera under LocalOnly
- [[Pattern-Local-Only-Rendering]] — Why camera is under LocalOnly

## Quick Reference

```csharp
// Get main camera
Camera cam = Camera.main;

// Screen to world ray
Ray ray = cam.ScreenPointToRay(Input.mousePosition);
if (Physics.Raycast(ray, out RaycastHit hit)) {
    Vector3 worldPos = hit.point;
}

// World to screen position
Vector3 screenPos = cam.WorldToScreenPoint(worldPosition);

// Check if visible
bool isVisible = cam.WorldToScreenPoint(worldPos).z > 0;

// Set FOV
cam.fieldOfView = 70f;

// Set culling mask
cam.cullingMask = LayerMask.GetMask("Default", "Player");
```

## Troubleshooting

| Problem | Solution |
|---|---|
| Black screen | Check camera Clear Flags, ensure camera enabled |
| Nothing renders | Check culling mask includes object layers |
| Camera doesn't follow player | Ensure camera is child of player GameObject |
| Multiple cameras fighting | Use depth, disable unnecessary cameras |
| Objects clipping near camera | Increase near clipping plane |
