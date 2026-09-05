# NavMeshAgent

**Category:** Unity AI Component
**Namespace:** `UnityEngine.AI`
**Package:** AI Navigation (Unity 2022.2+) or built-in (older Unity)

---

## What NavMeshAgent Is

A Component that **automatically moves a GameObject along a NavMesh toward a target**. You set a destination, it pathfinds and walks there, avoiding obstacles.

**Think of it as:** GPS + autopilot. You say "go to that building," and it figures out the route and drives.

---

## How It Works

1. A [[NavMesh]] must be baked in the scene (blue overlay on floor in Scene view)
2. NavMeshAgent samples the NavMesh to find valid paths
3. You call `SetDestination(targetPos)`
4. NavMeshAgent moves the GameObject toward it every frame, following the path

**POSSESSED uses it for:** Bot AI. Bots pick random points and `SetDestination` to wander. See [[BotBrain]].

---

## Key Properties

### Shape

```csharp
NavMeshAgent agent = GetComponent<NavMeshAgent>();

float radius = agent.radius;   // 0.5 for POSSESSED (matches CharacterController radius)
float height = agent.height;   // 2.0 for POSSESSED (matches CharacterController height)
```

**Why these match CharacterController:** Both need the same collision shape. A 0.5m NavMeshAgent in a 0.3m gap will path through it, but CharacterController collision will block. Mismatch = bots think they can go places they cannot.

### Movement

```csharp
float speed = agent.speed;              // m/s (3.5 default in POSSESSED)
float angularSpeed = agent.angularSpeed; // degrees/sec for rotation
float acceleration = agent.acceleration;  // m/s² (how fast it reaches top speed)
```

**Tuning:**
- `speed` too low → bots feel sluggish
- `speed` too high → bots overshoot, slide
- Default 3.5 works for most humanoid movement

### Path State

```csharp
bool hasPath = agent.hasPath;                 // Has a path been calculated?
float remainingDistance = agent.remainingDistance; // Meters to destination
bool pathPending = agent.pathPending;         // Is path calculation in progress?
Vector3 velocity = agent.velocity;            // Current movement speed (m/s vector)
```

**Typical arrival check:**
```csharp
if (!agent.pathPending && agent.remainingDistance <= agent.stoppingDistance) {
    if (agent.velocity.sqrMagnitude == 0f) {
        // Arrived
    }
}
```

**Why check velocity too:** `remainingDistance` can be 0 while the agent is still sliding to a stop (Unity quirk). Velocity confirms full stop.

### Stopping

```csharp
float stoppingDistance = agent.stoppingDistance; // Stop this many meters before target (0.5 default)
bool isStopped = agent.isStopped;                // Manually stopped?

agent.isStopped = true;   // Pause pathfinding
agent.isStopped = false;  // Resume
```

---

## Key Methods

### SetDestination

```csharp
bool success = agent.SetDestination(targetPosition);
```

**What it does:** Calculates a path to `targetPosition` and starts moving.

**Returns:** `true` if path found, `false` if unreachable (off NavMesh, too far, etc.)

**Example (POSSESSED [[BotBrain]]):**
```csharp
Vector3 randomPos = transform.position + Random.insideUnitSphere * 15f;
randomPos.y = transform.position.y;

if (NavMesh.SamplePosition(randomPos, out NavMeshHit hit, 15f, NavMesh.AllAreas)) {
    agent.SetDestination(hit.position);
}
```

**What `NavMesh.SamplePosition` does:** Finds the nearest valid NavMesh point near `randomPos`. Returns false if no NavMesh within search radius. See [[NavMesh API]].

### ResetPath

```csharp
agent.ResetPath();
```

Clears current path. Agent stops moving.

### Warp

```csharp
agent.Warp(newPosition);
```

Teleports agent to `newPosition` without pathfinding. Use for respawning or cutscene placement.

---

## NavMeshAgent vs CharacterController Conflict

**CRITICAL:** Both Components move the GameObject. Both enabled = they fight for control = violent jittering.

**The Rule:**
- **Bots:** NavMeshAgent ON, CharacterController OFF
- **Players:** CharacterController ON, NavMeshAgent OFF

**Implementation (in [[Actor Script]] `Start()`):**
```csharp
bool isBot = !photonView.IsMine;

if (isBot) {
    navAgent.enabled = true;
    controller.enabled = false;
} else {
    navAgent.enabled = false;
    controller.enabled = true;
}
```

