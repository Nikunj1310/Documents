# Block 9 - Bot AI

**Time Budget:** 60 minutes (7:00–8:00)
**Goal:** Write BotBrain.cs — autonomous actors that wander randomly

---

## What You're Building

AI controller for non-local actors. Uses [[NavMeshAgent]] to pathfind around walls. Bots wander to random positions, making the game testable solo.

**Design note:** This is a **minimal wandering AI** for jam testing. Ghost-specific behavior (hunting, investigating bells) can be added post-jam. For the 11-hour build, bots just need to move around and be killable.

---

## Prerequisites

- [ ] [[Block 2 - Scene Setup]] complete (NavMesh baked)
- [ ] [[Block 8 - Vision System]] complete
- [ ] Understanding of [[NavMeshAgent]]

---

## Checklist

- [ ] Create `Scripts/Game/BotBrain.cs`
- [ ] Attach to Actor.prefab
- [ ] Test: Non-owned actors wander (visible in multiplayer)
- [ ] Test: No jittering (NavMeshAgent vs CharacterController conflict avoided)
- [ ] Test: Bots pathfind around walls (don't phase through)

---

## Key Responsibilities

### 1. Enable NavMeshAgent, Disable CharacterController

```csharp
void Start() {
    if (!myActor.photonView.IsMine) {
        // This is a bot (not owned by local player)
        myActor.navAgent.enabled = true;
        myActor.controller.enabled = false;
        
        PickNewDestination();
    } else {
        // This is the local player
        enabled = false;  // Disable BotBrain script
    }
}
```

**Why disable BotBrain for local player:** Local player uses [[TopDownController]] (reads Input). Only remote/non-owned actors are bots.

**CRITICAL:** Only ONE movement system active at a time:
- Bots: NavMeshAgent ON, CharacterController OFF
- Players: CharacterController ON, NavMeshAgent OFF
- Both ON = jittering conflict

See [[NavMeshAgent vs CharacterController]].

### 2. Random Wandering

```csharp
void Update() {
    if (!myActor.navAgent.enabled) return;
    
    // Check if reached destination
    if (myActor.navAgent.remainingDistance <= myActor.navAgent.stoppingDistance) {
        if (!myActor.navAgent.hasPath || myActor.navAgent.velocity.sqrMagnitude == 0f) {
            // Wait 2 seconds, then pick new destination
            nextDestinationTime -= Time.deltaTime;
            if (nextDestinationTime <= 0f) {
                PickNewDestination();
                nextDestinationTime = waitTime;
            }
        }
    }
}

void PickNewDestination() {
    // Random position within 15m radius
    Vector3 randomDir = Random.insideUnitSphere * wanderRadius;
    randomDir += transform.position;
    randomDir.y = transform.position.y;  // Keep on same Y level
    
    // Find nearest valid NavMesh point
    if (NavMesh.SamplePosition(randomDir, out NavMeshHit hit, wanderRadius, NavMesh.AllAreas)) {
        myActor.navAgent.SetDestination(hit.position);
    }
}
```

**What is NavMesh.SamplePosition:**
- Finds nearest valid NavMesh point near given position
- `hit.position` is the walkable point
- Returns false if no NavMesh within search radius
- See [[NavMesh API]]

**Why check remainingDistance AND velocity:**
- `remainingDistance` alone can be 0 while bot is still moving (Unity quirk)
- `velocity.sqrMagnitude == 0` confirms bot has fully stopped

### 3. Minimal Attack Behavior (Optional)

For jam testing, bots don't need to attack. But if you want basic behavior:

```csharp
void Update() {
    // ... wandering logic ...
    
    // Look for nearby actors to attack (very simple)
    Actor target = FindNearestVisibleActor();
    if (target != null && Vector3.Distance(transform.position, target.transform.position) < 2.5f) {
        // Face target
        Vector3 dirToTarget = target.transform.position - transform.position;
        dirToTarget.y = 0f;
        transform.rotation = Quaternion.LookRotation(dirToTarget);
        
        // Attempt kill
        if (Time.time > nextAttackTime) {
            nextAttackTime = Time.time + attackCooldown;
            ArenaDirector.Instance.ClaimKill(target.actorIdx);
        }
    }
}

Actor FindNearestVisibleActor() {
    Actor nearest = null;
    float minDist = float.MaxValue;
    
    foreach (Actor other in ActorRegistry.Instance.AllActors) {
        if (other == myActor) continue;
        if (other.currentTier != VisTier.Full) continue;  // Must see target
        
        float dist = Vector3.Distance(transform.position, other.transform.position);
        if (dist < minDist) {
            minDist = dist;
            nearest = other;
        }
    }
    
    return nearest;
}
```

**For 11-hour jam:** Skip attack behavior. Just wandering is enough to test vision/kills.

---

## Full Script Structure

```csharp
using UnityEngine;
using UnityEngine.AI;
using Possessed;

public class BotBrain : MonoBehaviour {
    [Header("Wandering")]
    public float wanderRadius = 15f;
    public float waitTime = 2f;
    
    Actor myActor;
    float nextDestinationTime;
    
    void Awake() {
        myActor = GetComponent<Actor>();
    }
    
    void Start() {
        if (!myActor.photonView.IsMine) {
            myActor.navAgent.enabled = true;
            myActor.controller.enabled = false;
            PickNewDestination();
        } else {
            enabled = false;
        }
    }
    
    void Update() {
        if (!myActor.navAgent.enabled) return;
        
        // Check if reached destination
        if (myActor.navAgent.remainingDistance <= myActor.navAgent.stoppingDistance) {
            if (!myActor.navAgent.hasPath || myActor.navAgent.velocity.sqrMagnitude == 0f) {
                nextDestinationTime -= Time.deltaTime;
                if (nextDestinationTime <= 0f) {
                    PickNewDestination();
                    nextDestinationTime = waitTime;
                }
            }
        }
    }
    
    void PickNewDestination() { /* ... */ }
}
```

See **GUIDE-FULL.md §12.1** for complete implementation.

---

## Attach to Actor.prefab

1. Open Actor.prefab
2. Select root
3. Add Component → Bot Brain
4. Settings:
   - Wander Radius: 15
   - Wait Time: 2
5. Save prefab

---

## Verification Checklist

- [ ] BotBrain.cs compiles
- [ ] Attached to Actor.prefab
- [ ] Play mode (2 clients): Your actor uses CharacterController (WASD), other actor wanders (NavMeshAgent)
- [ ] No jittering (confirms only one movement system active)
- [ ] Bots pathfind around walls (don't walk through them)
- [ ] Bots change direction every few seconds

**Test scenario:**
1. Build WebGL
2. Run Editor + open WebGL build in browser
3. Editor: you control with WASD
4. Browser: bot wanders autonomously
5. Verify bot moves smoothly, paths around walls

---

## Common Mistakes

### 1. Both Movement Systems Enabled

**Symptom:** Actor jitters in place, shaking violently
**Cause:** NavMeshAgent and CharacterController both enabled
**Fix:**
```csharp
navAgent.enabled = true;
controller.enabled = false;  // MUST disable one
```

### 2. NavMesh Not Baked

**Symptom:** Bot doesn't move, console shows "Failed to create agent because it is not close enough to the NavMesh"
**Fix:** Bake NavMesh (Window → AI → Navigation → Bake). See [[Block 2 - Scene Setup]].

### 3. Bot Spawns Off NavMesh

**Symptom:** Same as above
**Fix:** Ensure spawn Y position places bot on/near floor. Actor with CharacterController height=2, center=(0,1,0) should spawn at y=0.5.

### 4. Random.insideUnitSphere Goes Underground

**Symptom:** Bot sometimes gets stuck, navAgent shows error
**Fix:**
```csharp
randomDir.y = transform.position.y;  // Keep on same Y level
```

### 5. Not Checking velocity Before Picking New Destination

**Symptom:** Bot picks new destination while still moving, paths look jerky
**Fix:** Check `velocity.sqrMagnitude == 0f` before calling PickNewDestination.

---

## Performance Notes

- **1 bot:** negligible cost (NavMeshAgent is highly optimized)
- **10 bots:** ~0.2ms/frame (acceptable)
- **100 bots:** Would need LOD (update fewer bots per frame)

For jam scope (≤10 players), no optimization needed.

---

## Time Breakdown

| Task | Time |
|---|---|
| Write wandering logic | 25 min |
| Write movement system toggle | 10 min |
| Attach to prefab | 5 min |
| Test solo | 10 min |
| Test multiplayer (2 clients) | 10 min |
| **Total** | **60 min** |

**Behind schedule?** This is **soft feature**. Can cut if behind. Game works with manual testing (2 human players). But solo testing is much faster, so worth the hour.

---

## What You Built

Autonomous bot AI:
- NavMeshAgent-based pathfinding
- Random wandering behavior
- Movement system conflict avoidance

**Next:** [[Block 10 - Audio System]] — spatial sound feedback

---

## Related Notes

- [[NavMeshAgent]]
- [[NavMesh API]]
- [[NavMeshAgent vs CharacterController]]
- [[CharacterController]]
- [[Vision Tiers]]
- [[Actor Script]]
- [[Block 2 - Scene Setup]]
- [[Block 8 - Vision System]]
- [[Block 10 - Audio System]]
