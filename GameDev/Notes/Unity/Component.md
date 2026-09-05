# Component

**Category:** Unity Core Concept
**Namespace:** `UnityEngine`
**Base Class:** `Component` (inherits from `Object`)

---

## What a Component Is

A **Component** is a piece of functionality you attach to a [[GameObject]]. The GameObject is the container; Components are what make it *do* something.

**Analogy:** A GameObject is an empty backpack. Components are the things you put in it — a flashlight, a map, a water bottle. The backpack itself does nothing.

---

## The Central Idea: Composition

Unity uses **composition over inheritance**. You do not create a `Player` class that inherits from `Character` which inherits from `Entity`. Instead you take an empty GameObject and *compose* it out of parts:

```
Actor (GameObject)
├── Transform            ← where it is
├── MeshRenderer         ← makes it visible
├── CharacterController  ← lets it move and collide
├── NavMeshAgent         ← lets AI path it around
├── AudioSource          ← lets it emit sound
├── PhotonView           ← makes it exist on the network
├── Actor.cs             ← your state and cached refs
├── TopDownController.cs ← your input handling
└── VisionSystem.cs      ← your vision evaluation
```

Remove `MeshRenderer` → it becomes invisible but still moves and collides. Remove `CharacterController` → it is visible but cannot move. Each Component is independent and swappable.

**Why this matters for POSSESSED:** One Actor prefab serves as both player and bot. The difference is purely which Components are enabled — `CharacterController` for players, `NavMeshAgent` for bots. No separate prefab, no inheritance hierarchy.

---

## Two Kinds of Components

### 1. Built-in Unity Components

Written by Unity, added via **Add Component** in the Inspector.

| Component | Purpose | Note |
|---|---|---|
| [[Transform]] | Position, rotation, scale | **Every** GameObject has one, cannot be removed |
| [[MeshRenderer]] | Draws a mesh | Without it the object is invisible |
| [[Collider]] | Collision shape | BoxCollider, SphereCollider, CapsuleCollider, MeshCollider |
| [[CharacterController]] | Movement + collision, no physics | Used by players |
| [[NavMeshAgent]] | AI pathfinding | Used by bots |
| [[Camera]] | Renders the scene | One per Actor, under LocalOnly |
| [[Light]] | Emits light | Spot (focus cone), Point (peripheral) |
| [[AudioSource 3D]] | Plays audio clips | Spatial Blend 1.0 for 3D |
| [[AudioLowPassFilter]] | Muffles audio | Wall occlusion effect |
| [[PhotonView]] | Network identity | From Photon PUN 2 |
| [[PhotonTransformView]] | Syncs transform | From Photon PUN 2 |

### 2. Your Scripts

Any C# class inheriting [[MonoBehaviour]] **is** a Component. You attach it exactly like a built-in one.

```csharp
public class Actor : MonoBehaviour {
    // This is now a Component. Add Component → Actor.
}
```

**Scripts that do NOT inherit MonoBehaviour are not Components** and cannot be attached to a GameObject. [[GhostGame]] and [[Sight]] are plain C# classes — deliberately, so they carry no Unity dependency.

---

## Adding Components

### In the Editor

1. Select the GameObject in the Hierarchy
2. Inspector → **Add Component** button
3. Search by name

### In Code

```csharp
// Add at runtime
Rigidbody rb = gameObject.AddComponent<Rigidbody>();

// Require it automatically when the script is added
[RequireComponent(typeof(CharacterController))]
public class TopDownController : MonoBehaviour { }
```

`[RequireComponent]` makes Unity add the dependency for you and blocks its removal. See [[Attributes]].

---

## Getting Components

This is the single most common operation in Unity scripting.

```csharp
// On this same GameObject
CharacterController cc = GetComponent<CharacterController>();

// Search children (includes self)
Camera cam = GetComponentInChildren<Camera>();

// Search parents (includes self)
Actor actor = GetComponentInParent<Actor>();

// All of a type, in children
MeshRenderer[] all = GetComponentsInChildren<MeshRenderer>();

// On some other object
Actor other = someGameObject.GetComponent<Actor>();

// Safe pattern when it may be absent
if (TryGetComponent<Rigidbody>(out var body)) {
    body.AddForce(Vector3.up);
}
```

### The Performance Rule

`GetComponent` walks the GameObject's component list every call. It is roughly **100× slower** than reading a cached field.

