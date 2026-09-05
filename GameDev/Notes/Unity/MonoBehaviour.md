# MonoBehaviour

## Overview

**MonoBehaviour** is the base class for all Unity scripts. Any C# script that attaches to a [[GameObject]] **must** inherit from MonoBehaviour.

**Key point:** MonoBehaviour scripts **are [[Component]]s**. You attach them to GameObjects the same way you attach Colliders or Lights.

## What is MonoBehaviour?

```csharp
public class MyScript : MonoBehaviour
{
    // This script can now be added as a Component to GameObjects
}
```

### MonoBehaviour vs Plain C# Class

| Feature | MonoBehaviour | Plain C# |
|---|---|---|
| Attach to GameObject | ✅ Yes | ❌ No |
| Unity lifecycle methods | ✅ Yes (Awake, Start, Update) | ❌ No |
| Serialize in Inspector | ✅ Yes | ❌ No |
| Access Unity APIs | ✅ Yes (transform, gameObject) | ⚠️ Limited |
| Unit testable | ⚠️ Hard (needs Unity) | ✅ Easy |

### When to Use Each

**Use MonoBehaviour when:**
- Script needs to run on a GameObject
- Need Unity lifecycle (Awake, Start, Update)
- Need to interact with Unity scene (transform, colliders, etc.)

**Use Plain C# when:**
- Pure logic, no Unity dependencies
- Want unit testing without Unity
- State machines, math utilities, data structures

**Example from POSSESSED:**
- `Actor.cs` — MonoBehaviour (attached to Actor prefab)
- `GhostGame.cs` — Plain C# (game rules, no Unity)
- `Sight.cs` — Plain C# (pure math, no Unity)

## Lifecycle Methods

Unity automatically calls certain methods on MonoBehaviour scripts at specific times.

### Complete Lifecycle Order

```
GameObject Created / Scene Loaded
    ↓
Awake()           — Initialize self (cache components)
    ↓
OnEnable()        — Called when enabled/activated
    ↓
Start()           — Initialize after all Awake() calls
    ↓
┌─────────────────┐
│  GAME LOOP      │
│                 │
│ FixedUpdate()   │ — Physics (50 times/sec, fixed)
│      ↓          │
│ Update()        │ — Every frame (varies: 30-144 fps)
│      ↓          │
│ LateUpdate()    │ — After all Update() calls
│      ↓          │
│ (repeat)        │
└─────────────────┘
    ↓
OnDisable()       — Called when disabled/deactivated
    ↓
OnDestroy()       — Cleanup (scene unload or Destroy())
```

### Awake()

**When:** GameObject instantiated or scene loaded (before Start)

**Use for:**
- Initializing references to components on **same GameObject**
- Setting up script state
- Runs even if script is disabled

```csharp
void Awake() {
    // Cache components (FAST: same GameObject)
    rb = GetComponent<Rigidbody>();
    collider = GetComponent<Collider>();
    photonView = GetComponent<PhotonView>();
    
    // Register with singleton
    ActorRegistry.Instance.Register(this);
}
```

**Why Awake not Start:** Awake runs before **all** Start() calls. Use when other objects depend on you being initialized.

### Start()

**When:** Before first frame, **after all Awake() calls**

**Use for:**
- Initializing logic that depends on other objects
- Finding other GameObjects in scene
- Runs only if script is enabled

```csharp
void Start() {
    // Find other objects (they're guaranteed to be Awake'd)
    cam = Camera.main;
    arenaDirector = FindObjectOfType<ArenaDirector>();
    
    // Enable/disable based on ownership
    bool isLocal = photonView.IsMine;
    localOnlyRoot.SetActive(isLocal);
}
```

**Awake vs Start:**
- **Awake:** Initialize **self**
- **Start:** Initialize **relationships with others**

### Update()

**When:** Every frame (frame rate varies: 30-144 fps)

**Use for:**
- Input polling
- Non-physics movement
- Animations
- Camera logic

```csharp
void Update() {
    // Input (always in Update, not FixedUpdate)
    if (Input.GetKeyDown(KeyCode.Space)) {
        Jump();
    }
    
    // Non-physics movement (with Time.deltaTime)
    transform.position += direction * speed * Time.deltaTime;
    
    // Vision system (check every frame)
    EvaluateVisionTiers();
}
```

