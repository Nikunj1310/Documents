# CharacterController

**Category:** Unity Component  
**Namespace:** `UnityEngine`  
**Used For:** Player movement with collision detection (no physics simulation)

---

## What is CharacterController?

**CharacterController** is a Unity component that lets you move a GameObject with collision detection, but **without full physics simulation**. It's specifically designed for player character movement.

**Think of it as:** A capsule-shaped collider that you can move programmatically. It handles collision with walls/floors automatically, but doesn't respond to physics forces like gravity from Rigidbody.

---

## Why CharacterController (Not Rigidbody)?

| Feature | CharacterController | Rigidbody |
|---|---|---|
| **Control** | Direct, precise control | Physics forces (can feel floaty) |
| **Gravity** | Built-in, simple | Must apply force every frame |
| **Slopes** | Handles automatically | Must script slope logic |
| **Stairs** | Step offset built-in | Needs complex scripting |
| **Pushing objects** | ❌ Can't push Rigidbodies | ✅ Can push other Rigidbodies |
| **Physics forces** | ❌ Immune to explosions, wind | ✅ Responds to forces |
| **Performance** | Faster | Slower (full physics) |
| **Use case** | **Player characters** | Physics objects (balls, crates) |

**In POSSESSED:** We use CharacterController for player movement because we want direct, responsive control. Bots use NavMeshAgent instead (which also doesn't use Rigidbody).

---

## Key Properties

### Shape Settings

```csharp
CharacterController controller = GetComponent<CharacterController>();

// Capsule shape
float radius = controller.radius;      // Radius of capsule (0.5m default)
float height = controller.height;      // Height of capsule (2.0m default)
Vector3 center = controller.center;    // Center offset from GameObject position

// Collision
float skinWidth = controller.skinWidth;  // Collision margin (0.08 default, don't change)
```

**In POSSESSED Actor prefab:**
- Radius: 0.5
- Height: 2.0
- Center: (0, 1, 0) — center of capsule is 1m above ground (capsule goes from y=0 to y=2)

### Movement Settings

```csharp
float slopeLimit = controller.slopeLimit;     // Max slope angle you can walk up (45° default)
float stepOffset = controller.stepOffset;     // Max step height you can walk up (0.3m default)
```

**Slope limit:** If you walk into a 50° slope and slopeLimit is 45°, you'll stop (can't climb it).

**Step offset:** Lets you walk up stairs. If step is 0.3m tall and stepOffset is 0.3m, you automatically step up. If step is 0.5m tall, you stop (too high).

### Ground Detection

```csharp
bool isGrounded = controller.isGrounded;  // Are we touching the ground?
```

**CRITICAL for POSSESSED:**
- `isGrounded` only works with **BoxCollider, SphereCollider, CapsuleCollider**
- **Does NOT work with MeshCollider** (like Plane uses)
- **This is why we use Cube for floor, not Plane**

See [[Plane vs Cube Floor]] for full explanation.

---

## Movement Methods

### SimpleMove (Built-in Gravity)

```csharp
Vector3 move = new Vector3(horizontal, 0, vertical) * speed;
controller.SimpleMove(move);
```

**What it does:**
- Moves the controller in the direction you specify
- Automatically applies **gravity** (no need to add vertical velocity)
- Returns `true` if touching ground, `false` if airborne
- Speed is in **meters per second** (not affected by Time.deltaTime)

**Example in POSSESSED:**
```csharp
void Update() {
    float h = Input.GetAxis("Horizontal");  // A/D keys
    float v = Input.GetAxis("Vertical");    // W/S keys
    Vector3 move = new Vector3(h, 0, v).normalized * moveSpeed;
    controller.SimpleMove(move);
}
```

### Move (Manual Control)

```csharp
Vector3 motion = velocity * Time.deltaTime;
controller.Move(motion);
```

