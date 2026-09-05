# Block 5 - Actor and Registry Scripts

**Time Budget:** 60 minutes (2:30–3:30)
**Goal:** Write Actor.cs (per-actor state) and ActorRegistry.cs (global actor list) MonoBehaviours

---

## What You're Building

The Unity bridge layer. These MonoBehaviours attach to GameObjects and connect plain C# core logic ([[GhostGame]], [[Sight]]) to the Unity scene.

---

## Prerequisites

- [ ] [[Block 4 - Core Scripts]] complete (GhostGame.cs, Sight.cs compile)
- [ ] [[Block 3 - Actor Prefab]] complete (Actor.prefab exists)
- [ ] Understanding of [[MonoBehaviour]] lifecycle

---

## Checklist

- [ ] Create `Scripts/Game/Actor.cs`
- [ ] Create `Scripts/Game/ActorRegistry.cs`
- [ ] Attach Actor.cs to Actor.prefab root
- [ ] Add ActorRegistry to scene
- [ ] Test compilation (no errors)
- [ ] Play mode: ActorRegistry.Instance exists, registers actors

---

## 1. Actor.cs (35 min)

**Location:** `Assets/Scripts/Game/Actor.cs`

**What it is:** Per-actor state and component cache. Attached to every Actor instance (players + bots).

**Why MonoBehaviour:** Needs Unity lifecycle (Awake/Start/OnDestroy) to cache components, register with registry, enable LocalOnly camera. See [[MonoBehaviour]].

### Key Fields

```csharp
[Header("Identity")]
public int actorIdx = -1;  // Index in GhostGame.alive[] (0, 1, 2...)

[Header("Cached Components")]
public PhotonView photonView;
public CharacterController controller;
public NavMeshAgent navAgent;
public Transform eyeAnchor;
public MeshRenderer bodyRenderer;
public TextMeshPro nameTag;
public GameObject localOnlyRoot;
public Light focusLight, periphLight;
public AudioSource audioSource;
public AudioLowPassFilter lowPassFilter;

[Header("Vision State")]
public VisTier currentTier = VisTier.Hidden;
public Material originalMaterial;
```

**Why cache components:** `GetComponent<T>()` is slow (~100x slower than field access). Cache in Awake(), reuse in Update(). See [[MonoBehaviour]] §Performance.

### Lifecycle Methods

```csharp
void Awake() {
    // Cache all components
    photonView = GetComponent<PhotonView>();
    controller = GetComponent<CharacterController>();
    navAgent = GetComponent<NavMeshAgent>();
    eyeAnchor = transform.Find("EyeAnchor");
    bodyRenderer = transform.Find("CapsuleBody").GetComponent<MeshRenderer>();
    nameTag = GetComponentInChildren<TextMeshPro>();
    localOnlyRoot = transform.Find("LocalOnly").gameObject;
    focusLight = localOnlyRoot.transform.Find("FocusLight").GetComponent<Light>();
    periphLight = localOnlyRoot.transform.Find("PeriphLight").GetComponent<Light>();
    audioSource = GetComponent<AudioSource>();
    lowPassFilter = GetComponent<AudioLowPassFilter>();
    
    // Cache original material for vision tier restoration
    originalMaterial = bodyRenderer.sharedMaterial;
    
    // Register with global registry
    ActorRegistry.Instance.Register(this);
}

void Start() {
    // Enable LocalOnly branch only for owned actor
    if (photonView != null && photonView.IsMine) {
        localOnlyRoot.SetActive(true);
        
        // Disable scene MainCamera (each player uses their own Actor camera)
        Camera mainCam = Camera.main;
        if (mainCam != null) mainCam.enabled = false;
    } else {
        localOnlyRoot.SetActive(false);
    }
    
    // Enable correct movement system
    bool isBot = (photonView == null || !photonView.IsMine);
    navAgent.enabled = isBot;
    controller.enabled = !isBot;
}

void OnDestroy() {
    // Unregister from global registry
    if (ActorRegistry.Instance != null) {
        ActorRegistry.Instance.Unregister(this);
    }
}
```

**Why Awake vs Start:**
- **Awake:** Runs first, used for initialization (caching components, registering)
- **Start:** Runs after all Awakes, used for setup that references other objects (enabling LocalOnly, disabling MainCamera)
- See [[MonoBehaviour]] §Lifecycle.

### Property

```csharp
public Vector3 EyePosition => eyeAnchor != null ? eyeAnchor.position : transform.position + Vector3.up * 1.5f;
```

**Why property:** Vision linecasts and kill range checks use eye-level position (y=1.5m), never ground level (y=0). Ground-level linecasts hit the floor and always return "blocked". See [[Vision Tiers]].

### Full Script Reference

See **GUIDE-FULL.md §7.1** for complete implementation.

---

## 2. ActorRegistry.cs (15 min)

**Location:** `Assets/Scripts/Game/ActorRegistry.cs`

**What it is:** Singleton tracking all Actor instances in the scene.

**Why needed:** [[VisionSystem]], [[ArenaDirector]], [[AudioRouter]] need to loop through all actors. Alternative would be `FindObjectsOfType<Actor>()` every frame (very slow).

