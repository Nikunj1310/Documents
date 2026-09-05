# 11-Hour Jam Timeline — POSSESSED

**Read UNITY-PHOTON-FUNDAMENTALS.md first** if you're new to Unity (explains GameObjects, Components, MonoBehaviour, Photon architecture).

**Read GUIDE-FULL.md for complete implementation** — this timeline tells you what to build and when.

**Key design points:**
- **Ghost is a virtual tag** (not a physical entity) that randomly transfers between players on kills
- **Camera is under Actor → LocalOnly** (no separate CameraRig GameObject)
- **No player possession of bots** — Ghost just moves between players randomly
- **Win conditions:** Kill possessed player = survivors win. Kill leaves 2 alive (one Ghost) = Ghost wins.

---

## Block 1: Project Setup (0:00–0:30)

**Goal:** Create Unity project, import Photon PUN 2, set up folders.

**What you're building:** Empty shell with dependencies installed.

### Steps:

1. **Create Unity project** (3D URP template)
2. **Import Photon PUN 2**
   - Asset Store → "Photon PUN 2 FREE" → Import
   - Register at photonengine.com/pun → Create app (type: Pun) → Copy App ID
   - Window → Photon Unity Networking → PUN Wizard → Setup Project → Paste App ID
   - **What is Photon PUN 2:** Multiplayer framework that syncs GameObjects across clients. See **UNITY-PHOTON-FUNDAMENTALS.md §5**.
   - **Why PUN 2:** WebGL support (browser builds), room codes built-in, works over internet via Photon Cloud.

3. **Install AI Navigation package**
   - Window → Package Manager → Unity Registry → "AI Navigation" → Install
   - **What is AI Navigation:** NavMesh baking system for bot pathfinding.

4. **Create folders** — see **GUIDE-FULL.md §3.5**:
   ```
   Assets/
   ├─ Resources/        (CRITICAL: Photon prefabs MUST be here)
   ├─ Scripts/Core/     (plain C#: GhostGame, Sight)
   ├─ Scripts/Game/     (MonoBehaviours: Actor, ArenaDirector, etc.)
   ├─ Scripts/Multiplayer/
   ├─ Scripts/Audio/
   ├─ Scripts/UI/
   ├─ Scenes/
   ├─ Audio/
   └─ Materials/
   ```

5. **Project Settings:**
   - Window → Rendering → Lighting → Environment → Ambient Intensity: **0** (pitch-black shadows)
   - Edit → Project Settings → Tags and Layers → Add layers: `Wall`, `Ground`

6. **Test WebGL build now** (5 min):
   - File → Build Settings → WebGL → Switch Platform → Build
   - **Why now:** Broken build pipeline at hour 10 = failed jam submission.

**Verification:** Resources/ folder exists, Photon connects (test with `PhotonNetwork.ConnectUsingSettings()`).

**Reference:** **GUIDE-FULL.md §3** (Project Setup)

---

## Block 2: Scene Setup (0:30–1:00)

**Goal:** Create arena with floor, walls, NavMesh, lighting.

**What you're building:** Physical game world with collision and pathfinding.

### Steps:

1. **Create Arena scene** — Save as `Scenes/Arena.unity`

