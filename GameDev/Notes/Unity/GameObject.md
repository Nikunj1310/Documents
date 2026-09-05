# GameObject

**Category:** Unity Core Concept  
**Namespace:** `UnityEngine`  
**Inherits From:** `Object`

---

## What is a GameObject?

A **GameObject** is the fundamental building block of every Unity scene. Think of it as an **empty container** — by itself, it does nothing. It only becomes useful when you add **Components** to it.

**Real-world analogy:** A GameObject is like an empty box. The box exists and has a position in space, but it has no functionality until you put things inside it (components like lights, scripts, colliders, etc.).

---

## Key Characteristics

### 1. Every GameObject Has a Transform
The **only** component every GameObject MUST have is a **Transform**. You cannot remove it.

The Transform defines:
- **Position:** Where the GameObject is in 3D space (x, y, z)
- **Rotation:** How the GameObject is rotated (Euler angles or quaternion)
- **Scale:** How big the GameObject is (1, 1, 1 = original size)

### 2. GameObjects Are Composed of Components
A GameObject by itself is useless. You add Components to give it functionality:

| Component | What It Does |
|---|---|
| **MeshRenderer** | Makes it visible (renders a 3D mesh) |
| **Collider** | Gives it collision (BoxCollider, SphereCollider, etc.) |
| **Rigidbody** | Adds physics simulation (gravity, forces) |
| **Light** | Makes it emit light (Point, Spot, Directional) |
| **AudioSource** | Makes it play sounds |
| **Your Scripts** | Custom behavior you write |

### 3. GameObjects Can Be Parents/Children
GameObjects can be **parented** to other GameObjects, creating a hierarchy:

```
Actor (parent)
├─ CapsuleBody (child)
├─ LocalOnly (child)
   ├─ Camera (grandchild)
   └─ FocusLight (grandchild)
```

**Why hierarchy matters:**
- Move parent → all children move with it automatically
- Rotate parent → all children rotate around parent's pivot
- Disable parent → all children are disabled too

**Example in POSSESSED:**
The Camera is a child of the Actor GameObject. When Actor moves to position (5, 0, 10), the Camera moves with it automatically — no script needed.

---

## Creating GameObjects

### In Unity Editor (Design Time)
1. **Hierarchy window → Right-click → Create Empty**
2. **GameObject menu → Create Empty**
3. **Right-click → 3D Object → Cube/Sphere/Capsule/etc.** (creates GameObject with MeshRenderer + Collider)

### In Code (Runtime)
```csharp
// Create empty GameObject
GameObject obj = new GameObject("MyObject");

// Create GameObject from prefab
GameObject instance = Instantiate(prefab, position, rotation);

// Find existing GameObject by name (expensive, avoid in Update)
GameObject floor = GameObject.Find("Floor");

// Find GameObject by tag (faster than by name)
GameObject player = GameObject.FindWithTag("Player");
```

---

## Common GameObject Methods

### Accessing Components
```csharp
// Get component on this GameObject
Rigidbody rb = GetComponent<Rigidbody>();

// Get component in children (searches child GameObjects too)
Light light = GetComponentInChildren<Light>();

// Get component in parent (searches up the hierarchy)
Actor actor = GetComponentInParent<Actor>();

// Get all components of a type
MeshRenderer[] renderers = GetComponentsInChildren<MeshRenderer>();
```

**Performance warning:** `GetComponent` is slow. Cache results in `Awake()` or `Start()`, don't call in `Update()`.

### Activating/Deactivating
```csharp
// Disable GameObject (and all children)
gameObject.SetActive(false);

// Enable GameObject
gameObject.SetActive(true);

// Check if active
bool isActive = gameObject.activeSelf;  // This GameObject only
bool isActiveInHierarchy = gameObject.activeInHierarchy;  // Including parents
```

**Example in POSSESSED:**
```csharp
// Enable LocalOnly branch only for your owned actor
localOnlyRoot.SetActive(photonView.IsMine);
```

### Destroying GameObjects
```csharp
// Destroy this GameObject
Destroy(gameObject);

// Destroy another GameObject
Destroy(otherObject);

// Destroy after delay (seconds)
Destroy(gameObject, 3f);

// Multiplayer: Use PhotonNetwork.Destroy for networked objects
PhotonNetwork.Destroy(gameObject);
```

**Important:** Destroy happens at **end of frame**, not immediately. The GameObject still exists for the rest of the current frame.

### Parent-Child Operations
```csharp
// Set parent
transform.SetParent(parentTransform);

// Remove parent (make root-level)
transform.SetParent(null);

// Get parent
Transform parent = transform.parent;

// Get children
int childCount = transform.childCount;
Transform firstChild = transform.GetChild(0);  // by index

// Find child by name
Transform child = transform.Find("ChildName");
```

---

## GameObject Properties

### Name
```csharp
string name = gameObject.name;  // Get name
gameObject.name = "NewName";    // Set name
```

### Tag
```csharp
string tag = gameObject.tag;  // Get tag
gameObject.tag = "Player";    // Set tag

// Check tag
if (gameObject.CompareTag("Enemy")) { ... }  // Faster than tag == "Enemy"
```