### Singleton Pattern

```csharp
public class ActorRegistry : MonoBehaviour {
    public static ActorRegistry Instance { get; private set; }
    
    List<Actor> actors = new List<Actor>();
    public IReadOnlyList<Actor> AllActors => actors;
    
    void Awake() {
        if (Instance != null) {
            Debug.LogError("Multiple ActorRegistry instances! Destroying duplicate.");
            Destroy(gameObject);
            return;
        }
        Instance = this;
    }
    
    public void Register(Actor a) {
        if (!actors.Contains(a)) {
            actors.Add(a);
        }
    }
    
    public void Unregister(Actor a) {
        actors.Remove(a);
    }
    
    public Actor GetActorByIndex(int idx) {
        return actors.FirstOrDefault(a => a.actorIdx == idx);
    }
}
```

**What is Singleton:** Class with exactly one instance, accessible globally via `Instance`. Pattern: check if Instance exists in Awake, destroy duplicates. See [[Singleton Pattern]].

**Why IReadOnlyList:** Prevents external code from modifying the list (`AllActors.Add()` won't compile). Only ActorRegistry can Register/Unregister.

### Add to Scene

1. Hierarchy → Create Empty
2. Name: `ActorRegistry`
3. Add Component → Actor Registry script

**Verification:** Play mode → check Console for "Multiple ActorRegistry instances" error (should not appear).

---

## 3. Attach Scripts to Prefab and Scene

### Actor.cs → Actor.prefab

1. Project → Assets/Resources/Actor.prefab → double-click to open
2. Select root GameObject
3. Inspector → Add Component → Actor (search for "Actor" script)
4. Save prefab (Ctrl+S)

### ActorRegistry → Scene

1. Hierarchy → Create Empty
2. Name: `ActorRegistry`
3. Add Component → Actor Registry script

---

## Verification Checklist

- [ ] Actor.cs compiles (no errors)
- [ ] ActorRegistry.cs compiles (no errors)
- [ ] Actor.cs attached to Actor.prefab root
- [ ] ActorRegistry GameObject in scene hierarchy
- [ ] Play mode: No "Multiple ActorRegistry instances" error
- [ ] Play mode: `ActorRegistry.Instance` is not null (check via Debug.Log or breakpoint)

**Test code (add to ActorRegistry.Start for verification):**
```csharp
void Start() {
    Debug.Log($"ActorRegistry initialized. Actor count: {actors.Count}");
}
```

---

## Common Mistakes

### 1. Forgetting to Register in Awake

**Wrong:**
```csharp
void Start() {
    ActorRegistry.Instance.Register(this);  // Too late, others may have tried to find you
}
```

**Right:**
```csharp
void Awake() {
    ActorRegistry.Instance.Register(this);  // Early registration
}
```

### 2. Not Caching EyeAnchor Transform

**Wrong:**
```csharp
public Vector3 EyePosition => transform.Find("EyeAnchor").position;  // Find every access = slow
```

**Right:**
```csharp
Transform eyeAnchor;  // Cached in Awake
public Vector3 EyePosition => eyeAnchor.position;  // Fast
```

### 3. Enabling Both Movement Systems

**Wrong:**
```csharp
// Both enabled = jittering conflict
navAgent.enabled = true;
controller.enabled = true;
```

**Right:**
```csharp
bool isBot = !photonView.IsMine;
navAgent.enabled = isBot;      // Bots only
controller.enabled = !isBot;   // Players only
```

### 4. Using transform.position for Vision

**Wrong:**
```csharp
Sight.Evaluate(transform.position, ...)  // Ground level, linecast hits floor
```

**Right:**
```csharp
Sight.Evaluate(EyePosition, ...)  // Eye level (y=1.5m)
```

---

## Time Breakdown

| Task | Time |
|---|---|
| Write Actor.cs | 25 min |
| Write ActorRegistry.cs | 10 min |
| Attach to prefab/scene | 5 min |
| Compilation/testing | 10 min |
| Fix errors | 10 min buffer |
| **Total** | **60 min** |

**Behind schedule?** These scripts are **hard gates**. You cannot skip. Cut time from [[Block 10 - Audio]] or [[Block 11 - UI]] instead.

---

## What You Built

- **Actor.cs:** Per-actor state, component cache, lifecycle management
- **ActorRegistry.cs:** Global actor list for fast iteration
- Bridge between plain C# core and Unity scene

**Next:** [[Block 6 - ArenaDirector]] — the orchestrator that connects everything.

---

## Related Notes

- [[MonoBehaviour]] — Unity script base class, lifecycle methods
- [[Singleton Pattern]] — one-instance design
- [[PhotonView]] — ownership tracking (IsMine)
- [[CharacterController]] — player movement
- [[NavMeshAgent]] — bot pathfinding
- [[Vision Tiers]] — why EyePosition matters
- [[Actor Script]] — detailed API reference
- [[Block 4 - Core Scripts]] — previous block
- [[Block 6 - ArenaDirector]] — next block