**Why this works:** Each Actor exists on all clients. On the owner's machine, `IsMine == true` → CharacterController reads input. On other machines, `IsMine == false` → NavMeshAgent runs bot AI. Different movement system per client, but all are synced via [[PhotonTransformView]].

Full explanation: [[NavMeshAgent vs CharacterController]].

---

## Common Mistakes

### 1. No NavMesh Baked

**Symptom:** Agent doesn't move, Console shows "Failed to create agent because it is not close enough to the NavMesh."

**Fix:** Bake NavMesh (Window → AI → Navigation → Bake, or NavMesh Surface component). See [[NavMesh Baking]].

### 2. Spawn Position Off NavMesh

**Symptom:** Same error as above.

**Fix:** Ensure spawn Y position is on/near floor. Use `NavMesh.SamplePosition` to find nearest valid point:

```csharp
Vector3 spawn = new Vector3(5, 0.5f, 10);
if (NavMesh.SamplePosition(spawn, out NavMeshHit hit, 2f, NavMesh.AllAreas)) {
    spawn = hit.position;  // Corrected to NavMesh surface
}
PhotonNetwork.Instantiate("Actor", spawn, Quaternion.identity);
```

### 3. Not Disabling CharacterController

**Symptom:** Actor jitters violently in place.

**Fix:** `controller.enabled = false` when enabling NavMeshAgent.

### 4. Checking remainingDistance Without pathPending

**Wrong:**
```csharp
if (agent.remainingDistance <= agent.stoppingDistance) {
    // Fires immediately before path calculation finishes
}
```

**Right:**
```csharp
if (!agent.pathPending && agent.remainingDistance <= agent.stoppingDistance) {
    // Only fires after path exists and destination reached
}
```

### 5. Forgetting to Flatten Random Y

**Wrong:**
```csharp
Vector3 random = transform.position + Random.insideUnitSphere * 15f;
agent.SetDestination(random);  // Y can be underground or in the air
```

**Right:**
```csharp
Vector3 random = transform.position + Random.insideUnitSphere * 15f;
random.y = transform.position.y;  // Keep on ground level
if (NavMesh.SamplePosition(random, out NavMeshHit hit, 15f, NavMesh.AllAreas)) {
    agent.SetDestination(hit.position);
}
```

---

## Performance

- **1 agent:** ~0.02ms/frame (negligible)
- **10 agents:** ~0.2ms/frame (fine for POSSESSED scope)
- **100 agents:** Would need LOD (stagger updates, reduce distant agent frequency)

NavMeshAgent is highly optimized. No performance concern for ≤10 bots.

---

## Quick Reference

| Property/Method | Purpose |
|---|---|
| `SetDestination(pos)` | Path to target, start moving |
| `ResetPath()` | Clear path, stop moving |
| `Warp(pos)` | Teleport without pathfinding |
| `speed` | Movement speed (m/s) |
| `remainingDistance` | Meters to destination |
| `pathPending` | Is path calculation in progress? |
| `velocity` | Current movement vector |
| `isStopped` | Pause/resume pathfinding |
| `enabled` | Enable/disable entire component |

---

## In POSSESSED

**Actor.prefab settings:**
- Radius: 0.5
- Height: 2.0
- Speed: 3.5
- Angular Speed: 120
- Acceleration: 8
- **Enabled: OFF by default** (enabled by [[BotBrain]] for non-owned actors only)

**Usage:**
```csharp
// BotBrain.cs
void Start() {
    if (!myActor.photonView.IsMine) {
        myActor.navAgent.enabled = true;
        myActor.controller.enabled = false;
        PickNewDestination();
    }
}

void PickNewDestination() {
    Vector3 randomPos = transform.position + Random.insideUnitSphere * 15f;
    randomPos.y = transform.position.y;
    
    if (NavMesh.SamplePosition(randomPos, out NavMeshHit hit, 15f, NavMesh.AllAreas)) {
        myActor.navAgent.SetDestination(hit.position);
    }
}
```

---

## Related Notes

- [[NavMesh]] — the walkable surface
- [[NavMesh Baking]] — creating the NavMesh
- [[NavMesh API]] — SamplePosition, Raycast, etc.
- [[NavMeshAgent vs CharacterController]] — the conflict and solution
- [[CharacterController]] — player movement alternative
- [[BotBrain]] — the script using NavMeshAgent in POSSESSED
- [[Block 2 - Scene Setup]] — NavMesh baking steps
- [[Block 9 - Bot AI]] — implementing wandering behavior