**Tags in POSSESSED:**
- `"Wall"` — Used for vision line-of-sight checks
- `"Ground"` — Used for mouse raycast (where to aim)
- `"Untagged"` — Default (Actor camera is Untagged, NOT MainCamera)

### Layer
```csharp
int layer = gameObject.layer;  // Get layer (int)
gameObject.layer = LayerMask.NameToLayer("Wall");  // Set by name

// Check if GameObject is on specific layer
if (gameObject.layer == LayerMask.NameToLayer("Wall")) { ... }
```

**Layers in POSSESSED:**
- `"Wall"` — Blocks vision linecasts
- `"Ground"` — Floor, mouse raycasts hit this

### Static
```csharp
bool isStatic = gameObject.isStatic;  // Check if static
gameObject.isStatic = true;           // Mark as static
```

**Static in POSSESSED:**
Used for NavMesh baking (older Unity workflow). Mark Floor + Walls as Navigation Static before baking.

---

## GameObject in POSSESSED

### Actor GameObject
The **Actor** is the main GameObject representing each player/bot:

**Components on Actor:**
- Transform (position, rotation, scale)
- CharacterController (movement + collision)
- NavMeshAgent (bot pathfinding, disabled for players)
- AudioSource (3D audio for screams/bells)
- AudioLowPassFilter (wall muffling)
- PhotonView (network sync)
- PhotonTransformView (auto-sync position/rotation)
- Actor.cs script (state, cached components)
- TopDownController.cs script (WASD input, if owned)
- VisionSystem.cs script (tier evaluation, if owned)
- BotBrain.cs script (AI pathfinding, if not owned)

**Children of Actor:**
- CapsuleBody (visual mesh)
- EyeAnchor (empty, y=1.5, for vision linecasts)
- NameTag (TextMeshPro, floats above head)
- LocalOnly (branch enabled only for owned actor)
  - Camera (renders scene from this position)
  - FocusLight (spotlight, matches vision cone)
  - PeriphLight (point light, peripheral vision sphere)

### Why This Structure?
- **Parent-child hierarchy:** Camera/lights follow Actor automatically
- **LocalOnly branch:** Only your camera/lights enabled (performance)
- **Single prefab:** All actors use same prefab, spawned at runtime

---

## Common Mistakes

### 1. Forgetting to Cache GetComponent
**Wrong:**
```csharp
void Update() {
    Rigidbody rb = GetComponent<Rigidbody>();  // SLOW: searches every frame
    rb.AddForce(Vector3.up);
}
```

**Right:**
```csharp
Rigidbody rb;
void Awake() {
    rb = GetComponent<Rigidbody>();  // Cache once
}
void Update() {
    rb.AddForce(Vector3.up);  // Fast: use cached reference
}
```

### 2. Using GameObject.Find in Update
**Wrong:**
```csharp
void Update() {
    GameObject player = GameObject.Find("Player");  // VERY SLOW: searches entire scene
}
```

**Right:**
```csharp
GameObject player;
void Start() {
    player = GameObject.Find("Player");  // Find once at start
}
void Update() {
    // Use cached reference
}
```

### 3. Comparing Tags with ==
**Slow:**
```csharp
if (gameObject.tag == "Enemy") { ... }
```

**Fast:**
```csharp
if (gameObject.CompareTag("Enemy")) { ... }
```

### 4. Destroying Networked Objects Incorrectly
**Wrong (multiplayer):**
```csharp
Destroy(networkObject);  // Only destroys on YOUR client
```

**Right (multiplayer):**
```csharp
PhotonNetwork.Destroy(networkObject);  // Destroys on all clients
```

---

## Key Takeaways

1. **GameObject = container, Component = functionality**
2. **Every GameObject has Transform** (position/rotation/scale)
3. **Parent-child hierarchy** makes children follow parent automatically
4. **Cache GetComponent results** — don't call in Update()
5. **GameObject.Find is expensive** — use sparingly, never in Update()
6. **SetActive(false)** disables GameObject and all children
7. **Destroy happens at end of frame**, not immediately

---

## Related Notes

- [[Component]] — Adding functionality to GameObjects
- [[Transform]] — Position, rotation, scale, hierarchy
- [[Prefab]] — Reusable GameObject templates
- [[MonoBehaviour]] — Writing scripts that attach to GameObjects
- [[Actor Script]] — The main GameObject in POSSESSED

---

## Quick Reference: GameObject Methods

| Method | What It Does |
|---|---|
| `GetComponent<T>()` | Find component on this GameObject |
| `GetComponentInChildren<T>()` | Find component in this or child GameObjects |
| `GetComponentInParent<T>()` | Find component in this or parent GameObjects |
| `SetActive(bool)` | Enable/disable this GameObject |
| `Destroy(GameObject)` | Destroy GameObject (end of frame) |
| `Find(string)` | Find GameObject by name (slow) |
| `FindWithTag(string)` | Find GameObject by tag (faster) |
| `CompareTag(string)` | Check if tag matches (fastest) |
| `transform.SetParent(Transform)` | Set parent GameObject |
| `transform.Find(string)` | Find child by name |

---

**Next:** Read [[Component]] to learn how to add functionality to GameObjects.