**CRITICAL:** Use `Time.deltaTime` for movement to be frame-rate independent.

```csharp
// WRONG: Speed depends on frame rate
transform.position += Vector3.forward * 5f;

// RIGHT: Frame-rate independent
transform.position += Vector3.forward * 5f * Time.deltaTime;
```

### FixedUpdate()

**When:** Fixed interval (default 50 times/second, regardless of frame rate)

**Use for:**
- Physics calculations
- Rigidbody forces
- Consistent timestep logic

```csharp
void FixedUpdate() {
    // Physics forces (always in FixedUpdate)
    rb.AddForce(direction * force);
    
    // Use Time.fixedDeltaTime (not Time.deltaTime)
    float physicsTime = Time.fixedDeltaTime;  // Always 0.02s
}
```

**Why FixedUpdate:**
- Physics must be deterministic
- Variable Update() timing breaks physics
- FixedUpdate guarantees consistent 0.02s timestep

**Update vs FixedUpdate:**

| Task | Use |
|---|---|
| Input polling | Update() |
| CharacterController movement | Update() |
| Rigidbody forces | FixedUpdate() |
| Vision checks | Update() |
| Camera following | LateUpdate() |

### LateUpdate()

**When:** Every frame, **after all Update() calls**

**Use for:**
- Camera following (camera moves after player movement finalized)
- Billboard rotation (face camera after camera moved)

```csharp
void LateUpdate() {
    // Camera follows player (runs after player's Update())
    transform.position = player.position + offset;
    
    // Billboard faces camera (runs after camera's Update())
    transform.LookAt(cam.transform);
}
```

### OnEnable() / OnDisable()

**When:** GameObject or component enabled/disabled

**Use for:**
- Subscribe/unsubscribe from events
- Start/stop coroutines
- Reset state

```csharp
void OnEnable() {
    // Subscribe to events
    GameManager.OnGameStart += HandleGameStart;
}

void OnDisable() {
    // Unsubscribe (prevents memory leaks)
    GameManager.OnGameStart -= HandleGameStart;
}
```

### OnDestroy()

**When:** GameObject destroyed or scene unloads

**Use for:**
- Cleanup (close connections, release resources)
- Unregister from singletons

```csharp
void OnDestroy() {
    // Unregister from registry
    ActorRegistry.Instance?.Unregister(this);
    
    // Unsubscribe from events
    if (game != null) {
        game.OnEvent -= HandleGameEvent;
    }
}
```

## Built-in Properties

MonoBehaviour provides convenient properties for accessing Unity systems.

```csharp
public class MyScript : MonoBehaviour {
    void Start() {
        // GameObject this script is attached to
        GameObject obj = gameObject;
        
        // Transform of this GameObject
        Transform t = transform;
        
        // Check if GameObject is active
        bool active = gameObject.activeSelf;
        
        // Enable/disable this script component
        enabled = false;
    }
}
```

## GetComponent (Finding Other Components)

```csharp
// Find component on same GameObject
Rigidbody rb = GetComponent<Rigidbody>();

// Find component in children
Camera cam = GetComponentInChildren<Camera>();

// Find component in parent
AudioSource audio = GetComponentInParent<AudioSource>();

// Find all components of type
Collider[] colliders = GetComponents<Collider>();
```

**PERFORMANCE WARNING:** `GetComponent` is slow. Cache results in Awake() or Start().

```csharp
// SLOW: Searches every frame
void Update() {
    Rigidbody rb = GetComponent<Rigidbody>();
    rb.AddForce(Vector3.up);
}

// FAST: Cache once in Awake
private Rigidbody rb;
void Awake() {
    rb = GetComponent<Rigidbody>();
}
void Update() {
    rb.AddForce(Vector3.up);
}
```

## Serialization ([SerializeField])

MonoBehaviour fields can be serialized (saved/loaded by Unity).

```csharp
public class MyScript : MonoBehaviour {
    // Public fields: Visible in Inspector, serialized
    public float speed = 5f;
    
    // Private fields: Hidden, not serialized
    private float health = 100f;
    
    // [SerializeField]: Private but visible in Inspector
    [SerializeField] private int maxHealth = 100;
    
    // [HideInInspector]: Public but hidden
    [HideInInspector] public float internalValue;
}
```

