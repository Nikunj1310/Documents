# PhotonView

**Category:** Photon PUN 2 Component  
**Namespace:** `Photon.Pun`

---

## What is PhotonView?

The core component for networked GameObjects. Every synchronized object needs one. PhotonView:

1. Assigns a unique **ViewID** to the GameObject (synced across all clients)
2. Tracks **ownership** — which client controls this GameObject
3. Routes **RPCs** to the correct GameObject on other clients
4. Manages **Observables** — components that sync data (like [[PhotonTransformView]])

**Think of it as:** The "network identity card" for a GameObject. Without PhotonView, Photon doesn't know the object exists.

---

## Adding PhotonView

**In Editor:**
1. Select GameObject
2. Add Component → Photon View

**In Prefab (REQUIRED for networked prefabs):**
Actor.prefab **must** have PhotonView to work with `PhotonNetwork.Instantiate("Actor", ...)`.

---

## Key Properties

### ViewID

```csharp
PhotonView pv = GetComponent<PhotonView>();
int viewID = pv.ViewID;  // Unique ID across all clients (e.g., 1001)
```

**What it is:** A unique integer assigned when the GameObject is instantiated. Same ViewID on all clients = same GameObject.

**When RPCs are called:** Photon uses ViewID to find which GameObject to invoke the method on.

**Example:** Client A calls `photonView.RPC("Die", RpcTarget.All)`. Photon sends message: "Call method 'Die' on ViewID 1001". All clients find GameObject with ViewID 1001 and call `Die()` on it.

---

### Ownership (IsMine)

```csharp
bool isMine = photonView.IsMine;  // Do I own this GameObject?
```

**What it means:**
- `true` — **You own this GameObject.** Read input, move it, send RPCs from it.
- `false` — **Someone else owns it.** Just watch. Receive synced position via [[PhotonTransformView]].

**Example in POSSESSED:**
```csharp
void Update() {
    if (!photonView.IsMine) return;  // Not my Actor, skip input
    
    // My Actor: read WASD input and move
    float h = Input.GetAxis("Horizontal");
    float v = Input.GetAxis("Vertical");
    controller.SimpleMove(new Vector3(h, 0, v) * speed);
}
```

**Why this matters:**
Without the `if (!photonView.IsMine) return;` check, **every client** would read input and try to move the GameObject. Result: 8 clients fighting for control = jittering chaos.

---

### Owner

```csharp
Photon.Realtime.Player owner = photonView.Owner;
int ownerActorNumber = owner.ActorNumber;  // 1, 2, 3...
string ownerName = owner.NickName;
```

**What it is:** The Photon Player object representing the client that owns this GameObject.

**ActorNumber:** Unique player ID (1 = first player, 2 = second, etc.). Persistent across scenes. Used for indexing into GhostGame.alive[] in POSSESSED.

---

### Observables

```csharp
// In Inspector: Observables list
```

**What it is:** List of components that automatically sync data across the network.

**Common Observable:**
- [[PhotonTransformView]] — syncs position/rotation/scale

**How to add:**
1. Add PhotonTransformView component to GameObject
2. PhotonView Inspector → Observables section → drag PhotonTransformView into list (or click + and select)

**Without Observable:** Position doesn't sync. GameObject moves on your machine, stays still on others.

---

## Ownership Models

### Creator Ownership (POSSESSED uses this)

```csharp
GameObject obj = PhotonNetwork.Instantiate("Actor", spawnPos, Quaternion.identity);
PhotonView pv = obj.GetComponent<PhotonView>();
// pv.IsMine == true (you spawned it, you own it)
```

**When to use:** Each player spawns their own Actor. You own yours for the entire game.

### Transfer Ownership

```csharp
photonView.TransferOwnership(newOwnerPlayer);
```

**When to use:** Advanced. Object starts owned by one client, later transfers to another.

**POSSESSED does NOT use this.** The Ghost is a virtual tag (`int hostIdx`), not an object that transfers. See [[Ownership]] for why.

---

## RPC Routing

When you call an RPC:
```csharp
photonView.RPC("TakeDamage", RpcTarget.All, 10);
```

