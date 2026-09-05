# Block 7 - Player Input

**Time Budget:** 60 minutes (4:30–5:30)
**Goal:** Write TopDownController.cs — WASD movement, mouse aiming, left-click attack

---

## What You're Building

Player control script. Only runs on `photonView.IsMine` actors (your controlled character). Remote actors are moved by [[PhotonTransformView]] sync.

---

## Prerequisites

- [ ] [[Block 6 - ArenaDirector]] complete
- [ ] Understanding of [[Input API]]
- [ ] Understanding of [[CharacterController]]

---

## Checklist

- [ ] Create `Scripts/Game/TopDownController.cs`
- [ ] Attach to Actor.prefab
- [ ] Test: WASD moves your actor
- [ ] Test: Mouse rotates your actor
- [ ] Test: Left-click triggers kill attempt
- [ ] Test: Remote actors move via PhotonTransformView (not input)

---

## Key Responsibilities

### 1. Movement (WASD)

```csharp
void Update() {
    if (!photonView.IsMine) return;  // Only owner reads input
    
    float h = Input.GetAxis("Horizontal");  // A/D or Left/Right arrow
    float v = Input.GetAxis("Vertical");    // W/S or Up/Down arrow
    
    Vector3 move = new Vector3(h, 0, v);
    if (move.sqrMagnitude > 1f) move.Normalize();  // Diagonal = same speed as cardinal
    
    controller.SimpleMove(move * moveSpeed);
}
```

**What is Input.GetAxis:**
- Returns -1 to 1 (smooth analog input)
- "Horizontal" = A/D keys or left joystick X-axis
- "Vertical" = W/S keys or left joystick Y-axis
- See [[Input API]]

**What is CharacterController.SimpleMove:**
- Moves character in m/s (not m/frame)
- Applies gravity automatically
- Handles slopes and steps
- Alternative: `Move(displacement * Time.deltaTime)` for manual gravity
- See [[CharacterController]]

**Why normalize diagonal:** `(1, 0, 1).magnitude = 1.414` → diagonal moves √2 faster. Normalize to cap at 1.0.

### 2. Rotation (Mouse Aim)

```csharp
void Update() {
    if (!photonView.IsMine) return;
    
    // Raycast from camera through mouse cursor to ground
    Ray ray = cam.ScreenPointToRay(Input.mousePosition);
    if (Physics.Raycast(ray, out RaycastHit hit, 200f, groundLayerMask)) {
        // Calculate direction from actor to hit point
        Vector3 lookDir = hit.point - transform.position;
        lookDir.y = 0f;  // Flatten to XZ plane (ignore height)
        
        if (lookDir.sqrMagnitude > 0.01f) {
            Quaternion targetRot = Quaternion.LookRotation(lookDir);
            transform.rotation = Quaternion.Slerp(transform.rotation, targetRot, rotSpeed * Time.deltaTime);
        }
    }
}
```

**What is Camera.ScreenPointToRay:**
- Converts 2D screen position (mouse cursor) to 3D ray from camera
- Ray starts at camera position, points toward cursor in world space
- See [[Camera]]

**What is Physics.Raycast:**
- Casts ray, returns true if hits collider
- `out RaycastHit hit` receives hit info (point, normal, collider)
- `groundLayerMask` filters: only hit objects on Ground layer
- See [[Physics API]]

**Why flatten to XZ plane:** Top-down game, actor rotates around Y-axis only (yaw). X/Z rotation (pitch/roll) would tilt actor sideways.

**What is Quaternion.Slerp:**
- Spherical linear interpolation between two rotations
- `Slerp(current, target, t)` → smooth rotation toward target
- `t = 0` → no change, `t = 1` → snap to target
- See [[Quaternion]]

### 3. Attack (Left-Click)

```csharp
void Update() {
    if (!photonView.IsMine) return;
    
    if (Input.GetMouseButtonDown(0)) {  // Left-click
        TryAttack();
    }
}

void TryAttack() {
    // Raycast from camera through mouse cursor
    Ray ray = cam.ScreenPointToRay(Input.mousePosition);
    if (Physics.Raycast(ray, out RaycastHit hit, 50f)) {
        Actor target = hit.collider.GetComponentInParent<Actor>();
        if (target != null && target != myActor) {
            // Check if target is in Full vision tier (can kill)
            if (target.currentTier == VisTier.Full) {
                ArenaDirector.Instance.ClaimKill(target.actorIdx);
            }
        }
    }
}
```

