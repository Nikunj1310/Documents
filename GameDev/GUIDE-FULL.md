# POSSESSED — Complete Master Guide (Beginner-Friendly)

**Game:** Top-down 3D social deception horror. 3–10 players, one hidden Ghost (a virtual tag) possessing one player. Every kill transfers the Ghost randomly. Kill the current host to win, or the Ghost survives until only 2 remain.

**Build target:** WebGL (plays in browser). Photon PUN 2 for multiplayer.

**Camera setup (FINAL):** Camera is a **child of the Actor prefab under LocalOnly**. No separate CameraRig GameObject. Camera follows automatically via parent-child hierarchy.

**For Unity beginners:** This guide explains *what* every component does, *why* we use it instead of alternatives, and *how* it connects. Read **UNITY-PHOTON-FUNDAMENTALS.md** first for GameObject/Component/MonoBehaviour basics.

---

## Table of Contents

1. [Game Rules (What You're Building)](#1-game-rules)
2. [Architectural Decisions](#2-architectural-decisions)
3. [Project Setup](#3-project-setup)
4. [Scene Setup](#4-scene-setup)
5. [Actor Prefab](#5-actor-prefab)
6. [Core Scripts (Plain C#)](#6-core-scripts)
7. [Game Layer Scripts (MonoBehaviours)](#7-game-layer-scripts)
8. [Vision System](#8-vision-system)
9. [Audio System](#9-audio-system)
10. [UI System](#10-ui-system)
11. [Photon Multiplayer](#11-photon-multiplayer)
12. [Bot AI](#12-bot-ai)
13. [Build & Submit](#13-build--submit)

---

## 1. Game Rules (What You're Building)

### 1.1 The Ghost (Virtual Entity)

- **One Ghost exists** — it's NOT a physical character, just a **tag/state** on one player
- **Possessed player knows they're possessed** — UI shows "YOU ARE POSSESSED" + countdown timer
- **Other players don't know who's possessed** — they must deduce from behavior and audio cues
- **The Ghost is invisible** — no visual indicator to other players (only the possessed player sees the UI)

### 1.2 Kill Mechanics

**How to kill:**
- Left-click while targeting someone within your vision cone
- **Survivor kill radius:** 2.0m (normal players)
- **Ghost kill radius:** 1.0m (possessed player must get closer)
- **Must have Full vision tier** (inside cone, range, clear line-of-sight)
- **3-second cooldown** per swing

**What happens on kill:**

| Victim | Result |
|---|---|
| **Non-possessed player** | Victim dies (scream from their position) → Ghost transfers **randomly** to another living player (never the killer) → New host sees possession UI + timer starts |
| **Possessed player** | Victim dies → **Ghost is destroyed** (shatter sound) → All alive players win |

### 1.3 Possession Timer

- **Default: 90 seconds** (host-configurable)
- Starts the moment you become possessed
- **Warning at 15 seconds remaining** (rising whisper sound)
- **If timer expires:** You are killed by the Ghost itself (scream from your position) → Ghost transfers randomly to another player

**Key point:** Sacrifice (letting timer expire) does NOT destroy the Ghost. It just kills you and relocates the Ghost. You lose either way as the host.

### 1.4 Win Conditions

| Situation | Outcome | Winners |
|---|---|---|
| Possessed player killed by another player | **Survivors Win** | All alive players split prize pool |
| A kill leaves exactly **2 alive** (one is Ghost) | **Ghost Wins** | The player who made that kill gets 100% of pool |
| Timer expires leaving exactly **2 alive** | **No Winner** | Nobody (punishes passive play) |

**Dead players always lose** — even if survivors win later, you don't share the prize if you died earlier.

### 1.5 Vision Tiers (Targeting Rules)

| Tier | Condition | What You See | Can Kill? |
|---|---|---|---|
| **Full** | Inside focus cone + range + clear line-of-sight | Normal model + nametag | ✅ YES |
| **Silhouette** | Within 4.5m peripheral sphere, any direction, clear LOS | Black silhouette, no nametag | ❌ NO |
| **Hidden** | Out of range OR wall blocking | Invisible | ❌ NO |

**Focus cone:**
- **40° half-angle, 14m range** while moving
- Opens to **62° half-angle, 16m range** after standing still ~0.5s
- Closes fast when moving (lerp 12/s), opens slow when still (lerp 4/s)

**Kill validation:** Client claims kill → Master Client re-checks (range, cone, LOS, cooldown) → Broadcasts result. Clients never self-declare kills.

---

## 2. Architectural Decisions

### 2.1 Plain C# Core (GhostGame, Sight)

**What:** Game rules live in plain C# classes with zero Unity dependencies.

**Why:**
- Testable without running Unity
- Same logic works for mobile version later (different input, same rules)
- Easier to reason about (state changes in one place)

**GhostGame.cs:** Possession state, timer, kill validation, win conditions
**Sight.cs:** Vision tier math (cone angles, distances, pure Vector3 math)

### 2.2 Camera Under Actor Prefab (LocalOnly)

**What:** Camera is a child of Actor → LocalOnly → Camera. No separate CameraRig GameObject.

**Why:**
- **Automatic following:** Camera moves with Actor (parent-child hierarchy), zero scripts needed
- **Multiplayer-safe:** `localOnlyRoot.SetActive(photonView.IsMine)` enables only your camera
- **Simpler:** No CameraFollow script, no target switching on possession

**When you "possess" someone in this 3D version:**
- There's no possession mechanic for players to take over bots
- The Ghost tag just moves between players randomly on kills
- Your camera stays with your Actor the whole time

### 2.3 Local-Only Rendering (Lights)

**What:** Each Actor has FocusLight + PeriphLight under LocalOnly branch. Only your local Actor enables these.

**Why:**
- **Performance:** 1 actor with 2 shadow lights = fine. 8 actors with 16 shadow lights = slideshow in WebGL
- **Logic:** Only you need to see YOUR vision cone. Other players are just lit capsules

### 2.4 Raycast Vision, Not Collider Triggers

**What:** `Sight.Evaluate()` checks cone angle + distance, then `Physics.Linecast` checks walls.

**Why:**
- Triggers can't detect wall occlusion (line-of-sight)
- Hard to implement cone angles with triggers
- Raycasts give precise eye-to-target visibility

### 2.5 Secret hostIdx (Master Client Only)

**What:** Only Master Client knows `GhostGame.hostIdx`. Possessed player learns they're possessed via targeted RPC.

**Why:**
- Hidden information = deception gameplay
- Audio cues (transfer bell) are the only hints
- Prevents "everyone chases the highlighted player"

---

## 3. Project Setup

### 3.1 Unity Version & Template

- **Unity 2021.3 LTS or newer** (use what you already have installed)
- **3D URP template** (Universal Render Pipeline)
  - **Why URP:** Easier post-processing (vignette, film grain), better WebGL performance
  - **Trade-off:** Slightly slower on very old hardware vs Built-in RP

### 3.2 Import Photon PUN 2

**Why Photon PUN 2 not Unity Netcode (NGO):**
- ✅ WebGL multiplayer works (WebSocket fallback)
- ✅ Room codes out-of-the-box
- ✅ Works over internet (Photon Cloud relay)
- ❌ NGO doesn't support WebGL hosting, requires Unity Relay setup, LAN-only by default

**Steps:**
1. Asset Store → Search "Photon PUN 2 FREE" → Import
2. Register at photonengine.com/pun → Create app (type: Pun) → Copy App ID
3. Unity: Window → Photon Unity Networking → PUN Wizard → Setup Project → Paste App ID
4. Verify `PhotonServerSettings` asset exists in `Assets/Photon/PhotonUnityNetworking/Resources/`

### 3.3 Install AI Navigation Package

Window → Package Manager → Unity Registry → Search "AI Navigation" → Install

**Why:** NavMesh baking for bot pathfinding. Without this, bots can't navigate around walls.

### 3.4 Project Settings

**Ambient Lighting (makes shadows matter):**
- Window → Rendering → Lighting → Environment tab
- **Ambient Intensity: 0** (or very low ~0.1)
- **Why:** Unlit areas should be dark. Horror aesthetic requires pitch-black shadows.

**Custom Layers:**
- Edit → Project Settings → Tags and Layers → Layers
- Add: `Wall` (for vision line-of-sight checks)
- Add: `Ground` (for floor, mouse raycasts)

### 3.5 Folder Structure

```
Assets/
├─ Scenes/
│  └─ Arena.unity
├─ Scripts/
│  ├─ Core/               (plain C#)
│  │  ├─ GhostGame.cs
│  │  ├─ Sight.cs
│  │  └─ IVoiceLayer.cs
│  ├─ Game/               (MonoBehaviours)
│  │  ├─ Actor.cs
│  │  ├─ ActorRegistry.cs
│  │  ├─ ArenaDirector.cs
│  │  ├─ TopDownController.cs
│  │  ├─ VisionSystem.cs
│  │  └─ BotBrain.cs
│  ├─ Multiplayer/
│  │  └─ PhotonCallbacks.cs
│  ├─ Audio/
│  │  └─ AudioRouter.cs
│  └─ UI/
│     └─ UIRouter.cs
├─ Resources/             (CRITICAL: Photon prefabs MUST be here)
│  └─ Actor.prefab
├─ Audio/
│  ├─ scream.wav
│  ├─ bell.wav
│  ├─ crescendo.wav
│  └─ shatter.wav
└─ Materials/
   └─ Silhouette.mat      (unlit black)
```

### 3.6 Test WebGL Build Now

File → Build Settings → WebGL → Switch Platform → Build

**Why now:** Discovering a broken build pipeline at hour 10 kills your jam submission.

---

## 4. Scene Setup

### 4.1 Floor

GameObject → 3D Object → **Cube** (NOT Plane)
- Scale: (30, 0.1, 30)
- Position: (0, 0, 0)
- Layer: Ground
- Material: Dark grey unlit

**Why Cube not Plane:** CharacterController.isGrounded requires BoxCollider/SphereCollider/CapsuleCollider. Plane uses MeshCollider (doesn't work reliably).

### 4.2 Walls

GameObject → 3D Object → Cube (×4-8 for room boundaries)
- Scale: (10, 2.5, 0.2) or similar (2.5m tall)
- Layer: Wall
- Material: Mid-grey unlit
- Position to form 4-5 rooms with doorways (2-3 unit gaps)

**Why 2.5m tall:** Above eye level (1.5m) so vision linecasts hit walls, not too oppressive.

### 4.3 NavMesh Baking

**Path A (Unity 2022.2+ with AI Navigation package installed):**
1. Create empty GameObject → Name: `NavMesh`
2. Add Component → NavMesh Surface
3. Settings: Collect Objects: All, Use Geometry: Render Meshes, Agent Type: Humanoid
4. Click **Bake** button on component
5. **Verify:** Blue overlay on floor in Scene view

**Path B (Older Unity, no AI Navigation package):**
1. Select Floor + Walls
2. Inspector → Static dropdown → Navigation Static
3. Window → AI → Navigation → Bake tab → Bake
4. **Verify:** Blue overlay on floor

**Common issue:** Blue overlay insets from walls → Normal (agent radius 0.5m). Bots can't hug walls exactly.

### 4.4 Lighting

**Directional Light** (already in scene):
- Rotation: (50, -30, 0) for angled top-down light
- Intensity: ~1
- Color: Slightly warm white

**Environment:**
- Window → Rendering → Lighting → Environment
- Skybox: Default or black
- Ambient Intensity: 0 (pitch black shadows)

---

## 5. Actor Prefab

### 5.1 Hierarchy

```
Actor (root)
├─ CapsuleBody (visual mesh)
├─ EyeAnchor (empty, y=1.5, for vision linecasts)
├─ NameTag (TextMeshPro world-space text, y=2.5)
└─ LocalOnly (enabled ONLY for photonView.IsMine)
   ├─ Camera (position: 0, 18, -8 relative to Actor, rotation: 70, 0, 0)
   ├─ FocusLight (Spot Light, shadows ON)
   └─ PeriphLight (Point Light, range 4.5, shadows ON)
```

**CRITICAL:** Camera is a **child of Actor → LocalOnly**, NOT a separate GameObject in scene. No CameraRig. No CameraFollow script needed.

### 5.2 Components on Actor Root

| Component | Settings | Why |
|---|---|---|
| **CharacterController** | Radius 0.5, Height 2.0, Center (0,1,0) | Player movement, handles collision without physics |
| **NavMeshAgent** | Speed 3.5, Radius 0.5, Height 2.0, **Disabled by default** | Bot pathfinding (enable for bots, disable for players) |
| **AudioSource** | Spatial Blend 1.0, Play On Awake OFF | 3D positional audio for screams/bells |
| **AudioLowPassFilter** | Cutoff 22000 Hz default | Muffles to 800 Hz when walls block LOS |
| **PhotonView** | Observables: PhotonTransformView | Network sync, ownership tracking |
| **PhotonTransformView** | Sync Position ON, Rotation ON, Interpolate ON | Auto-syncs transform across clients |
| **Actor.cs** | — | Your script (state, tier, hostIdx) |
| **TopDownController.cs** | Enabled for photonView.IsMine only | WASD + mouse input |
| **VisionSystem.cs** | Enabled for photonView.IsMine only | Tier evaluation + rendering |
| **BotBrain.cs** | Enabled for !photonView.IsMine only | AI pathfinding |

**CRITICAL NavMeshAgent + CharacterController conflict:**
- **Bots:** NavMeshAgent ON, CharacterController OFF
- **Players:** CharacterController ON, NavMeshAgent OFF
- **Both ON = jittering** (they fight for control)

### 5.3 CapsuleBody

GameObject → 3D Object → Capsule
- Child of Actor root
- Position: (0, 0, 0) relative to Actor
- Remove CapsuleCollider (CharacterController handles collision)
- **What gets hidden/shown:** This mesh's MeshRenderer is controlled by vision tiers

### 5.4 EyeAnchor

Create Empty child of Actor
- Position: (0, 1.5, 0) relative to Actor
- **CRITICAL:** All vision linecasts and range checks use `EyeAnchor.position`, never `transform.position`
- **Why:** Ground-level linecasts (y=0) hit the floor and always return "blocked"

### 5.5 NameTag

GameObject → UI → Text - TextMeshPro (world-space, not Canvas-based)
- Position: (0, 2.5, 0) relative to Actor
- Font Size: ~2, Alignment: Center/Middle
- Add Billboard script:
```csharp
using UnityEngine;
public class Billboard : MonoBehaviour {
    Camera cam;
    void Start() { cam = Camera.main; }
    void LateUpdate() { 
        if (cam) 
        { 
	        transform.LookAt(cam); 
	        transform.Rotate(0, 180, 0); 
	    }
    }
}
```
- **Enabled only at VisTier.Full** (see VisionSystem)

### 5.6 LocalOnly Branch

Create Empty child of Actor → Name: `LocalOnly`
- **Disabled by default** in prefab
- Enabled at runtime: `localOnlyRoot.SetActive(photonView.IsMine);`

**Camera (child of LocalOnly):**
- Add Camera component
- Position: (0, 18, -8) relative to Actor
- Rotation: (70, 0, 0) — pitch down 70°
- Tag: **Untagged** (NOT MainCamera)
- Clear Flags: Skybox
- Field of View: ~60

**Why child of Actor:** Camera moves automatically with Actor (parent-child hierarchy). No script needed.

**FocusLight (child of LocalOnly):**
- Type: Spot Light
- Range: 14, Spot Angle: 80 (animated by VisionSystem)
- Intensity: 3-5
- Shadows: Soft Shadows
- Color: White

**PeriphLight (child of LocalOnly):**
- Type: Point Light
- Range: 4.5
- Intensity: 1-2
- Shadows: Soft Shadows

### 5.7 Save as Prefab

1. Drag Actor from Hierarchy → **Assets/Resources/** folder
2. **MUST be in Resources/** — `PhotonNetwork.Instantiate("Actor", ...)` only works if prefab is here
3. Delete Actor from Hierarchy (spawned at runtime)

---

## 6. Core Scripts (Plain C#)

### 6.1 Core/GhostGame.cs

**What:** State machine for possession, kills, timers, win conditions. Zero Unity dependencies.

```csharp
using System;
using System.Linq;

namespace Possessed {

public enum GameEvent { Scream, TransferBell, TimerWarning, HostExecuted, GhostShattered }
public enum Phase { Lobby, Hunt, Results }
public enum Ending { None, SurvivorsWin, GhostWins, NoWinner }

public class GhostGame {
    public Phase phase = Phase.Lobby;
    public Ending ending = Ending.None;
    public int hostIdx = -1;              // who is currently possessed
    public bool[] alive;
    public float possessTimer;
    public int payoutKillerIdx = -1;      // for GhostWins: who made the kill that left 2 alive

    // Settings
    public float possessDuration = 90f;
    public bool transferCanHitKiller = false;  // false = killer excluded from random transfer
    public float survivorKillRadius = 2.0f;
    public float ghostKillRadius = 1.0f;

    public event Action<GameEvent, int> OnEvent;  // (eventType, actorIdx)

    Random rng = new Random();

    public void Begin(int playerCount) {
        phase = Phase.Hunt;
        alive = Enumerable.Repeat(true, playerCount).ToArray();
        hostIdx = rng.Next(playerCount);
        possessTimer = possessDuration;
        OnEvent?.Invoke(GameEvent.TransferBell, hostIdx);  // initial possession bell
    }

    public void Tick(float dt) {
        if (phase != Phase.Hunt) return;

        if (hostIdx >= 0 && alive[hostIdx]) {
            possessTimer -= dt;

            if (possessTimer <= 15f && possessTimer + dt > 15f) {
                OnEvent?.Invoke(GameEvent.TimerWarning, hostIdx);
            }

            if (possessTimer <= 0f) {
                // Timer expired — Ghost kills host
                OnEvent?.Invoke(GameEvent.HostExecuted, hostIdx);
                alive[hostIdx] = false;
                int aliveCount = alive.Count(a => a);

                if (aliveCount == 2) {
                    // 2 left after timer expiry = NoWinner (punish passive play)
                    ending = Ending.NoWinner;
                    phase = Phase.Results;
                } else if (aliveCount > 2) {
                    // Transfer Ghost randomly
                    TransferGhost(hostIdx);
                } else {
                    // 1 or 0 left = survivors win by default (edge case)
                    ending = Ending.SurvivorsWin;
                    phase = Phase.Results;
                }
            }
        }
    }

    public bool TryKill(int killerIdx, int victimIdx, float distance) {
        if (phase != Phase.Hunt) return false;
        if (!alive[killerIdx] || !alive[victimIdx]) return false;
        if (killerIdx == victimIdx) return false;

        // Range check
        float maxRange = (killerIdx == hostIdx) ? ghostKillRadius : survivorKillRadius;
        if (distance > maxRange) return false;

        // Valid kill
        OnEvent?.Invoke(GameEvent.Scream, victimIdx);
        alive[victimIdx] = false;

        if (victimIdx == hostIdx) {
            // Killed the Ghost host → Ghost destroyed
            OnEvent?.Invoke(GameEvent.GhostShattered, hostIdx);
            ending = Ending.SurvivorsWin;
            phase = Phase.Results;
            hostIdx = -1;
        } else {
            // Killed non-host → Ghost transfers randomly
            int aliveCount = alive.Count(a => a);
            if (aliveCount == 2) {
                // Exactly 2 left (one is Ghost) → Ghost wins
                ending = Ending.GhostWins;
                payoutKillerIdx = killerIdx;
                phase = Phase.Results;
            } else {
                TransferGhost(killerIdx);
            }
        }

        return true;
    }

    void TransferGhost(int excludeIdx) {
        var candidates = Enumerable.Range(0, alive.Length)
            .Where(i => alive[i] && (transferCanHitKiller || i != excludeIdx))
            .ToList();

        if (candidates.Count == 0) {
            // No valid targets (shouldn't happen, but handle gracefully)
            hostIdx = -1;
            ending = Ending.SurvivorsWin;
            phase = Phase.Results;
            return;
        }

        hostIdx = candidates[rng.Next(candidates.Count)];
        possessTimer = possessDuration;
        OnEvent?.Invoke(GameEvent.TransferBell, hostIdx);
    }

    public bool IsAlive(int idx) => idx >= 0 && idx < alive.Length && alive[idx];
    public int AliveCount() => alive.Count(a => a);
}

}
```

**Key methods:**
- `Begin(playerCount)` — Start match, pick first Ghost host randomly
- `Tick(dt)` — Update possession timer, fire warning/expiry events
- `TryKill(killerIdx, victimIdx, distance)` — Validate + apply kill, return true if success
- `TransferGhost(excludeIdx)` — Pick random living player (optionally excluding killer), reset timer, fire TransferBell event

### 6.2 Core/Sight.cs

**What:** Pure math for vision tier evaluation. Returns Hidden/Silhouette/Full based on cone angle, distance, movement state.

```csharp
using UnityEngine;

namespace Possessed {

public enum VisTier { Hidden, Silhouette, Full }

public static class Sight {
    // Focus cone (narrow when moving, wide when still)
    const float CONE_MOVING_HALF_ANGLE = 40f;
    const float CONE_STILL_HALF_ANGLE = 62f;
    const float CONE_MOVING_RANGE = 14f;
    const float CONE_STILL_RANGE = 16f;

    // Peripheral sphere
    const float PERIPH_RADIUS = 4.5f;

    public static VisTier Evaluate(Vector3 observerPos, Vector3 observerFwd, Vector3 targetPos, bool observerMoving) {
        Vector3 toTarget = targetPos - observerPos;
        float distance = toTarget.magnitude;
        if (distance < 0.01f) return VisTier.Hidden;  // same position

        Vector3 dirToTarget = toTarget / distance;
        float angle = Vector3.Angle(observerFwd, dirToTarget);

        // Check focus cone
        float coneAngle = observerMoving ? CONE_MOVING_HALF_ANGLE : CONE_STILL_HALF_ANGLE;
        float coneRange = observerMoving ? CONE_MOVING_RANGE : CONE_STILL_RANGE;

        if (angle <= coneAngle && distance <= coneRange) {
            return VisTier.Full;
        }

        // Check peripheral sphere
        if (distance <= PERIPH_RADIUS) {
            return VisTier.Silhouette;
        }

        return VisTier.Hidden;
    }
}

}
```

**How it connects:** VisionSystem calls this for every other actor every frame, then does `Physics.Linecast` to check walls. If linecast hits wall → downgrade to Hidden.

### 6.3 Core/IVoiceLayer.cs

**What:** Interface for voice chat systems (Vivox, stub, etc.). GhostGame calls `MuteLocal(true)` when you die.

```csharp
namespace Possessed {

public interface IVoiceLayer {
    void MuteLocal(bool muted);
}

// Stub implementation (no voice chat)
public class VoiceStub : IVoiceLayer {
    public void MuteLocal(bool muted) { /* no-op */ }
}

}
```

---

## 7. Game Layer Scripts (MonoBehaviours)

### 7.1 Game/Actor.cs

**What:** Per-actor state and cached component references. Attached to Actor prefab root.

```csharp
using UnityEngine;
using Photon.Pun;
using TMPro;

namespace Possessed {

public class Actor : MonoBehaviour {
    [Header("Cached References")]
    public int actorIdx = -1;                    // index in GhostGame.alive[]
    public PhotonView photonView;
    public CharacterController controller;
    public NavMeshAgent navAgent;
    public Transform eyeAnchor;
    public MeshRenderer bodyRenderer;
    public TextMeshPro nameTag;
    public GameObject localOnlyRoot;
    public Light focusLight;
    public Light periphLight;
    public AudioSource audioSource;
    public AudioLowPassFilter lowPassFilter;

    [Header("Vision State")]
    public VisTier currentTier = VisTier.Hidden;

    void Awake() {
        photonView = GetComponent<PhotonView>();
        controller = GetComponent<CharacterController>();
        navAgent = GetComponent<NavMeshAgent>();
        audioSource = GetComponent<AudioSource>();
        lowPassFilter = GetComponent<AudioLowPassFilter>();

        eyeAnchor = transform.Find("EyeAnchor");
        bodyRenderer = transform.Find("CapsuleBody").GetComponent<MeshRenderer>();
        nameTag = transform.Find("NameTag").GetComponent<TextMeshPro>();
        localOnlyRoot = transform.Find("LocalOnly").gameObject;
        focusLight = localOnlyRoot.transform.Find("FocusLight").GetComponent<Light>();
        periphLight = localOnlyRoot.transform.Find("PeriphLight").GetComponent<Light>();

        ActorRegistry.Instance.Register(this);
    }

    void Start() {
        bool isLocal = photonView != null && photonView.IsMine;
        localOnlyRoot.SetActive(isLocal);

        if (isLocal) {
            // Disable scene MainCamera (we use Actor camera now)
            var mainCam = Camera.main;
            if (mainCam != null) mainCam.gameObject.SetActive(false);
        }
    }

    void OnDestroy() {
        ActorRegistry.Instance?.Unregister(this);
    }

    public Vector3 EyePosition => eyeAnchor != null ? eyeAnchor.position : transform.position + Vector3.up * 1.5f;
}

}
```

### 7.2 Game/ActorRegistry.cs

**What:** Singleton tracking all Actor instances. Provides `AllActors` list for VisionSystem, ArenaDirector, etc.

```csharp
using System.Collections.Generic;
using UnityEngine;

namespace Possessed {

public class ActorRegistry : MonoBehaviour {
    public static ActorRegistry Instance { get; private set; }

    List<Actor> actors = new List<Actor>();
    public IReadOnlyList<Actor> AllActors => actors;

    void Awake() {
        if (Instance != null && Instance != this) {
            Destroy(gameObject);
            return;
        }
        Instance = this;
    }

    public void Register(Actor a) {
        if (!actors.Contains(a)) actors.Add(a);
    }

    public void Unregister(Actor a) {
        actors.Remove(a);
    }
}

}
```

**Add to scene:** Create empty GameObject → Name: `ActorRegistry` → Add Component: ActorRegistry script.

### 7.3 Game/ArenaDirector.cs

**What:** Owns GhostGame instance, connects to Photon, spawns actors, routes events to audio/UI, handles kill RPCs.

```csharp
using UnityEngine;
using Photon.Pun;
using Photon.Realtime;

namespace Possessed {

public class ArenaDirector : MonoBehaviourPunCallbacks {
    GhostGame game;
    Actor localActor;
    AudioRouter audioRouter;
    UIRouter uiRouter;

    void Start() {
        audioRouter = FindObjectOfType<AudioRouter>();
        uiRouter = FindObjectOfType<UIRouter>();

        // Connect to Photon
        PhotonNetwork.ConnectUsingSettings();
    }

    public override void OnConnectedToMaster() {
        PhotonNetwork.JoinLobby();
    }

    public override void OnJoinedLobby() {
        RoomOptions opts = new RoomOptions { MaxPlayers = 8 };
        PhotonNetwork.JoinOrCreateRoom("Arena", opts, TypedLobby.Default);
    }

    public override void OnJoinedRoom() {
        // Spawn local actor
        Vector3 spawnPos = new Vector3(Random.Range(-12f, 12f), 0.5f, Random.Range(-12f, 12f));
        GameObject actorObj = PhotonNetwork.Instantiate("Actor", spawnPos, Quaternion.identity);
        localActor = actorObj.GetComponent<Actor>();
        localActor.actorIdx = PhotonNetwork.LocalPlayer.ActorNumber - 1;  // 0-indexed

        // Master Client initializes GhostGame
        if (PhotonNetwork.IsMasterClient) {
            game = new GhostGame();
            game.OnEvent += OnGameEvent;
            game.Begin(PhotonNetwork.PlayerList.Length);
        }
    }

    void Update() {
        if (game != null && PhotonNetwork.IsMasterClient) {
            game.Tick(Time.deltaTime);
        }
    }

    void OnGameEvent(GameEvent ev, int actorIdx) {
        // Route to audio
        audioRouter?.PlayEvent(ev, actorIdx);

        // Route to UI
        if (ev == GameEvent.TransferBell && actorIdx == localActor.actorIdx) {
            uiRouter?.ShowPossessionUI(true);
        }

        // Handle game over
        if (game.phase == Phase.Results) {
            uiRouter?.ShowResults(game.ending, game.AliveCount());
        }
    }

    [PunRPC]
    void KillRPC(int killerIdx, int victimIdx) {
        if (!PhotonNetwork.IsMasterClient) return;

        // Validate kill (Master Client authority)
        var killer = FindActorByIdx(killerIdx);
        var victim = FindActorByIdx(victimIdx);
        if (killer == null || victim == null) return;

        float distance = Vector3.Distance(killer.EyePosition, victim.EyePosition);
        
        // Check vision tier (must be Full to kill)
        bool isMoving = killer.controller.velocity.magnitude > 0.1f;
        VisTier tier = Sight.Evaluate(killer.EyePosition, killer.transform.forward, victim.EyePosition, isMoving);
        if (tier != VisTier.Full) return;

        // Check line-of-sight
        if (Physics.Linecast(killer.EyePosition, victim.EyePosition, out RaycastHit hit)) {
            if (hit.collider.gameObject.layer == LayerMask.NameToLayer("Wall")) return;
        }

        // Valid kill
        if (game.TryKill(killerIdx, victimIdx, distance)) {
            photonView.RPC("BroadcastKill", RpcTarget.All, victimIdx);
        }
    }

    [PunRPC]
    void BroadcastKill(int victimIdx) {
        var victim = FindActorByIdx(victimIdx);
        if (victim != null) {
            PhotonNetwork.Destroy(victim.gameObject);
        }
    }

    Actor FindActorByIdx(int idx) {
        foreach (var a in ActorRegistry.Instance.AllActors) {
            if (a.actorIdx == idx) return a;
        }
        return null;
    }
}

}
```

**Add to scene:** Create empty GameObject → Name: `ArenaDirector` → Add Component: ArenaDirector + PhotonView.

### 7.4 Game/TopDownController.cs

**What:** WASD + mouse input, moves CharacterController. Only runs if `photonView.IsMine`.

```csharp
using UnityEngine;

namespace Possessed {

public class TopDownController : MonoBehaviour {
    [SerializeField] float moveSpeed = 5f;
    [SerializeField] float swingCooldown = 3f;

    Actor actor;
    Camera cam;
    float lastSwingTime = -999f;

    void Start() {
        actor = GetComponent<Actor>();
        cam = actor.localOnlyRoot.GetComponentInChildren<Camera>();
    }

    void Update() {
        if (!actor.photonView.IsMine) return;

        // Movement
        float h = Input.GetAxis("Horizontal");
        float v = Input.GetAxis("Vertical");
        Vector3 move = new Vector3(h, 0, v).normalized * moveSpeed * Time.deltaTime;
        actor.controller.SimpleMove(move);

        // Rotation (face mouse cursor)
        Ray ray = cam.ScreenPointToRay(Input.mousePosition);
        if (Physics.Raycast(ray, out RaycastHit hit, 100f, LayerMask.GetMask("Ground"))) {
            Vector3 lookDir = (hit.point - transform.position).normalized;
            lookDir.y = 0;
            if (lookDir.sqrMagnitude > 0.01f) {
                transform.rotation = Quaternion.LookRotation(lookDir);
            }
        }

        // Attack (left-click)
        if (Input.GetMouseButtonDown(0) && Time.time >= lastSwingTime + swingCooldown) {
            TrySwing();
            lastSwingTime = Time.time;
        }
    }

    void TrySwing() {
        // Raycast from camera to find target
        Ray ray = cam.ScreenPointToRay(Input.mousePosition);
        if (Physics.Raycast(ray, out RaycastHit hit, 100f)) {
            Actor target = hit.collider.GetComponent<Actor>();
            if (target != null && target != actor && target.currentTier == VisTier.Full) {
                // Send kill request to Master Client
                FindObjectOfType<ArenaDirector>().photonView.RPC("KillRPC", RpcTarget.MasterClient, actor.actorIdx, target.actorIdx);
            }
        }
    }
}

}
```

**Attach to Actor prefab:** Add Component: TopDownController, set moveSpeed = 5.

---

## 8. Vision System

### 8.1 Game/VisionSystem.cs

**What:** Evaluates vision tiers for all other actors every frame. Applies tier to renderers (hide/silhouette/show). Only runs on local player.

```csharp
using UnityEngine;

namespace Possessed {

public class VisionSystem : MonoBehaviour {
    Actor localActor;
    Material silhouetteMat;

    [Header("Cone Animation")]
    [SerializeField] float coneOpenSpeed = 4f;
    [SerializeField] float coneCloseSpeed = 12f;
    float currentConeAngle = 80f;  // spotlight angle (40° half-angle = 80° spot angle)
    float currentConeRange = 14f;

    void Start() {
        localActor = GetComponent<Actor>();
        
        // Create silhouette material
        silhouetteMat = new Material(Shader.Find("Unlit/Color"));
        silhouetteMat.color = Color.black;
    }

    void Update() {
        if (!localActor.photonView.IsMine) return;

        bool isMoving = localActor.controller.velocity.magnitude > 0.1f;

        // Animate cone
        float targetAngle = isMoving ? 80f : 124f;  // 40° → 62° half-angle
        float targetRange = isMoving ? 14f : 16f;
        float speed = isMoving ? coneCloseSpeed : coneOpenSpeed;

        currentConeAngle = Mathf.Lerp(currentConeAngle, targetAngle, Time.deltaTime * speed);
        currentConeRange = Mathf.Lerp(currentConeRange, targetRange, Time.deltaTime * speed);

        localActor.focusLight.spotAngle = currentConeAngle;
        localActor.focusLight.range = currentConeRange;

        // Evaluate all other actors
        foreach (var other in ActorRegistry.Instance.AllActors) {
            if (other == localActor) continue;

            VisTier tier = Sight.Evaluate(localActor.EyePosition, localActor.transform.forward, other.EyePosition, isMoving);

            // Check wall occlusion
            if (tier != VisTier.Hidden) {
                if (Physics.Linecast(localActor.EyePosition, other.EyePosition, out RaycastHit hit)) {
                    if (hit.collider.gameObject.layer == LayerMask.NameToLayer("Wall")) {
                        tier = VisTier.Hidden;
                    }
                }
            }

            // Apply tier
            other.currentTier = tier;
            ApplyTier(other, tier);
        }
    }

    void ApplyTier(Actor actor, VisTier tier) {
        switch (tier) {
            case VisTier.Hidden:
                actor.bodyRenderer.enabled = false;
                actor.nameTag.enabled = false;
                break;
            case VisTier.Silhouette:
                actor.bodyRenderer.enabled = true;
                actor.bodyRenderer.material = silhouetteMat;
                actor.nameTag.enabled = false;
                break;
            case VisTier.Full:
                actor.bodyRenderer.enabled = true;
                actor.bodyRenderer.material = actor.bodyRenderer.sharedMaterial;  // restore original
                actor.nameTag.enabled = true;
                break;
        }
    }
}

}
```

**Attach to Actor prefab:** Add Component: VisionSystem.

---

## 9. Audio System

### 9.1 Audio/AudioRouter.cs

**What:** Routes GhostGame events to 3D audio. Plays scream/bell/crescendo/shatter from actor positions. Handles wall muffling.

```csharp
using UnityEngine;

namespace Possessed {

public class AudioRouter : MonoBehaviour {
    [Header("Audio Clips")]
    [SerializeField] AudioClip screamClip;
    [SerializeField] AudioClip bellClip;
    [SerializeField] AudioClip crescendoClip;
    [SerializeField] AudioClip shatterClip;

    Actor localActor;

    void Start() {
        localActor = FindObjectOfType<Actor>();  // local player's actor
    }

    public void PlayEvent(GameEvent ev, int actorIdx) {
        Actor actor = FindActorByIdx(actorIdx);
        if (actor == null) return;

        AudioClip clip = null;
        switch (ev) {
            case GameEvent.Scream: clip = screamClip; break;
            case GameEvent.TransferBell: clip = bellClip; break;
            case GameEvent.TimerWarning: clip = crescendoClip; break;
            case GameEvent.HostExecuted: clip = screamClip; break;
            case GameEvent.GhostShattered: clip = shatterClip; break;
        }

        if (clip != null) {
            actor.audioSource.PlayOneShot(clip);
        }
    }

    void Update() {
        if (localActor == null) return;

        // Wall muffling: check LOS to each actor, set filter cutoff
        foreach (var other in ActorRegistry.Instance.AllActors) {
            if (other == localActor) continue;

            bool wallBlocks = Physics.Linecast(localActor.EyePosition, other.EyePosition, out RaycastHit hit)
                && hit.collider.gameObject.layer == LayerMask.NameToLayer("Wall");

            other.lowPassFilter.cutoffFrequency = wallBlocks ? 800f : 22000f;
        }
    }

    Actor FindActorByIdx(int idx) {
        foreach (var a in ActorRegistry.Instance.AllActors) {
            if (a.actorIdx == idx) return a;
        }
        return null;
    }
}

}
```

**Add to scene:** Create empty GameObject → Name: `AudioRouter` → Add Component: AudioRouter, assign audio clips.

---

## 10. UI System

### 10.1 Canvas Setup

Hierarchy → UI → Canvas
- Render Mode: Screen Space - Overlay

**HUD (alive count, possession warning):**
- Add TextMeshProUGUI → Top-left → "Alive: X"
- Add Panel → Full screen, red tint, "YOU ARE POSSESSED" text, Timer text → Disabled by default

**Defeat Screen:**
- Add Panel → Full screen, dark red, "YOU DIED" text → Disabled by default

**Results Screen:**
- Add Panel → Full screen, "SURVIVORS WIN" / "GHOST WINS" text → Disabled by default

### 10.2 UI/UIRouter.cs

**What:** Shows/hides UI elements based on game state.

```csharp
using UnityEngine;
using TMPro;

namespace Possessed {

public class UIRouter : MonoBehaviour {
    [Header("HUD")]
    [SerializeField] TextMeshProUGUI aliveCountText;
    [SerializeField] GameObject possessionPanel;
    [SerializeField] TextMeshProUGUI possessionTimerText;

    [Header("Screens")]
    [SerializeField] GameObject defeatScreen;
    [SerializeField] GameObject resultsScreen;
    [SerializeField] TextMeshProUGUI resultsText;

    GhostGame game;

    public void ShowPossessionUI(bool show) {
        possessionPanel.SetActive(show);
    }

    void Update() {
        if (game != null) {
            aliveCountText.text = $"Alive: {game.AliveCount()}";
            if (possessionPanel.activeSelf) {
                possessionTimerText.text = $"{Mathf.CeilToInt(game.possessTimer)}s";
            }
        }
    }

    public void ShowResults(Ending ending, int aliveCount) {
        resultsScreen.SetActive(true);
        switch (ending) {
            case Ending.SurvivorsWin:
                resultsText.text = "SURVIVORS WIN";
                break;
            case Ending.GhostWins:
                resultsText.text = "GHOST WINS";
                break;
            case Ending.NoWinner:
                resultsText.text = "NO WINNER";
                break;
        }
    }
}

}
```

**Add to scene:** Create empty GameObject → Name: `UIRouter` → Add Component: UIRouter, assign UI elements.

---

## 11. Photon Multiplayer

### 11.1 Summary

- **ArenaDirector** handles connection flow: Connect → JoinLobby → JoinOrCreateRoom → Spawn actors
- **PhotonView + PhotonTransformView** on Actor prefab auto-syncs position/rotation
- **RPCs** for kill validation: Client sends KillRPC to Master Client → Master validates → BroadcastKill to all
- **Master Client authority:** Only Master Client runs GhostGame.Tick() and validates kills
- **No ownership transfer** in this version (no player possession mechanic — Ghost is just a virtual tag)

### 11.2 Key Points

- Actor.prefab MUST be in Assets/Resources/
- `PhotonNetwork.Instantiate("Actor", ...)` spawns on all clients
- `photonView.IsMine` determines local vs remote actors
- Master Client = first player in room, migrates automatically if they leave

---

## 12. Bot AI

### 12.1 Game/BotBrain.cs

**What:** NavMeshAgent-based pathfinding. Picks random NavMesh points, moves there. Only runs on non-local actors.

```csharp
using UnityEngine;
using UnityEngine.AI;

namespace Possessed {

public class BotBrain : MonoBehaviour {
    Actor actor;
    bool isBot;

    void Start() {
        actor = GetComponent<Actor>();
        isBot = !actor.photonView.IsMine;

        if (isBot) {
            actor.navAgent.enabled = true;
            actor.controller.enabled = false;
            PickNewTarget();
        } else {
            actor.navAgent.enabled = false;
            actor.controller.enabled = true;
        }
    }

    void Update() {
        if (!isBot) return;

        if (!actor.navAgent.pathPending && actor.navAgent.remainingDistance <= actor.navAgent.stoppingDistance) {
            PickNewTarget();
        }
    }

    void PickNewTarget() {
        Vector3 randomPos = new Vector3(Random.Range(-12f, 12f), 0, Random.Range(-12f, 12f));
        if (NavMesh.SamplePosition(randomPos, out NavMeshHit hit, 5f, NavMesh.AllAreas)) {
            actor.navAgent.SetDestination(hit.position);
        }
    }
}

}
```

**Attach to Actor prefab:** Add Component: BotBrain.

---

## 13. Build & Submit

### 13.1 WebGL Build

File → Build Settings → WebGL → Build
- Compression: Brotli (smaller files)
- Template: Minimal

Test locally: Open index.html in browser.

### 13.2 itch.io Upload

- Zip build folder
- itch.io → Create Project → HTML → Upload zip
- Check "This file will be played in the browser"
- Set index.html as primary

---

**All sections complete. Camera is under Actor → LocalOnly. No separate CameraRig. No player possession of bots. Ghost is a virtual tag that transfers randomly on kills.**