Photon:
1. Sends message: "Call 'TakeDamage' on ViewID 1001 with parameter 10"
2. All clients find GameObject with ViewID 1001
3. All clients call `TakeDamage(10)` on that GameObject's script

**Without PhotonView:** Photon doesn't know which GameObject you're talking about. RPC fails silently.

See [[RPC]] for full explanation.

---

## Common Settings (Inspector)

### Owner

Dropdown in Inspector:

| Option | Meaning |
|---|---|
| **Creator (Takeover)** | Spawner owns it initially, others can take over with TransferOwnership |
| **Scene** | Nobody owns it (static world object) |
| **Master Client** | Always owned by Master Client |
| **Fixed** | Ownership never changes, assigned at spawn |

**POSSESSED:** Uses default (Creator Takeover). Each client spawns their own Actor, owns it forever.

### Synchronization

- **Off** — Don't sync (Observable components ignored)
- **Unreliable On Change** — Send when Observable detects change, UDP (fast, packets can be lost)
- **Reliable Delta Compressed** — Send deltas only, TCP-like (slower, guaranteed delivery)

**POSSESSED:** Default (Unreliable On Change). Position changes constantly, losing a packet is fine (next one corrects it).

---

## PhotonView in POSSESSED

**Actor.prefab:**
- Has PhotonView component
- Observables: PhotonTransformView (syncs position/rotation)
- Ownership: Creator Takeover (each client spawns/owns their own Actor)

**Usage:**
```csharp
// Cache in Actor.cs Awake()
photonView = GetComponent<PhotonView>();

// Check ownership in Update()
if (!photonView.IsMine) return;  // Skip input if not mine

// Enable LocalOnly camera/lights only for owned Actor
localOnlyRoot.SetActive(photonView.IsMine);
```

**RPCs:**
ArenaDirector's PhotonView handles kill validation RPCs. Actor script doesn't send RPCs itself — it calls `arenaDirector.photonView.RPC(...)` instead.

---

## Common Mistakes

### 1. Forgetting PhotonView on Prefab

**Wrong:**
- Create prefab without PhotonView
- `PhotonNetwork.Instantiate("Actor", ...)` → silent fail or error

**Right:**
- Add PhotonView component to prefab before saving

### 2. Not Adding Observable to List

**Wrong:**
- Add PhotonTransformView component
- Forget to add it to PhotonView Observables list
- Position doesn't sync

**Right:**
- PhotonView Inspector → Observables → Add PhotonTransformView

### 3. Moving Non-Owned Objects

**Wrong:**
```csharp
void Update() {
    // Moves on your machine only, doesn't sync
    transform.position += Vector3.forward * Time.deltaTime;
}
```

**Right:**
```csharp
void Update() {
    if (!photonView.IsMine) return;  // Only owner moves
    transform.position += Vector3.forward * Time.deltaTime;
    // PhotonTransformView syncs to other clients
}
```

### 4. Using ViewID Before Instantiated

**Wrong:**
```csharp
void Start() {
    int id = photonView.ViewID;  // May be 0 if not instantiated yet
}
```

**Right:**
```csharp
void Start() {
    if (photonView.ViewID == 0) {
        Debug.LogError("Not a networked object!");
        return;
    }
}
```

---

## Quick Reference

| Property/Method | Purpose |
|---|---|
| `ViewID` | Unique network ID |
| `IsMine` | Do I own this? |
| `Owner` | Who owns this? |
| `RPC(methodName, target, params)` | Call method on other clients |
| `TransferOwnership(player)` | Change owner |
| `Observables` | List of syncing components |

---

## Related Notes

- [[PhotonTransformView]] — position/rotation sync component
- [[RPC]] — remote method calls
- [[Ownership]] — who controls what
- [[PhotonNetwork API]] — spawning networked objects
- [[Photon PUN 2 Architecture]] — how it all fits together
- [[Master Client]] — the validating client
- [[Actor Script]] — uses PhotonView in POSSESSED
- [[ArenaDirector]] — handles RPCs via its PhotonView