**Wrong — searches every frame:**
```csharp
void Update() {
    GetComponent<CharacterController>().SimpleMove(move);
}
```

**Right — search once, reuse forever:**
```csharp
CharacterController controller;

void Awake() {
    controller = GetComponent<CharacterController>();
}

void Update() {
    controller.SimpleMove(move);
}
```

This is exactly why [[Actor Script]] caches every reference in `Awake()`. See [[Lifecycle Methods]].

---

## Enabling and Disabling

Most Components have an `enabled` flag. Disabling stops the Component working without destroying it.

```csharp
navAgent.enabled = false;    // stop pathfinding, keep settings
controller.enabled = true;   // resume movement
bodyRenderer.enabled = false; // become invisible
myScript.enabled = false;    // Update() stops being called
```

**Component `enabled` vs GameObject `SetActive`:**

| | Effect |
|---|---|
| `component.enabled = false` | That one Component stops. Everything else on the GameObject keeps running. |
| `gameObject.SetActive(false)` | The **whole** GameObject and all its children stop. |

POSSESSED uses both:
- `navAgent.enabled` / `controller.enabled` — swap movement systems on one Actor ([[NavMeshAgent vs CharacterController]])
- `localOnlyRoot.SetActive(photonView.IsMine)` — disable an entire branch of camera + lights ([[Local-Only Rendering]])

---

## Destroying Components

```csharp
Destroy(GetComponent<Rigidbody>());  // removes just this Component
Destroy(gameObject);                 // removes the whole GameObject
```

Removal happens at end of frame, not immediately.

---

## How Components Communicate

### Cached direct reference (preferred)

```csharp
public class TopDownController : MonoBehaviour {
    Actor myActor;
    void Awake() { myActor = GetComponent<Actor>(); }
    void Update() { if (!myActor.photonView.IsMine) return; }
}
```

### Through a singleton

```csharp
ArenaDirector.Instance.ClaimKill(targetIdx);
```

Used by [[ActorRegistry]], [[ArenaDirector]], [[AudioRouter]], [[UIRouter]]. See [[Singleton Pattern]].

### Through C# events

```csharp
game.OnEvent += HandleGameEvent;   // ArenaDirector subscribes to GhostGame
```

This is how the plain-C# [[GhostGame]] notifies Unity code without ever referencing Unity.

---

## Common Mistakes

### 1. Not caching GetComponent
Covered above. The most frequent Unity performance bug there is.

### 2. Assuming a Component exists
```csharp
// Throws NullReferenceException if there is no Rigidbody
GetComponent<Rigidbody>().AddForce(...);

// Safe
if (TryGetComponent<Rigidbody>(out var rb)) rb.AddForce(...);
```

### 3. Confusing `enabled` with `SetActive`
Disabling a script does not hide the object. Disabling the GameObject stops every script on it, including the one trying to re-enable it later.

### 4. Two Components fighting over the same Transform
[[CharacterController]] and [[NavMeshAgent]] both move the GameObject. Both enabled → visible jitter. Enable exactly one. See [[NavMeshAgent vs CharacterController]].

### 5. Expecting plain C# classes to be attachable
`GhostGame` has no `MonoBehaviour` base, so **Add Component** will never find it. It is instantiated with `new GhostGame(...)` inside [[ArenaDirector]] instead.

---

## Quick Reference

| Operation | Code |
|---|---|
| Add | `gameObject.AddComponent<T>()` |
| Get on self | `GetComponent<T>()` |
| Get in children | `GetComponentInChildren<T>()` |
| Get in parents | `GetComponentInParent<T>()` |
| Get many | `GetComponentsInChildren<T>()` |
| Safe get | `TryGetComponent<T>(out var c)` |
| Disable one Component | `component.enabled = false` |
| Disable whole object | `gameObject.SetActive(false)` |
| Remove | `Destroy(component)` |

---

## Related Notes

- [[GameObject]] — the container Components attach to
- [[Transform]] — the one mandatory Component
- [[MonoBehaviour]] — how your scripts become Components
- [[Lifecycle Methods]] — where to cache references
- [[Attributes]] — `[RequireComponent]`, `[SerializeField]`
- [[Prefab]] — saving a composed GameObject for reuse
- [[Actor Script]] — the component-caching pattern in practice
- [[Block 3 - Actor Prefab]] — assembling all of these together