2. **Floor** — see **GUIDE-FULL.md §4.1**:
   - GameObject → 3D Object → **Cube** (NOT Plane)
   - Scale: (30, 0.1, 30), Position: (0, 0, 0)
   - Layer: Ground
   - **Why Cube not Plane:** CharacterController.isGrounded requires BoxCollider (Plane uses MeshCollider which doesn't work). See **UNITY-PHOTON-FUNDAMENTALS.md §7.1**.
   - **What is BoxCollider:** Component defining box-shaped collision boundary. CharacterController uses it to detect ground contact.

3. **Walls** — see **GUIDE-FULL.md §4.2**:
   - GameObject → 3D Object → Cube (×4-8)
   - Scale: (10, 2.5, 0.2) or similar (2.5m tall)
   - Layer: Wall
   - Position to form 4-5 rooms with 2-3 unit doorway gaps
   - **Why 2.5m tall:** Above eye level (1.5m) for vision linecasts, not oppressively tall.

4. **Bake NavMesh** — see **GUIDE-FULL.md §4.3**:
   
   **What is NavMesh:** Navigation mesh — simplified walkable surface for AI pathfinding. NavMeshAgent uses this to find paths around obstacles.
   
   **Path A (Unity 2022.2+ with AI Navigation package):**
   - Create empty GameObject → Name: `NavMesh`
   - Add Component → NavMesh Surface
   - Settings: Collect Objects: All, Use Geometry: Render Meshes, Agent Type: Humanoid
   - Click **Bake** button
   - **Verify:** Blue overlay on floor in Scene view
   
   **Path B (Older Unity, no package):**
   - Select Floor + Walls → Inspector → Static → Navigation Static
   - Window → AI → Navigation → Bake tab → Bake
   - **Verify:** Blue overlay on floor
   
   **Common issue:** Blue overlay insets from walls → Normal (agent radius 0.5m).

5. **Lighting:**
   - **Directional Light** (already exists): Rotation (50, -30, 0), Intensity ~1
   - **Environment:** Window → Rendering → Lighting → Ambient Intensity: 0 (horror aesthetic: pitch-black shadows)

**Verification:** Floor + walls exist, NavMesh blue overlay visible, scene playable (can move around).

**Reference:** **GUIDE-FULL.md §4** (Scene Setup)

---

## Block 3: Actor Prefab (1:00–1:30)

**Goal:** Create Actor prefab with all components (visual, movement, network, local-only camera/lights).

**What you're building:** The player/bot character template. Every actor is an instance of this prefab.

### Hierarchy:

```
Actor (root)
├─ CapsuleBody (visual mesh)
├─ EyeAnchor (empty, y=1.5, for vision linecasts)
├─ NameTag (TextMeshPro world-space text, y=2.5)
└─ LocalOnly (enabled ONLY for photonView.IsMine)
   ├─ Camera (0, 18, -8 relative to Actor, rotation 70, 0, 0)
   ├─ FocusLight (Spot Light, shadows ON)
   └─ PeriphLight (Point Light, range 4.5, shadows ON)
```

**CRITICAL CAMERA SETUP:** Camera is a **child of Actor → LocalOnly**, NOT a separate GameObject in scene. No CameraRig. No CameraFollow script. Camera follows Actor automatically via parent-child hierarchy.

### Components on Actor Root:

| Component | Settings | Why |
|---|---|---|
| **CharacterController** | Radius 0.5, Height 2.0, Center (0,1,0) | Player movement with collision, no physics. See **UNITY-PHOTON-FUNDAMENTALS.md §1.2**. |
| **NavMeshAgent** | Speed 3.5, Radius 0.5, Height 2.0, **Disabled by default** | Bot pathfinding. Enable for bots, disable for players. |
| **AudioSource** | Spatial Blend 1.0, Play On Awake OFF | 3D positional audio (screams/bells come from actor position). |
| **AudioLowPassFilter** | Cutoff 22000 Hz | Muffles to 800 Hz when walls block line-of-sight. |
| **PhotonView** | Observables: PhotonTransformView | Network sync, ownership tracking. See **UNITY-PHOTON-FUNDAMENTALS.md §5.3**. |
| **PhotonTransformView** | Sync Position ON, Rotation ON, Interpolate ON | Auto-syncs position/rotation across clients. See **UNITY-PHOTON-FUNDAMENTALS.md §5.4**. |

**CRITICAL NavMeshAgent + CharacterController conflict:**
- **Bots:** NavMeshAgent ON, CharacterController OFF
- **Players:** CharacterController ON, NavMeshAgent OFF
- **Both ON = jittering** (they fight for control). See **UNITY-PHOTON-FUNDAMENTALS.md §7.3**.

### Build Steps:

1. **Create root:** Hierarchy → Create Empty → Name: `Actor`, Position (0, 1, 0)

2. **CapsuleBody:** Right-click Actor → 3D Object → Capsule, Position (0, 0, 0), Remove CapsuleCollider (CharacterController handles collision)

3. **EyeAnchor:** Right-click Actor → Create Empty, Name: `EyeAnchor`, Position (0, 1.5, 0)
   - **Why:** Vision linecasts use eye height, not ground level (ground linecasts hit floor).

4. **NameTag:** Right-click Actor → UI → Text - TextMeshPro (world-space), Position (0, 2.5, 0), Font Size ~2, Center alignment
   - Add Billboard script (see **GUIDE-FULL.md §5.5**)

5. **Add components to Actor root:**
   - CharacterController (Radius 0.5, Height 2.0, Center 0,1,0)
   - NavMeshAgent (Speed 3.5, **Disabled**)
   - AudioSource (Spatial Blend 1.0)
   - AudioLowPassFilter
   - PhotonView
   - PhotonTransformView (add to PhotonView Observables list)

6. **LocalOnly branch:**
   - Right-click Actor → Create Empty → Name: `LocalOnly`, Position (0, 0, 0)
   - **What is LocalOnly:** Container for rendering only the owning client should see (camera, lights). At runtime: `localOnlyRoot.SetActive(photonView.IsMine)`.
   - **Why:** Performance (1 actor with 2 shadow lights = fine, 8 actors with 16 = slideshow). See **UNITY-PHOTON-FUNDAMENTALS.md §6.2**.

7. **Camera under LocalOnly:**
   - Right-click LocalOnly → Camera
   - Position: (0, 18, -8) relative to Actor
   - Rotation: (70, 0, 0) — pitch down 70°
   - Tag: **Untagged** (NOT MainCamera)
   - Clear Flags: Skybox, FOV: ~60
   - **Why child of Actor:** Moves automatically with Actor (parent-child hierarchy). Zero scripts needed.

8. **FocusLight under LocalOnly:**
   - Right-click LocalOnly → Light → Spot Light
   - Position: (0, 1.5, 0), Type: Spot, Range: 14, Spot Angle: 80, Intensity: 3-5
   - Shadows: Soft Shadows (load-bearing: shadows show wall occlusion)
   - **What is Spot Light:** Cone-shaped light (like flashlight). Matches vision cone. See **UNITY-PHOTON-FUNDAMENTALS.md §1.2**.

9. **PeriphLight under LocalOnly:**
   - Right-click LocalOnly → Light → Point Light
   - Position: (0, 1.5, 0), Range: 4.5, Intensity: 1-2
   - Shadows: Soft Shadows
   - **What is Point Light:** Omnidirectional light (like light bulb). Shows 4.5m peripheral vision sphere.

10. **Save as prefab:**
    - Drag Actor from Hierarchy → **Assets/Resources/** folder
    - **MUST be in Resources/** — `PhotonNetwork.Instantiate("Actor", ...)` only works here. See **UNITY-PHOTON-FUNDAMENTALS.md §7.4**.
    - Delete Actor from Hierarchy (spawned at runtime)

**Verification:**
- Actor.prefab in Resources/
- Hierarchy: Actor → CapsuleBody, EyeAnchor, NameTag, LocalOnly (Camera, FocusLight, PeriphLight)
- PhotonView has PhotonTransformView in Observables list

**Reference:** **GUIDE-FULL.md §5** (Actor Prefab)

---

## Block 4: Core Scripts (Plain C#) (1:30–2:30)

**Goal:** Implement GhostGame (possession state machine) and Sight (vision tier math). Plain C#, zero Unity dependencies.

**What you're building:** The game rules engine. Handles Ghost transfers, kill validation, win conditions.

### Scripts to Write:

1. **Scripts/Core/GhostGame.cs** — see **GUIDE-FULL.md §6.1**:
   - **What:** State machine for Ghost possession, kill validation, timer, win conditions
   - **Why plain C#:** Testable without Unity, same logic works for mobile version later
   - Key types: `enum GameEvent`, `enum Phase`, `enum Ending`
   - Key state: `int hostIdx` (who is possessed), `bool[] alive`, `float possessTimer`
   - Key methods:
     - `Begin(playerCount)` — Start match, pick first Ghost host randomly
     - `Tick(dt)` — Update timer, fire warning/expiry events
     - `TryKill(killerIdx, victimIdx, distance)` — Validate + apply kill, return true if success
     - `TransferGhost(excludeIdx)` — Random transfer, reset timer, fire TransferBell event
   - **Event system:** `event Action<GameEvent, int> OnEvent` — fires Scream, TransferBell, TimerWarning, etc.

2. **Scripts/Core/Sight.cs** — see **GUIDE-FULL.md §6.2**:
   - **What:** Pure math for vision tier evaluation
   - **Key method:** `static VisTier Evaluate(Vector3 observerPos, Vector3 observerFwd, Vector3 targetPos, bool observerMoving)`
   - **Logic:**
     - Focus cone: 40° half-angle / 14m range (moving) → 62° / 16m (still)
     - Peripheral sphere: 4.5m radius (returns Silhouette)
     - Hidden: Outside both
   - **Why pure Vector3 math:** No Unity dependencies, deterministic, same on all clients

3. **Scripts/Core/IVoiceLayer.cs** — see **GUIDE-FULL.md §6.3**:
   - **What:** Interface for voice chat (stub for now)
   - `void MuteLocal(bool muted)` — GhostGame calls this when you die
   - Stub implementation: `VoiceStub` class (no-op)

**Key Design Point from design.txt:**
- **Random transfer on kill** — Kill non-possessed player → Ghost moves randomly to another living player (excludes killer by default)
- **Ghost destruction** — Kill possessed player → Ghost destroyed, survivors win
- **Win condition: 2 alive (one Ghost)** — Ghost wins, killer gets prize pool
- **Possession timer** — 90s default, warning at 15s, expires → you die, Ghost transfers randomly

**Verification:**
- All three files compile (no errors)
- GhostGame has `Begin()`, `Tick()`, `TryKill()`, `TransferGhost()` methods
- Sight has `Evaluate()` returning VisTier

**Reference:** **GUIDE-FULL.md §6** (Core Scripts)

---

## Block 5: Actor Script + Registry (2:30–3:30)

**Goal:** Implement Actor.cs (per-actor state) and ActorRegistry.cs (global actor list).

**What you're building:** MonoBehaviours that bridge plain C# core with Unity scene.

### Scripts to Write:

1. **Scripts/Game/Actor.cs** — see **GUIDE-FULL.md §7.1**:
   - **What:** Per-actor state and cached component references
   - **Why MonoBehaviour:** Attached to Actor prefab, needs Awake/Start lifecycle. See **UNITY-PHOTON-FUNDAMENTALS.md §2.1**.
   - **Fields:**
     ```csharp
     public int actorIdx = -1;  // index in GhostGame.alive[]
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
     public VisTier currentTier = VisTier.Hidden;
     ```
   - **Why cache components:** `GetComponent<T>()` is slow if called every frame. Cache in Awake(). See **UNITY-PHOTON-FUNDAMENTALS.md §2.3**.
   - **Lifecycle:**
     - `Awake()` — Cache all components, register with ActorRegistry
     - `Start()` — Enable LocalOnly if `photonView.IsMine`, disable scene MainCamera
     - `OnDestroy()` — Unregister from ActorRegistry
   - **Property:** `Vector3 EyePosition` — returns `eyeAnchor.position` (eye-level for vision checks)

2. **Scripts/Game/ActorRegistry.cs** — see **GUIDE-FULL.md §7.2**:
   - **What:** Singleton tracking all Actor instances
   - **Why:** VisionSystem, ArenaDirector, AudioRouter need to loop through all actors
   - **Pattern:**
     ```csharp
     public static ActorRegistry Instance { get; private set; }
     List<Actor> actors = new List<Actor>();
     public IReadOnlyList<Actor> AllActors => actors;
     ```
   - **Methods:** `Register(Actor a)`, `Unregister(Actor a)`
   - **What is Singleton:** Class with only one instance, accessible globally via `Instance`. See **UNITY-PHOTON-FUNDAMENTALS.md §2.2**.

3. **Attach Actor.cs to Actor prefab:**
   - Open Actor.prefab → Select root → Add Component: Actor script

4. **Add ActorRegistry to scene:**
   - Hierarchy → Create Empty → Name: `ActorRegistry` → Add Component: ActorRegistry script

**Verification:**
- Actor.cs compiles, attached to Actor prefab
- ActorRegistry.cs compiles, attached to GameObject in scene
- Play mode: ActorRegistry.Instance exists (no null reference errors)

**Reference:** **GUIDE-FULL.md §7.1-7.2**

---

## Block 6: ArenaDirector (Game Lifecycle) (3:30–4:30)

**Goal:** Implement ArenaDirector (owns GhostGame, connects to Photon, spawns actors, routes events).

**What you're building:** The orchestrator. Connects everything together.

### Script to Write:

**Scripts/Game/ArenaDirector.cs** — see **GUIDE-FULL.md §7.3**:

**What it does:**
1. Connects to Photon (ConnectUsingSettings → JoinLobby → JoinOrCreateRoom)
2. Spawns local Actor when joined room
3. Master Client initializes GhostGame, calls `Begin(playerCount)`
4. Every frame (Update): Master Client calls `game.Tick(Time.deltaTime)`
5. Subscribes to `game.OnEvent` → routes to AudioRouter/UIRouter
6. Handles kill RPCs (client sends claim, Master validates, broadcasts result)

**Key Photon callbacks:**
- `OnConnectedToMaster()` → `PhotonNetwork.JoinLobby()`
- `OnJoinedLobby()` → `PhotonNetwork.JoinOrCreateRoom("Arena", ...)`
- `OnJoinedRoom()` → Spawn actor, init GhostGame if Master Client
- **What is MonoBehaviourPunCallbacks:** Base class for Photon callback methods. See **UNITY-PHOTON-FUNDAMENTALS.md §5.6**.

**Kill RPC flow:**
1. Client calls `photonView.RPC("KillRPC", RpcTarget.MasterClient, killerIdx, victimIdx)`
2. Master Client receives, validates:
   - Check distance (survivor: 2m, ghost: 1m)
   - Check vision tier (must be Full)
   - Check line-of-sight (Physics.Linecast, wall blocks?)
   - Check cooldown (3s)
3. If valid: `game.TryKill(killerIdx, victimIdx, distance)`
4. Master broadcasts: `photonView.RPC("BroadcastKill", RpcTarget.All, victimIdx)`
5. All clients destroy victim Actor

**Add to scene:**
- Hierarchy → Create Empty → Name: `ArenaDirector`
- Add Component: ArenaDirector script
- Add Component: PhotonView (required for RPCs)

**Verification:**
- ArenaDirector.cs compiles
- Play mode: Connects to Photon, joins room, spawns Actor
- Console shows "Connected to Photon Cloud", "Joined room"

**Reference:** **GUIDE-FULL.md §7.3** (ArenaDirector), **GUIDE-FULL.md §11** (Photon Multiplayer)

---

## Block 7: Player Input (4:30–5:30)

**Goal:** Implement TopDownController (WASD movement, mouse aiming, left-click attack).

**What you're building:** Player control. Only runs on `photonView.IsMine` actors.

### Script to Write:

**Scripts/Game/TopDownController.cs** — see **GUIDE-FULL.md §7.4**:

**What it does:**
- **Movement:** Read WASD input → `controller.SimpleMove(move)`
- **Rotation:** Raycast from camera to ground (mouse cursor) → `transform.rotation = Quaternion.LookRotation(lookDir)`
- **Attack:** Left-click → Raycast from camera → Find Actor target → Check `currentTier == VisTier.Full` → Send KillRPC to Master Client
- **Cooldown:** 3 seconds between swings

**Key input methods:**
- `Input.GetAxis("Horizontal")` — A/D or arrow keys, returns -1 to 1
- `Input.GetAxis("Vertical")` — W/S or arrow keys
- `Input.GetMouseButtonDown(0)` — Left-click, true for one frame
- **What is Input:** Global Unity API for keyboard/mouse. See **UNITY-PHOTON-FUNDAMENTALS.md §4.2**.

**Why `if (!photonView.IsMine) return;`:**
- Only the owner should read input and move
- Other clients receive synced position from PhotonTransformView
- See **UNITY-PHOTON-FUNDAMENTALS.md §5.3**.

**Attach to Actor prefab:**
- Open Actor.prefab → Select root → Add Component: TopDownController, set moveSpeed = 5

**Verification:**
- TopDownController.cs compiles, attached to Actor prefab
- Play mode: WASD moves your Actor, mouse rotates, left-click sends kill RPC

**Reference:** **GUIDE-FULL.md §7.4**

---

## Block 8: Vision System (5:30–7:00)

**Goal:** Implement VisionSystem (evaluates vision tiers, applies to renderers, animates cone).

**What you're building:** The gameplay truth — who can see whom, who can kill whom.

### Script to Write:

**Scripts/Game/VisionSystem.cs** — see **GUIDE-FULL.md §8.1**:

**What it does:**
1. Every frame: Loop through all other actors
2. Call `Sight.Evaluate(localEyePos, localFwd, otherEyePos, isMoving)` → returns VisTier
3. If not Hidden: `Physics.Linecast(localEyes, otherEyes)` → if hits Wall layer → downgrade to Hidden
4. Store tier in `other.currentTier`
5. Apply tier to renderer:
   - Hidden: `bodyRenderer.enabled = false`, `nameTag.enabled = false`
   - Silhouette: `bodyRenderer.enabled = true`, `material = silhouetteMat` (unlit black), `nameTag.enabled = false`
   - Full: `bodyRenderer.enabled = true`, restore original material, `nameTag.enabled = true`
6. Animate spotlight cone: Narrow (80° / 14m) when moving, wide (124° / 16m) when still

**Key Unity APIs:**
- `Physics.Linecast(start, end, out RaycastHit hit)` — Cast ray from start to end, returns true if hits collider. See **UNITY-PHOTON-FUNDAMENTALS.md §4.4**.
- `LayerMask.NameToLayer("Wall")` — Get layer index from name
- `MeshRenderer.material` — Get/set material (creates instance, use `sharedMaterial` for prefab default)

**Create Silhouette Material:**
- Assets → Create → Material → Name: `Silhouette`
- Shader: Unlit/Color, Color: Black

**Attach to Actor prefab:**
- Open Actor.prefab → Select root → Add Component: VisionSystem

**Verification:**
- VisionSystem.cs compiles, attached to Actor prefab
- Play mode: Other actors hidden/silhouette/full based on your vision
- Spotlight angle animates when you stop moving

**Reference:** **GUIDE-FULL.md §8** (Vision System)

---

## Block 9: Bot AI (7:00–8:00)

**Goal:** Implement BotBrain (NavMeshAgent pathfinding, random target selection).

**What you're building:** Autonomous AI that makes non-local actors move around.

### Script to Write:

**Scripts/Game/BotBrain.cs** — see **GUIDE-FULL.md §12.1**:

**What it does:**
- `Start()`: Check `!photonView.IsMine` → if true, enable NavMeshAgent, disable CharacterController, pick first target
- `Update()`: Check if reached destination (`remainingDistance <= stoppingDistance`) → pick new target
- `PickNewTarget()`: Random position, `NavMesh.SamplePosition()` to find nearest valid NavMesh point, `SetDestination()`

**Why NavMeshAgent:**
- Automatic pathfinding around obstacles
- You set destination, it handles movement
- See **UNITY-PHOTON-FUNDAMENTALS.md §1.2**.

**CRITICAL NavMeshAgent + CharacterController conflict:**
- Bots: NavMeshAgent ON, CharacterController OFF
- Players: CharacterController ON, NavMeshAgent OFF
- See **UNITY-PHOTON-FUNDAMENTALS.md §7.3**.

**Attach to Actor prefab:**
- Open Actor.prefab → Select root → Add Component: BotBrain

**Verification:**
- BotBrain.cs compiles, attached to Actor prefab
- Play mode with 2+ actors: Non-owned actors wander (NavMeshAgent), owned actor awaits input (CharacterController)
- No jittering (proves conflict avoided)

**Reference:** **GUIDE-FULL.md §12** (Bot AI)

---

## Block 10: Audio System (8:00–8:30)

**Goal:** Implement AudioRouter (plays event sounds from actor positions, handles wall muffling).

**What you're building:** Spatial audio feedback — screams, bells, crescendos from 3D positions.

### Script to Write:

**Scripts/Audio/AudioRouter.cs** — see **GUIDE-FULL.md §9.1**:

**What it does:**
1. Subscribe to `game.OnEvent` (via ArenaDirector)
2. Map events to audio clips:
   - Scream → scream.wav
   - TransferBell → bell.wav
   - TimerWarning → crescendo.wav
   - HostExecuted → scream.wav
   - GhostShattered → shatter.wav
3. Find actor by idx, play `actor.audioSource.PlayOneShot(clip)`
4. Every frame: Check LOS to each actor, set `lowPassFilter.cutoffFrequency` (800 Hz if wall blocks, 22000 Hz if clear)

**Audio clips:**
- Download free horror sounds (freesound.org, OpenGameArt.org)
- Import to Assets/Audio/
- Assign in AudioRouter Inspector

**What is AudioLowPassFilter:**
- Cuts high frequencies above cutoff
- 22000 Hz = no filtering (human hearing ~20kHz max)
- 800 Hz = heavy muffling (sounds like through a wall)
- See **UNITY-PHOTON-FUNDAMENTALS.md §1.2**.

**Add to scene:**
- Hierarchy → Create Empty → Name: `AudioRouter` → Add Component: AudioRouter, assign clips

**Verification:**
- AudioRouter.cs compiles
- Play mode: Kills play scream from victim position, Ghost transfers play bell from new host position
- Audio muffles when walls block LOS

**Reference:** **GUIDE-FULL.md §9** (Audio System)

---

## Block 11: UI System (8:30–9:30)

**Goal:** Implement UIRouter (HUD, possession panel, defeat screen, results screen).

**What you're building:** Visual feedback — alive count, possession timer, game over screens.

### Canvas Setup:

**Hierarchy → UI → Canvas:**
- Render Mode: Screen Space - Overlay
- **What is Screen Space - Overlay:** UI renders on top of everything, always visible. See **UNITY-PHOTON-FUNDAMENTALS.md** (Canvas section).

**HUD elements:**
1. **Alive Count** (TextMeshProUGUI, top-left):
   - Text: "Alive: X"
   - Updates every frame from `game.AliveCount()`

2. **Possession Panel** (Panel, full-screen, red tint, disabled by default):
   - Text: "YOU ARE POSSESSED"
   - Timer text: "Xs remaining"
   - Enabled when `OnEvent(TransferBell, localActorIdx)` fires

**Defeat Screen (Panel, disabled by default):**
- Full-screen, dark red tint
- Text: "YOU DIED"
- Enabled when local actor dies

**Results Screen (Panel, disabled by default):**
- Full-screen
- Text: "SURVIVORS WIN" / "GHOST WINS" / "NO WINNER"
- Enabled when `game.phase == Phase.Results`

### Script to Write:

**Scripts/UI/UIRouter.cs** — see **GUIDE-FULL.md §10.2**:

**What it does:**
- `Update()`: Update alive count text, possession timer text
- `ShowPossessionUI(bool show)`: Enable/disable possession panel
- `ShowResults(Ending ending, int aliveCount)`: Show results screen with appropriate text

**Serialized fields:**
```csharp
[SerializeField] TextMeshProUGUI aliveCountText;
[SerializeField] GameObject possessionPanel;
[SerializeField] TextMeshProUGUI possessionTimerText;
[SerializeField] GameObject defeatScreen;
[SerializeField] GameObject resultsScreen;
[SerializeField] TextMeshProUGUI resultsText;
```

**Add to scene:**
- Hierarchy → Create Empty → Name: `UIRouter` → Add Component: UIRouter, assign UI elements from Canvas

**Verification:**
- UIRouter.cs compiles
- Play mode: Alive count updates, possession panel shows when you're possessed, results screen shows on game end

**Reference:** **GUIDE-FULL.md §10** (UI System)

---

## Block 12: Multiplayer Testing (9:30–10:00)

**Goal:** Test with 2+ clients (Editor + build).

**What you're testing:** Network sync, kill validation, Ghost transfers, win conditions.

### Steps:

1. **Build WebGL** (File → Build Settings → WebGL → Build)
2. **Run both:** Editor play mode + open build in browser
3. **Test scenarios:**
   - Both players spawn, see each other
   - Movement syncs (other player's position updates)
   - Vision tiers work (Full = visible + can kill, Silhouette = visible but can't kill, Hidden = invisible)
   - Kill non-possessed player → Ghost transfers (bell sound from new host position)
   - Kill possessed player → Ghost destroyed (shatter sound), survivors win
   - Let possession timer expire → host dies, Ghost transfers
   - Get to 2 alive (one Ghost) → Ghost wins

**Common issues:**
- **Actor doesn't spawn:** Check Actor.prefab in Resources/ folder
- **Movement doesn't sync:** Check PhotonTransformView in PhotonView Observables list
- **Can't kill anyone:** Check vision tier (must be Full), check kill radius (2m survivor, 1m ghost)
- **RPC not received:** Check PhotonView on ArenaDirector, check RPC method has `[PunRPC]` attribute

**Reference:** **GUIDE-FULL.md §11** (Photon Multiplayer)

---

## Block 13: Polish & Build (10:00–11:00)

**Goal:** Final polish, WebGL build, upload to itch.io.

### Polish Pass:

1. **Audio:** Adjust volumes, test all event sounds
2. **Visuals:** Tweak lighting, add post-processing (vignette, film grain)
3. **Balance:** Adjust possession timer (90s too long? too short?), kill radii, vision cone angles
4. **UI:** Polish text, colors, layout

### WebGL Build:

**File → Build Settings → WebGL:**
- Compression Format: Brotli (smaller files, slower build)
- Template: Minimal

**Player Settings:**
- Company Name: Your name
- Product Name: POSSESSED
- WebGL Template: Minimal

**Build → Choose folder → Wait (~5-10 min)**

### Test Build:

- Open index.html in browser
- Test solo (connects, spawns, HUD works)
- Test multiplayer (open two tabs, both join same room)

### Upload to itch.io:

1. Zip entire build folder
2. itch.io → Dashboard → Create New Project
3. Kind: HTML
4. Upload: Zip file
5. Check "This file will be played in the browser"
6. Set index.html as primary file
7. Embed options: Click to launch in fullscreen
8. Add description:
   ```
   POSSESSED — Top-down 3D social deception horror
   
   One of you carries a Ghost. Every kill relocates it randomly.
   Strike down the current host to win, or the Ghost survives until only 2 remain.
   
   Controls:
   - WASD: Move
   - Mouse: Aim
   - Left-click: Attack (2m range, 3s cooldown)
   
   Vision:
   - Bright cone: Full vision (can kill)
   - Dim ring: Peripheral (can't kill)
   - Outside: Hidden
   ```
9. Add screenshots (capture from build)
10. Save → View page → Test game loads

**Verification:** itch.io page loads, game playable in browser, multiplayer works.

**Reference:** **GUIDE-FULL.md §13** (Build & Submit)

---

## Gates & Triage

**Hard gates (must complete):**
- Blocks 1-3: Project/scene/prefab setup
- Block 4: Core scripts (GhostGame, Sight)
- Block 5-6: Actor, ActorRegistry, ArenaDirector
- Block 7: TopDownController (input)
- Block 8: VisionSystem

**Soft features (cut if behind):**
- Block 10: Audio (silent game still works)
- Block 11: UI polish (minimal HUD is fine)
- Block 12: Extensive multiplayer testing (basic test is enough)

**If 2 hours behind:**
- Skip Block 10 (audio)
- Reduce Block 11 to minimal HUD (alive count only)
- Reduce Block 12 to 15 min
- Ship barebones

**If 4 hours behind:**
- Skip Blocks 10-11
- Ship with zero audio, minimal UI (just gameplay)

---

## Final Checklist

- [ ] Actor.prefab in Resources/ folder
- [ ] Camera under Actor → LocalOnly (no separate CameraRig)
- [ ] NavMeshAgent disabled by default on Actor prefab (enable for bots only)
- [ ] PhotonTransformView in PhotonView Observables list
- [ ] All walls tagged "Wall", floor tagged "Ground"
- [ ] NavMesh baked (blue overlay visible)
- [ ] ArenaDirector has PhotonView component
- [ ] ActorRegistry in scene
- [ ] AudioRouter in scene (if using audio)
- [ ] UIRouter in scene
- [ ] WebGL build tested in browser
- [ ] Multiplayer tested (2+ clients)

---

**Complete timeline. Camera under Actor → LocalOnly. Ghost is virtual tag. No player possession of bots. Random transfer on kills. All blocks reference correct GUIDE-FULL.md sections.**
