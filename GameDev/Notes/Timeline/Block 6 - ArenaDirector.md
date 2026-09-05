# Block 6 - ArenaDirector

**Time Budget:** 60 minutes (3:30–4:30)
**Goal:** Write ArenaDirector.cs — the orchestrator connecting GhostGame, Photon, actors, audio, and UI

---

## What You're Building

The central coordinator. ArenaDirector:
1. Connects to Photon Cloud
2. Spawns actors when room joined
3. Initializes GhostGame (Master Client only)
4. Routes kill RPCs from clients to Master for validation
5. Fires events to AudioRouter and UIRouter

**Think of it as:** The game's nervous system connecting all organs.

---

## Prerequisites

- [ ] [[Block 5 - Actor and Registry Scripts]] complete
- [ ] Understanding of [[Photon PUN 2 Architecture]]
- [ ] Understanding of [[Master Client]] validation

---

## Checklist

- [ ] Create `Scripts/Game/ArenaDirector.cs`
- [ ] Add ArenaDirector to scene
- [ ] Add PhotonView component to ArenaDirector GameObject
- [ ] Test: connects to Photon, joins room, spawns actor
- [ ] Test: Master Client initializes GhostGame
- [ ] Test: kill RPC flow works (claim → validate → broadcast)

---

## Key Responsibilities

### 1. Photon Connection Flow

```csharp
Start() → PhotonNetwork.ConnectUsingSettings()
  ↓ OnConnectedToMaster()
  ↓ PhotonNetwork.JoinLobby()
  ↓ OnJoinedLobby()
  ↓ PhotonNetwork.JoinOrCreateRoom("Arena", roomOptions, TypedLobby.Default)
  ↓ OnJoinedRoom()
  ↓ SpawnLocalActor()
  ↓ if (PhotonNetwork.IsMasterClient) InitializeGame()
```

See [[Room and Lobby]] for callback details.

### 2. Actor Spawning

```csharp
void OnJoinedRoom() {
    Vector3 spawnPos = new Vector3(
        Random.Range(-10f, 10f), 
        0.5f, 
        Random.Range(-10f, 10f)
    );
    
    GameObject actorObj = PhotonNetwork.Instantiate(
        "Actor",  // Must be in Resources/ folder
        spawnPos,
        Quaternion.identity
    );
    
    Actor actor = actorObj.GetComponent<Actor>();
    actor.actorIdx = PhotonNetwork.LocalPlayer.ActorNumber - 1;  // 0-indexed
}
```

**Why y=0.5f:** Actor has CharacterController with height 2.0, center (0,1,0). Ground is at y=0. Spawn at 0.5 ensures Actor stands on floor.

**Why ActorNumber - 1:** Photon ActorNumbers are 1-indexed (1, 2, 3...). GhostGame.alive[] is 0-indexed. Subtract 1 to match.

### 3. GhostGame Initialization (Master Client Only)

```csharp
void OnJoinedRoom() {
    // ... spawn actor ...
    
    if (PhotonNetwork.IsMasterClient) {
        // Wait 2 seconds for all clients to spawn
        Invoke(nameof(InitializeGame), 2f);
    }
}

void InitializeGame() {
    int playerCount = PhotonNetwork.CurrentRoom.PlayerCount;
    game = new GhostGame(n => Random.Range(0, n));
    game.OnEvent += HandleGameEvent;
    game.Begin(playerCount);
    
    Debug.Log($"Game initialized with {playerCount} players. First host: {game.hostIdx}");
}
```

**Why 2-second delay:** Photon room join is asynchronous. Wait for all clients to spawn before starting game.

**What is Master Client:** The first player who joined the room. Validates all kills, owns authoritative GhostGame state. See [[Master Client]].

### 4. Kill Validation Pattern

**Client claims kill:**
```csharp
public void ClaimKill(int targetActorIdx) {
    if (!PhotonNetwork.IsConnected) return;
    if (Time.time < nextSwingTime) return;
    
    nextSwingTime = Time.time + swingCooldown;
    photonView.RPC(nameof(ClaimKillRPC), RpcTarget.MasterClient, localActorIdx, targetActorIdx);
}
```

**Master Client validates:**
```csharp
[PunRPC]
void ClaimKillRPC(int killerIdx, int victimIdx, PhotonMessageInfo info) {
    if (!PhotonNetwork.IsMasterClient) return;
    
    // Re-validate everything (client claims are untrusted)
    Actor killer = ActorRegistry.Instance.GetActorByIndex(killerIdx);
    Actor victim = ActorRegistry.Instance.GetActorByIndex(victimIdx);
    
    if (killer == null || victim == null) return;
    
    // Check distance
    float dist = Vector3.Distance(killer.EyePosition, victim.EyePosition);
    bool isPossessed = (game.hostIdx == killerIdx);
    float maxDist = isPossessed ? ghostKillRadius : survivorKillRadius;
    if (dist > maxDist) return;
    
    // Check vision tier (must be Full to kill)
    if (victim.currentTier != VisTier.Full) return;
    
    // Check line-of-sight
    if (Physics.Linecast(killer.EyePosition, victim.EyePosition, wallLayerMask)) return;
    
    // Valid kill — apply to GhostGame
    bool success = game.TryKill(killerIdx, victimIdx, dist);
    if (success) {
        photonView.RPC(nameof(BroadcastKillRPC), RpcTarget.All, victimIdx);
    }
}
```