**Best practice:** Use `[SerializeField] private` instead of `public` to maintain encapsulation.

## Coroutines

MonoBehaviour can start coroutines (functions that run over multiple frames).

```csharp
void Start() {
    StartCoroutine(FadeOut());
}

IEnumerator FadeOut() {
    float alpha = 1f;
    while (alpha > 0f) {
        alpha -= Time.deltaTime;
        renderer.material.color = new Color(1, 1, 1, alpha);
        yield return null;  // Wait one frame
    }
}
```

**Use cases:**
- Timed sequences (wait 3 seconds, then do X)
- Animations (fade in/out)
- Async operations

## Invoke (Delayed Calls)

```csharp
// Call method after 3 seconds
Invoke("DoSomething", 3f);

// Call method repeatedly every 1 second
InvokeRepeating("DoSomething", 0f, 1f);

// Cancel all invokes
CancelInvoke();

void DoSomething() {
    Debug.Log("Called!");
}
```

## MonoBehaviour in POSSESSED

### MonoBehaviour Scripts

- [[Actor]] — Per-actor state, cached components
- [[ActorRegistry]] — Singleton tracking all actors
- [[ArenaDirector]] — Game lifecycle, Photon connection
- [[TopDownController]] — Player input
- [[VisionSystem]] — Vision tier evaluation
- [[BotBrain]] — AI pathfinding
- [[AudioRouter]] — Audio event routing
- [[UIRouter]] — UI state management

### Plain C# Scripts

- [[GhostGame]] — Game rules, state machine
- [[Sight]] — Vision tier math
- `IVoiceLayer` — Voice chat interface

## Common Mistakes

### 1. Forgetting Time.deltaTime
```csharp
// WRONG: Frame-rate dependent
transform.position += Vector3.forward * speed;

// RIGHT: Frame-rate independent
transform.position += Vector3.forward * speed * Time.deltaTime;
```

### 2. Physics in Update() Instead of FixedUpdate()
```csharp
// WRONG: Physics in Update (inconsistent)
void Update() {
    rb.AddForce(Vector3.up * 10f);
}

// RIGHT: Physics in FixedUpdate
void FixedUpdate() {
    rb.AddForce(Vector3.up * 10f);
}
```

### 3. GetComponent Every Frame
```csharp
// WRONG: Slow
void Update() {
    GetComponent<Rigidbody>().AddForce(Vector3.up);
}

// RIGHT: Cache in Awake
private Rigidbody rb;
void Awake() { rb = GetComponent<Rigidbody>(); }
void Update() { rb.AddForce(Vector3.up); }
```

### 4. Not Unsubscribing from Events
```csharp
// MEMORY LEAK: Never unsubscribes
void Start() {
    GameManager.OnEvent += HandleEvent;
}

// CORRECT: Unsubscribe in OnDestroy
void OnDestroy() {
    GameManager.OnEvent -= HandleEvent;
}
```

## Related Notes

- [[Component]] — MonoBehaviour is a component type
- [[GameObject]] — What MonoBehaviour attaches to
- [[Lifecycle Methods]] — Detailed lifecycle explanation
- [[Time API]] — Time.deltaTime, Time.time
- [[Actor]] — Example MonoBehaviour in POSSESSED

## Quick Reference

```csharp
public class ExampleScript : MonoBehaviour {
    // Serialize private field
    [SerializeField] private float speed = 5f;
    
    // Cache components
    private Rigidbody rb;
    
    // Initialize self
    void Awake() {
        rb = GetComponent<Rigidbody>();
    }
    
    // Initialize relationships
    void Start() {
        GameManager.OnEvent += HandleEvent;
    }
    
    // Frame update
    void Update() {
        if (Input.GetKey(KeyCode.W)) {
            transform.position += transform.forward * speed * Time.deltaTime;
        }
    }
    
    // Physics update
    void FixedUpdate() {
        rb.AddForce(Vector3.up * 10f);
    }
    
    // Cleanup
    void OnDestroy() {
        GameManager.OnEvent -= HandleEvent;
    }
    
    void HandleEvent() {
        // Event handler
    }
}
```