**Why raycast for attack:** Mouse cursor → world position → find Actor under cursor. Alternative: check all actors in radius (less precise, more expensive).

**Why check currentTier:** Vision tier is gameplay truth. If target is Silhouette or Hidden, you cannot kill them even if raycast hits their collider. See [[Vision Tiers]].

---

## Full Script Structure

```csharp
using UnityEngine;
using Photon.Pun;

public class TopDownController : MonoBehaviour {
    [Header("Movement")]
    public float moveSpeed = 5f;
    public float rotSpeed = 10f;
    
    [Header("Layers")]
    public LayerMask groundLayerMask;
    
    PhotonView photonView;
    CharacterController controller;
    Camera cam;
    Actor myActor;
    
    void Awake() {
        photonView = GetComponent<PhotonView>();
        controller = GetComponent<CharacterController>();
        myActor = GetComponent<Actor>();
    }
    
    void Start() {
        // Cache camera (expensive to find every frame)
        cam = Camera.main;
        if (cam == null) {
            // LocalOnly camera is not tagged MainCamera
            cam = GetComponentInChildren<Camera>();
        }
    }
    
    void Update() {
        if (!photonView.IsMine) return;  // Only owner controls
        
        HandleMovement();
        HandleRotation();
        HandleAttack();
    }
    
    void HandleMovement() { /* ... */ }
    void HandleRotation() { /* ... */ }
    void HandleAttack() { /* ... */ }
}
```

See **GUIDE-FULL.md §7.4** for complete implementation.

---

## Attach to Actor.prefab

1. Project → Assets/Resources/Actor.prefab → Open
2. Select root
3. Add Component → Top Down Controller
4. Settings:
   - Move Speed: 5
   - Rot Speed: 10
   - Ground Layer Mask: Select "Ground" layer
5. Save prefab

---

## Verification Checklist

- [ ] TopDownController.cs compiles
- [ ] Attached to Actor.prefab
- [ ] Play mode (solo): WASD moves your actor
- [ ] Play mode (solo): Mouse cursor rotates actor toward cursor
- [ ] Play mode (solo): Left-click on dummy actor triggers kill (console log or visual feedback)
- [ ] Play mode (2 clients): Your actor moves with WASD, other actor moves via network sync

---

## Common Mistakes

### 1. Not Checking photonView.IsMine

**Wrong:**
```csharp
void Update() {
    // Runs on all actors, every client reads input = chaos
    float h = Input.GetAxis("Horizontal");
    controller.SimpleMove(...);
}
```

**Right:**
```csharp
void Update() {
    if (!photonView.IsMine) return;  // Only owner
    // ...
}
```

### 2. Using Camera.main Every Frame

**Wrong:**
```csharp
void Update() {
    Ray ray = Camera.main.ScreenPointToRay(...);  // Camera.main is FindObjectWithTag = slow
}
```

**Right:**
```csharp
Camera cam;
void Start() { cam = Camera.main; }  // Cache once
void Update() { Ray ray = cam.ScreenPointToRay(...); }
```

### 3. Not Flattening lookDir

**Wrong:**
```csharp
Quaternion.LookRotation(lookDir);  // Actor tilts up/down toward cursor
```

**Right:**
```csharp
lookDir.y = 0f;  // Flatten to XZ plane
Quaternion.LookRotation(lookDir);  // Actor rotates on Y-axis only
```

### 4. Using GetAxis vs GetAxisRaw

**GetAxis:** Smooth (accelerates/decelerates)
**GetAxisRaw:** Instant (-1, 0, 1)

For top-down action, either works. GetAxisRaw feels snappier.

---

## Time Breakdown

| Task | Time |
|---|---|
| Write movement | 15 min |
| Write rotation | 15 min |
| Write attack | 15 min |
| Attach to prefab | 5 min |
| Test solo | 5 min |
| Test multiplayer | 5 min |
| **Total** | **60 min** |

**Behind schedule?** This is a **hard gate**. Cannot skip player input. Cut time from [[Block 10 - Audio]] instead.

---

## What You Built

Player control:
- WASD movement via CharacterController
- Mouse aiming via raycasting to ground
- Left-click attack via raycast + vision tier check

**Next:** [[Block 8 - Vision System]] — the gameplay truth layer

---

## Related Notes

- [[Input API]]
- [[CharacterController]]
- [[Camera]]
- [[Physics API]]
- [[PhotonView]]
- [[Vision Tiers]]
- [[ArenaDirector]]
- [[Block 6 - ArenaDirector]]
- [[Block 8 - Vision System]]