**What it does:**
- Moves the controller by exact amount (in meters)
- **Does NOT apply gravity automatically** (you must add vertical velocity yourself)
- More control than SimpleMove, but more complex

**When to use:**
- Jumping (need to control vertical velocity)
- Custom gravity
- Flying characters

**Example with jumping:**
```csharp
float verticalVelocity = 0f;

void Update() {
    // Horizontal movement
    float h = Input.GetAxis("Horizontal");
    float v = Input.GetAxis("Vertical");
    Vector3 move = new Vector3(h, 0, v).normalized * moveSpeed;
    
    // Jumping
    if (controller.isGrounded && Input.GetButtonDown("Jump")) {
        verticalVelocity = jumpSpeed;
    }
    
    // Gravity
    verticalVelocity += Physics.gravity.y * Time.deltaTime;
    move.y = verticalVelocity;
    
    // Apply movement
    controller.Move(move * Time.deltaTime);
}
```

**In POSSESSED:** We use **SimpleMove** (no jumping needed, simpler).

---

## Collision Detection

### CollisionFlags

```csharp
CollisionFlags flags = controller.Move(motion);

if ((flags & CollisionFlags.Below) != 0) {
    Debug.Log("Hit ground");
}
if ((flags & CollisionFlags.Sides) != 0) {
    Debug.Log("Hit wall");
}
if ((flags & CollisionFlags.Above) != 0) {
    Debug.Log("Hit ceiling");
}
```

**What it returns:**
- `CollisionFlags.Below` — Touching ground
- `CollisionFlags.Sides` — Touching wall
- `CollisionFlags.Above` — Touching ceiling
- `CollisionFlags.None` — Not touching anything

### OnControllerColliderHit

```csharp
void OnControllerColliderHit(ControllerColliderHit hit) {
    Debug.Log("Hit: " + hit.collider.gameObject.name);
    
    // Push Rigidbodies (requires extra code, CharacterController doesn't do this automatically)
    Rigidbody rb = hit.collider.GetComponent<Rigidbody>();
    if (rb != null && !rb.isKinematic) {
        rb.AddForce(hit.moveDirection * pushPower, ForceMode.Impulse);
    }
}
```

**When it fires:** Every time the CharacterController collides with something.

**Use case:** Detecting what you bumped into, pushing objects.

---

## Common Patterns

### WASD Movement (POSSESSED Style)

```csharp
[SerializeField] float moveSpeed = 5f;
CharacterController controller;

void Awake() {
    controller = GetComponent<CharacterController>();
}

void Update() {
    // Get input
    float h = Input.GetAxis("Horizontal");  // -1 to 1
    float v = Input.GetAxis("Vertical");    // -1 to 1
    
    // Calculate movement direction
    Vector3 move = new Vector3(h, 0, v).normalized * moveSpeed;
    
    // Apply movement (SimpleMove handles gravity)
    controller.SimpleMove(move);
}
```

### Mouse-Aim Rotation (POSSESSED Style)

```csharp
Camera cam;

void Start() {
    cam = GetComponentInChildren<Camera>();  // LocalOnly camera
}

void Update() {
    // Raycast from camera to ground
    Ray ray = cam.ScreenPointToRay(Input.mousePosition);
    if (Physics.Raycast(ray, out RaycastHit hit, 100f, LayerMask.GetMask("Ground"))) {
        // Look at mouse cursor position
        Vector3 lookDir = (hit.point - transform.position).normalized;
        lookDir.y = 0;  // Keep rotation horizontal only
        
        if (lookDir.sqrMagnitude > 0.01f) {
            transform.rotation = Quaternion.LookRotation(lookDir);
        }
    }
}
```

---

## CharacterController vs NavMeshAgent Conflict

**CRITICAL BUG:** Both CharacterController and NavMeshAgent try to move the same GameObject.

**Problem:**
- CharacterController: Moves via `SimpleMove()` or `Move()`
- NavMeshAgent: Moves automatically toward `SetDestination()`
- Both enabled → they fight for control → **jittering/stuttering**