**Broadcast result to all clients:**
```csharp
[PunRPC]
void BroadcastKillRPC(int victimIdx) {
    Actor victim = ActorRegistry.Instance.GetActorByIndex(victimIdx);
    if (victim != null) {
        PhotonNetwork.Destroy(victim.photonView);  // Removes from all clients
    }
}
```

**Why this pattern:** Prevents cheating. Client sends "I killed X", Master re-checks distance/vision/LOS from authoritative positions, only then applies. See [[Kill Validation Pattern]].

### 5. Event Routing

```csharp
void HandleGameEvent(GameEvent evt, int actorIdx) {
    // Route to audio
    if (AudioRouter.Instance != null) {
        AudioRouter.Instance.PlayEvent(evt, actorIdx);
    }
    
    // Route to UI
    if (UIRouter.Instance != null) {
        switch (evt) {
            case GameEvent.TransferBell:
                if (actorIdx == localActorIdx) {
                    UIRouter.Instance.ShowPossessionUI(true);
                }
                break;
            case GameEvent.Scream:
            case GameEvent.HostExecuted:
                if (actorIdx == localActorIdx) {
                    UIRouter.Instance.ShowDefeatScreen();
                }
                break;
        }
    }
    
    // Check for game end
    if (game.phase == Phase.Results) {
        UIRouter.Instance.ShowResults(game.ending);
    }
}
```

---

## Full Script Structure

```csharp
using Photon.Pun;
using Photon.Realtime;
using UnityEngine;
using Possessed;

public class ArenaDirector : MonoBehaviourPunCallbacks {
    public static ArenaDirector Instance { get; private set; }
    
    [Header("Settings")]
    public float survivorKillRadius = 2.0f;
    public float ghostKillRadius = 1.0f;
    public float swingCooldown = 3f;
    
    [Header("Layers")]
    public LayerMask wallLayerMask;
    
    GhostGame game;
    PhotonView photonView;
    int localActorIdx = -1;
    float nextSwingTime;
    
    void Awake() {
        Instance = this;
        photonView = GetComponent<PhotonView>();
    }
    
    void Start() {
        PhotonNetwork.ConnectUsingSettings();
    }
    
    void Update() {
        if (game != null && PhotonNetwork.IsMasterClient) {
            game.Tick(Time.deltaTime);
        }
    }
    
    // ... Photon callbacks, kill RPCs, event routing ...
}
```

See **GUIDE-FULL.md §7.3** for complete implementation.

---

## Add to Scene

1. Hierarchy → Create Empty
2. Name: `ArenaDirector`
3. Add Component → Arena Director script
4. Add Component → Photon View (required for RPCs)
5. Inspector → Arena Director:
   - Survivor Kill Radius: 2.0
   - Ghost Kill Radius: 1.0
   - Swing Cooldown: 3.0
   - Wall Layer Mask: Select "Wall" layer

---

## Verification Checklist

- [ ] ArenaDirector.cs compiles
- [ ] ArenaDirector GameObject in scene
- [ ] PhotonView component on ArenaDirector
- [ ] Play mode: Console shows "Connected to Photon Cloud"
- [ ] Play mode: Console shows "Joined room"
- [ ] Play mode: Actor spawns (visible in Hierarchy)
- [ ] Play mode (2 clients): Both actors visible to each other
- [ ] Play mode (2 clients): Kill works (victim disappears)

---

## Common Mistakes

### 1. Missing PhotonView on ArenaDirector

**Symptom:** RPC methods never fire, no errors
**Fix:** Add PhotonView component to ArenaDirector GameObject

### 2. Actor Prefab Not in Resources

**Symptom:** `PhotonNetwork.Instantiate("Actor", ...)` returns null
**Fix:** Move Actor.prefab to Assets/Resources/ folder

### 3. Forgetting [PunRPC] Attribute

**Wrong:**
```csharp
void ClaimKillRPC(int killerIdx, int victimIdx) { ... }  // Won't work
```

**Right:**
```csharp
[PunRPC]
void ClaimKillRPC(int killerIdx, int victimIdx) { ... }
```

### 4. Non-Master Client Initializing Game

**Wrong:**
```csharp
void OnJoinedRoom() {
    game = new GhostGame(...);  // Every client creates their own game = desync
}
```

**Right:**
```csharp
void OnJoinedRoom() {
    if (PhotonNetwork.IsMasterClient) {
        game = new GhostGame(...);  // Only Master owns authoritative game
    }
}
```

---

## Time Breakdown

| Task | Time |
|---|---|
| Write connection flow | 15 min |
| Write kill validation RPCs | 20 min |
| Write event routing | 10 min |
| Add to scene, configure | 5 min |
| Test compilation | 5 min |
| Test Photon connection | 5 min |
| **Total** | **60 min** |

---

## What You Built

Central orchestrator connecting:
- Photon Cloud (rooms, spawning)
- GhostGame (state machine)
- Actors (kill validation)
- AudioRouter (event sounds)
- UIRouter (visual feedback)

**Next:** [[Block 7 - Player Input]] — WASD movement, mouse aiming, attack

---

## Related Notes

- [[Photon PUN 2 Architecture]]
- [[Master Client]]
- [[Kill Validation Pattern]]
- [[RPC]]
- [[Room and Lobby]]
- [[MonoBehaviourPunCallbacks]]
- [[GhostGame Script]]
- [[Block 5 - Actor and Registry Scripts]]
- [[Block 7 - Player Input]]