**Solution:**
Only enable ONE at a time:

```csharp
void Start() {
    bool isBot = !photonView.IsMine;
    
    if (isBot) {
        // Bot: Use NavMeshAgent for AI pathfinding
        navAgent.enabled = true;
        controller.enabled = false;
    } else {
        // Player: Use CharacterController for input
        controller.enabled = true;
        navAgent.enabled = false;
    }
}
```

See [[NavMeshAgent + CharacterController Conflict]] for full explanation.

---

## Common Mistakes

### 1. Using Plane for Ground (isGrounded Fails)

**Wrong:**
```
Floor = Plane (uses MeshCollider)
```
- CharacterController.isGrounded doesn't work reliably with MeshCollider
- Result: `isGrounded` always false, jumping breaks

**Right:**
```
Floor = Cube, scale (30, 0.1, 30) (uses BoxCollider)
```
- CharacterController.isGrounded works correctly
- See [[Plane vs Cube Floor]]

### 2. Forgetting Time.deltaTime with Move()

**Wrong:**
```csharp
controller.Move(velocity);  // Movement is framerate-dependent
```

**Right:**
```csharp
controller.Move(velocity * Time.deltaTime);  // Framerate-independent
```

**Note:** SimpleMove does NOT need Time.deltaTime (it's built-in).

### 3. Not Normalizing Movement Vector

**Wrong:**
```csharp
Vector3 move = new Vector3(h, 0, v) * speed;  // Diagonal is faster (1.41x)
```

**Right:**
```csharp
Vector3 move = new Vector3(h, 0, v).normalized * speed;  // Same speed all directions
```

**Why:** Vector (1, 0, 1) has magnitude 1.41 (diagonal). Normalized brings it to magnitude 1.0.

### 4. Trying to Use Physics Forces

**Doesn't work:**
```csharp
controller.AddForce(Vector3.up);  // CharacterController has no AddForce method
```

**Use Rigidbody if you need:**
- Explosions pushing player
- Wind zones affecting player
- Other physics forces

---

## Performance Notes

- **CharacterController is faster than Rigidbody** (no full physics simulation)
- **isGrounded is cheap** — safe to check every frame
- **Move() is more expensive than just setting transform.position**, but handles collision

---

## In POSSESSED

**Actor.prefab CharacterController settings:**
- Radius: 0.5
- Height: 2.0
- Center: (0, 1, 0)
- Slope Limit: 45
- Step Offset: 0.3
- Skin Width: 0.08

**Usage:**
- **Player actors:** CharacterController enabled, NavMeshAgent disabled
- **Bot actors:** CharacterController disabled, NavMeshAgent enabled
- **Movement script:** TopDownController.cs uses `SimpleMove()`

---

## Quick Reference

| Property/Method | Purpose |
|---|---|
| `radius` | Capsule radius (0.5m for POSSESSED) |
| `height` | Capsule height (2.0m for POSSESSED) |
| `center` | Offset from GameObject position |
| `isGrounded` | Touching ground? (only works with Box/Sphere/Capsule colliders) |
| `SimpleMove(Vector3)` | Move with built-in gravity (m/s) |
| `Move(Vector3)` | Move with manual control (meters) |
| `slopeLimit` | Max walkable slope angle (45° default) |
| `stepOffset` | Max auto-step height (0.3m default) |

---

## Related Notes

- [[NavMeshAgent]] — AI pathfinding (used for bots)
- [[NavMeshAgent + CharacterController Conflict]] — Why only enable one
- [[Plane vs Cube Floor]] — Why Cube floor is required
- [[TopDownController]] — Script using CharacterController in POSSESSED
- [[Rigidbody]] — Physics-based movement (alternative)

---

**Next:** Read [[NavMeshAgent]] to understand bot movement, then [[TopDownController]] to see CharacterController in action.
