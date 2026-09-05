# Unity & Photon PUN 2 Fundamentals

**Purpose**: Beginner-friendly explanation of Unity's architecture and how Photon PUN 2 multiplayer works. Read this before coding if you're new to Unity.

---

## 1. Unity Hierarchy Basics

### 1.1 GameObject
**What**: The fundamental building block of every Unity scene. A GameObject is just a container — it does nothing by itself.

**Think of it as**: An empty box. The box exists, but it has no behavior until you put things inside it.

**Examples**:
- A player character is a GameObject
- A light is a GameObject
- An empty organizational container is a GameObject

**Key point**: GameObjects are **composed** of Components. The GameObject itself is just the container.

### 1.2 Component
**What**: Components add functionality to GameObjects. Every GameObject has at least one component: **Transform**.

**Think of it as**: Lego pieces you attach to the GameObject box. Each piece adds a specific capability.

**Common Unity Components**:
- **Transform**: Position, rotation, scale (every GameObject has one, always)
- **MeshRenderer**: Makes the GameObject visible by rendering a 3D mesh
- **Collider**: Defines collision shape (BoxCollider, SphereCollider, CapsuleCollider, MeshCollider)
- **Rigidbody**: Adds physics simulation (gravity, forces, collisions with physics)
- **CharacterController**: Movement + collision without full physics (better for player control)
- **Camera**: Renders the scene from a viewpoint
- **Light**: Illuminates the scene (Point, Spot, Directional)
- **AudioSource**: Plays audio clips
- **NavMeshAgent**: AI pathfinding on a NavMesh surface

**How to add**: Select GameObject in Hierarchy → Inspector → Add Component button

**Key point**: Components work together. A GameObject with MeshRenderer + BoxCollider is visible and has collision. Remove the MeshRenderer, it's invisible but still has collision.

### 1.3 Transform
**What**: Every GameObject has a Transform component. It defines:
- **Position**: Where the GameObject is in 3D space (x, y, z)
- **Rotation**: How the GameObject is rotated (Euler angles or quaternion)
- **Scale**: How big the GameObject is (1, 1, 1 is original size)

**Parent-Child Hierarchy**:
- GameObjects can be **parented** to other GameObjects
- Child transforms are **relative to parent**
- Move parent → children move with it
- Rotate parent → children rotate around parent's pivot

**Example**:
```
Actor (root GameObject)
├─ CapsuleBody (child: visual mesh)
├─ LocalOnly (child: organizational empty)
   ├─ Camera (grandchild: follows Actor because parented)
   ├─ FocusLight (grandchild: follows Actor)
   └─ PeriphLight (grandchild: follows Actor)
```

If Actor moves to position (5, 0, 10), **all children move with it automatically**. Camera doesn't need a script to follow — the parent-child relationship handles it.

### 1.4 Prefab
**What**: A reusable GameObject template. Create once, instantiate many times.

**Think of it as**: A cookie cutter. The prefab is the template; instances are the cookies.

**Why use prefabs**:
- Multiplayer: Every player needs an identical Actor → Actor.prefab
- Consistency: Change prefab → all instances update
- Spawning: `Instantiate(prefab)` creates a new instance at runtime

**Blue vs White text in Hierarchy**:
- **Blue**: Prefab instance (linked to prefab asset)
- **White**: Regular GameObject (not linked to prefab)

**Overrides**: Changes to a prefab instance can be applied back to the prefab asset or reverted.

---

## 2. MonoBehaviour & Scripting

### 2.1 What is MonoBehaviour?
**MonoBehaviour** is the base class for all Unity scripts. Any C# script that attaches to a GameObject **must** inherit from `MonoBehaviour`.

```csharp
public class MyScript : MonoBehaviour
{
    // This script can now be added as a Component
}
```

**Key point**: MonoBehaviour scripts **are Components**. You attach them to GameObjects the same way you attach Colliders or Lights.

**Example**:
- `Actor.cs : MonoBehaviour` → Actor script is a component on the Actor GameObject
- `TopDownController.cs : MonoBehaviour` → Movement script is a component on Actor
- `BotBrain.cs : MonoBehaviour` → AI script is a component on Actor

**Plain C# classes** (like `GhostGame.cs`, `Sight.cs`) do **not** inherit MonoBehaviour. They can't be attached to GameObjects. They're just plain data structures or logic classes.

### 2.2 MonoBehaviour Lifecycle Methods
Unity automatically calls certain methods on MonoBehaviour scripts at specific times. You override these to add your logic.

**Common lifecycle methods** (in order):

1. **Awake()**
   - Called when the script instance is loaded (scene load or `Instantiate()`)
   - Happens **before** Start()
   - Use for: Initializing references to other components on the **same GameObject**
   - Example: `rb = GetComponent<Rigidbody>();`

2. **Start()**
   - Called before the first frame update, **after all Awake() calls**
   - Use for: Initializing logic that depends on other objects being ready
   - Example: `cam = Camera.main;` (assumes MainCamera exists)

3. **Update()**
   - Called **every frame** (frame rate varies: 30fps, 60fps, 144fps)
   - Use for: Input polling, non-physics logic, animations
   - Example: `if (Input.GetKey(KeyCode.W)) { ... }`
   - **Timing varies**: On a 60fps machine, Update() runs 60 times/second. On 30fps, 30 times/second.

4. **FixedUpdate()**
   - Called at a **fixed interval** (default: 50 times/second, regardless of frame rate)
   - Use for: Physics calculations (Rigidbody forces, CharacterController movement)
   - Example: `rb.AddForce(direction * speed);`
   - **Why fixed**: Physics must be deterministic. Variable Update() timing breaks physics.

5. **LateUpdate()**
   - Called every frame **after all Update() calls**
   - Use for: Camera following (so camera moves after player movement is finalized)

6. **OnDestroy()**
   - Called when GameObject is destroyed or scene unloads
   - Use for: Cleanup (unsubscribe from events, close connections)

**Key rule**: Physics code goes in `FixedUpdate()`. Everything else goes in `Update()` or `Start()`.

### 2.3 Common Methods

**GetComponent<T>()**:
```csharp
Rigidbody rb = GetComponent<Rigidbody>();
```
- Finds a component of type `T` on the **same GameObject**
- Returns `null` if not found
- **Expensive**: Don't call every frame. Cache in `Awake()` or `Start()`.

**GameObject.Find(string name)**:
```csharp
GameObject floor = GameObject.Find("Floor");
```
- Searches **entire scene** for GameObject with exact name
- **Very expensive**: Only use in `Start()`, never in `Update()`
- Returns `null` if not found

**Instantiate()**:
```csharp
GameObject newEnemy = Instantiate(enemyPrefab, position, rotation);
```
- Creates a new instance of a prefab
- **Multiplayer caveat**: Use `PhotonNetwork.Instantiate()` instead for networked objects

**Destroy()**:
```csharp
Destroy(gameObject); // Destroys the GameObject this script is on
Destroy(otherObject); // Destroys another GameObject
```
- Destroys a GameObject (removed from scene at end of frame)
- **Multiplayer caveat**: Use `PhotonNetwork.Destroy()` for networked objects

---

## 3. Attributes

Attributes modify how Unity treats fields in your MonoBehaviour scripts. They go in `[SquareBrackets]` above the field.

### 3.1 [SerializeField]
```csharp
[SerializeField] private float speed = 5f;
```
- **What**: Makes a private field visible in the Unity Inspector
- **Why**: You want to tweak values in the Inspector without making fields public
- **Without it**: Private fields are hidden, public fields are exposed (but public breaks encapsulation)

### 3.2 [Header("Label")]
```csharp
[Header("Movement")]
[SerializeField] private float speed = 5f;
[SerializeField] private float jumpHeight = 2f;

[Header("Combat")]
[SerializeField] private int maxHealth = 100;
```
- **What**: Adds a bold label in the Inspector to organize fields
- **Why**: Readability when you have many serialized fields

### 3.3 [RequireComponent(typeof(T))]
```csharp
[RequireComponent(typeof(Rigidbody))]
public class PlayerMovement : MonoBehaviour
{
    // Unity automatically adds a Rigidbody when you add this script
}
```
- **What**: Forces Unity to add component `T` if it's missing
- **Why**: Prevents errors from missing dependencies

### 3.4 [Tooltip("description")]
```csharp
[Tooltip("Speed in meters per second")]
[SerializeField] private float speed = 5f;
```
- **What**: Shows a tooltip when you hover over the field in Inspector
- **Why**: Helps you remember what the field does

### 3.5 Photon Attributes

**[PunRPC]**:
```csharp
[PunRPC]
void TakeDamage(int amount)
{
    health -= amount;
}
```
- **What**: Marks a method as a **Remote Procedure Call** (can be invoked across the network)
- **Why**: Allows other clients to trigger this method on your GameObject
- **Usage**: `photonView.RPC("TakeDamage", RpcTarget.All, 10);`

---

## 4. Global APIs

Unity provides static classes for accessing global systems. You don't instantiate them — just call them directly.

### 4.1 Time
```csharp
float deltaTime = Time.deltaTime; // Time since last frame (varies)
float fixedDeltaTime = Time.fixedDeltaTime; // Fixed physics timestep (0.02s default)
float timeSinceLevelLoad = Time.time; // Seconds since scene loaded
```
- **Time.deltaTime**: Use in `Update()` for frame-rate-independent movement: `transform.position += direction * speed * Time.deltaTime;`
- **Why**: Without deltaTime, movement is faster on high-fps machines

### 4.2 Input
```csharp
bool jumpPressed = Input.GetKeyDown(KeyCode.Space);
bool jumpHeld = Input.GetKey(KeyCode.Space);
bool jumpReleased = Input.GetKeyUp(KeyCode.Space);

float horizontal = Input.GetAxis("Horizontal"); // -1 to 1 (A/D or arrow keys)
float vertical = Input.GetAxis("Vertical"); // -1 to 1 (W/S or arrow keys)
```
- **GetKeyDown**: True for **one frame** when key is first pressed
- **GetKey**: True **every frame** while key is held
- **GetKeyUp**: True for **one frame** when key is released

### 4.3 Camera.main
```csharp
Camera mainCam = Camera.main;
```
- **What**: Returns the Camera with tag "MainCamera"
- **Why**: Convenient way to access the main camera
- **Performance trap**: `Camera.main` searches for the camera **every time**. Cache it in `Start()`:
```csharp
private Camera cam;
void Start() { cam = Camera.main; }
void Update() { /* use cam, not Camera.main */ }
```

### 4.4 Physics.Raycast
```csharp
if (Physics.Raycast(origin, direction, out RaycastHit hit, maxDistance))
{
    Debug.Log("Hit: " + hit.collider.gameObject.name);
}
```
- **What**: Casts an invisible ray from `origin` in `direction` up to `maxDistance`
- **Returns**: `true` if ray hits a collider, `false` otherwise
- **out RaycastHit hit**: Outputs info about what was hit (point, normal, collider, distance)
- **Use case**: Vision system (can Actor A see Actor B? Raycast from eyes to target, check if walls block it)

---

## 5. Photon PUN 2 Architecture

### 5.1 What is Photon PUN 2?
**Photon Unity Networking (PUN) 2** is a multiplayer framework for Unity. It handles:
- **Connecting players** to the same game session (Room)
- **Synchronizing data** across clients (position, rotation, health, etc.)
- **Remote Procedure Calls (RPCs)** — trigger methods on other clients
- **Ownership** — which client controls which GameObject

**Why PUN 2 over Unity Netcode (NGO)**:
- **WebGL support**: PUN 2 works in browser builds, NGO doesn't (yet)
- **Simpler API**: Less boilerplate for small projects
- **Photon Cloud**: Free tier, no server setup needed

**Architecture**:
- **Client-Server topology** (not peer-to-peer)
- **Photon Cloud**: Photon's servers relay messages between clients (you don't run a dedicated server)
- **Authoritative server**: Not enforced by default — you write validation logic (see Master Client)

### 5.2 PhotonNetwork (Global API)
**PhotonNetwork** is a static class providing global multiplayer operations.

**Connection**:
```csharp
PhotonNetwork.ConnectUsingSettings(); // Connect to Photon Cloud
PhotonNetwork.JoinLobby(); // Enter lobby
PhotonNetwork.JoinOrCreateRoom(roomName, roomOptions, typedLobby); // Join/create room
```

**Room state**:
```csharp
bool inRoom = PhotonNetwork.InRoom; // Are we in a game room?
bool isMasterClient = PhotonNetwork.IsMasterClient; // Are we the Master Client?
Player localPlayer = PhotonNetwork.LocalPlayer; // Our Player object (persistent across scenes)
```

**Master Client**:
- **What**: The first player who joined the room
- **Why**: Someone needs to make authoritative decisions (is this kill valid? who spawns as ghost?)
- **Migration**: If Master Client disconnects, Photon assigns a new Master Client automatically
- **Not a dedicated server**: Master Client is just another player with extra responsibility

**Spawning networked objects**:
```csharp
GameObject obj = PhotonNetwork.Instantiate("Actor", position, rotation);
```
- **Must use PhotonNetwork.Instantiate**, not `GameObject.Instantiate`
- **Prefab must be in Resources/ folder** (Photon limitation)
- **Ownership**: The client who instantiates owns the object (can send RPCs, sync transform)

**Destroying networked objects**:
```csharp
PhotonNetwork.Destroy(gameObject);
```
- **Must use PhotonNetwork.Destroy**, not `GameObject.Destroy`
- Only the **owner** or **Master Client** can destroy networked objects

### 5.3 PhotonView Component
**PhotonView** is the core component for networked GameObjects. Every synchronized object needs one.

**What it does**:
- Assigns a unique **ViewID** to the GameObject (synced across all clients)
- Tracks **ownership** (which client controls this object)
- Routes **RPCs** to the correct GameObject on other clients
- Manages **Observables** (components that sync data, like PhotonTransformView)

**Key fields** (Inspector):
- **Owner**: Who controls this object (Creator, Scene, Master Client, Fixed)
- **Observables**: List of components to sync (e.g., PhotonTransformView)

**Usage in code**:
```csharp
PhotonView pv = GetComponent<PhotonView>();
bool isMine = pv.IsMine; // Do I own this GameObject?
int ownerActorNumber = pv.Owner.ActorNumber; // Which player owns this?
```

**Why IsMine matters**:
- Only the **owner** should send input to move the object
- Other clients just **receive synced position** from owner
- Example:
```csharp
void Update()
{
    if (photonView.IsMine)
    {
        // I own this Actor — read input and move
        float h = Input.GetAxis("Horizontal");
        controller.Move(new Vector3(h, 0, 0) * speed * Time.deltaTime);
    }
    else
    {
        // Someone else owns this Actor — I just watch the synced position
    }
}
```

### 5.4 PhotonTransformView Component
**PhotonTransformView** syncs a GameObject's Transform (position, rotation, scale) across the network.

**How it works**:
- Owner client sends their Transform every few frames
- Other clients receive the data and **interpolate** to smooth position (avoids jitter from network lag)

**Settings** (Inspector):
- **Sync Position**: Should position be synced?
- **Sync Rotation**: Should rotation be synced?
- **Sync Scale**: Should scale be synced?
- **Interpolate**: Smooth movement between received updates (always enable for moving objects)

**Why use it**:
- Without PhotonTransformView, moving an object on your machine doesn't move it on other clients
- With PhotonTransformView, movement is automatically synced

**Add to PhotonView**:
1. Select GameObject with PhotonView
2. Inspector → PhotonView → Observables → Add Component → PhotonTransformView
3. Enable position/rotation sync

### 5.5 Remote Procedure Calls (RPCs)
**RPCs** let you call a method on a networked GameObject **across all clients** (or specific targets).

**How it works**:
1. You call `photonView.RPC("MethodName", target, parameters)`
2. Photon sends a message to the specified clients
3. Each client finds the GameObject with that PhotonView
4. Each client invokes the method with the same parameters

**Example**:
```csharp
// Method marked with [PunRPC]
[PunRPC]
void Die(int killerID)
{
    Debug.Log("Killed by player " + killerID);
    // Play death animation, disable controls, etc.
}

// Calling the RPC (runs on all clients)
photonView.RPC("Die", RpcTarget.All, myKillerID);
```

**RpcTarget options**:
- **RpcTarget.All**: Every client in the room (including yourself)
- **RpcTarget.Others**: Every client except yourself
- **RpcTarget.MasterClient**: Only the Master Client
- **RpcTarget.AllBuffered**: All + late-joiners receive it when they join

**Why use RPCs**:
- **Events**: Notify all clients something happened (player died, door opened)
- **Validation**: Master Client receives kill request, validates, then broadcasts result
- **Not for frequent updates**: Don't RPC every frame (use PhotonTransformView for position sync)

**Parameter types**:
- Only **serializable types**: int, float, string, Vector3, Quaternion, byte, bool
- **Not GameObject or Component** — use ViewID or ActorNumber instead

### 5.6 Room & Lobby System
**Lobby**:
- Waiting area before joining a room
- Can list available rooms (if enabled in Photon settings)
- You're not in a game yet

**Room**:
- The actual game session
- All players in the same room see each other's networked objects
- **Room properties**: Custom data (game mode, map, max players)
- **Player properties**: Custom data per player (team, score, skin)

**Flow**:
1. `PhotonNetwork.ConnectUsingSettings()` → Connect to Photon Cloud
2. `OnConnectedToMaster()` callback → Now connected
3. `PhotonNetwork.JoinLobby()` → Enter lobby
4. `OnJoinedLobby()` callback → Can now join/create rooms
5. `PhotonNetwork.JoinOrCreateRoom(...)` → Enter room (or create if doesn't exist)
6. `OnJoinedRoom()` callback → You're in the game, spawn player
7. Play the game
8. `PhotonNetwork.LeaveRoom()` → Return to lobby

**Callbacks**:
- Inherit from `MonoBehaviourPunCallbacks` to receive Photon events:
```csharp
public class MyNetworkManager : MonoBehaviourPunCallbacks
{
    public override void OnConnectedToMaster()
    {
        Debug.Log("Connected to Photon Cloud");
    }

    public override void OnJoinedRoom()
    {
        Debug.Log("Joined room, spawn player now");
    }
}
```

### 5.7 Ownership Transfer
**What**: Changing which client owns a networked GameObject.

**Why**: In advanced scenarios (not this game), you might transfer ownership dynamically. In POSSESSED, ownership is set at spawn and doesn't change — each player owns their own Actor for the entire match.

**How**:
```csharp
photonView.TransferOwnership(newOwnerPlayer);
```

**Example** (hypothetical):
1. An object is owned by Player A
2. Player B needs to take control
3. Call `obj.photonView.TransferOwnership(playerB)`
4. Now `obj.photonView.IsMine == true` for Player B
5. Player B can send input, control the object

**In POSSESSED**: You don't use ownership transfer. The Ghost is a virtual tag tracked by Master Client in `GhostGame.hostIdx`, not a physical object that changes ownership.

### 5.8 Network Instantiation Flow
**Standard Unity**:
```csharp
GameObject obj = Instantiate(prefab, position, rotation);
```
- Only creates on **your local machine**
- Other clients don't see it

**Photon PUN 2**:
```csharp
GameObject obj = PhotonNetwork.Instantiate("Actor", position, rotation, 0);
```
- Creates on **all clients in the room**
- Photon sends spawn message to others
- Other clients instantiate the same prefab at the same position
- **String name**: Must match the prefab name in **Resources/ folder**
- **Last parameter (0)**: Group number (usually 0, advanced feature for scene syncing)

**Prefab requirements**:
- Must have a **PhotonView** component
- Must be in **Resources/ folder** or a subfolder of Resources/
- **Name must match exactly** (case-sensitive)

**Ownership**:
- The client who calls `PhotonNetwork.Instantiate()` **owns** the object
- Owner's `photonView.IsMine == true`, others' is `false`

---

## 6. Why This Architecture?

### 6.1 Plain C# Core (GhostGame, Sight)
**Why not MonoBehaviour**?
- **Testability**: Plain C# classes can be unit-tested without Unity
- **Separation**: Game logic is independent of Unity rendering/input
- **Multiplayer-safe**: No hidden dependencies on Unity singletons (Camera.main, Time.time)

**GhostGame.cs**:
- Pure state machine: `Tuple<GState, IVoice>[] actors`, possession rules
- **No Unity**: No Transform, no GameObject, no Update()
- **Why**: The game rules are the same on every client. Network only syncs inputs/kills, not the entire game state.

**Sight.cs**:
- Pure math: cone calculations, distance checks
- **No Unity**: Just Vector3 math
- **Why**: Vision rules are deterministic. Given same positions, all clients compute same visibility.

### 6.2 Local-Only Rendering
**Why not sync lights/camera**?
- **Performance**: Syncing 8 lights + 8 cameras = network spam
- **Unnecessary**: Each client only needs **their own** camera + lights
- **LocalOnly pattern**: Camera/lights are children of Actor, but `SetActive(photonView.IsMine)` — only enabled for the actor you own

**How it works**:
1. Actor spawns on all clients
2. Everyone sees the Actor's CapsuleBody (synced)
3. Only **you** see your Actor's Camera/FocusLight/PeriphLight (`localOnlyRoot.SetActive(photonView.IsMine)`)
4. Other players' lights are disabled on your machine — you don't see them, they don't see yours

**Result**: 8 players = 8 cameras exist, but each client only activates 1 (their own).

### 6.3 Raycast-Based Vision
**Why not collider triggers**?
- **Walls**: Trigger volumes can't detect line-of-sight occlusion
- **Angles**: Hard to implement cone vision with triggers
- **Precision**: Raycasts give exact eye-to-target visibility

**How it works**:
1. VisionSystem.Update() loops through all actors
2. For each actor pair: `Sight.Check(observerPos, observerFwd, targetPos, isMoving)`
3. Returns Hidden/Silhouette/Full based on cone/distance
4. If not Hidden: `Physics.Linecast(eyes, target)` — if hits wall, downgrade to Hidden
5. Result: Per-frame, per-actor visibility state

**Why every frame**?
- Actors are moving → vision changes every frame
- **Optimization**: Could cache last N frames, but for 8 actors, it's negligible

### 6.4 Secret hostIdx
**Why not broadcast who's possessed**?
- **Hidden information**: Other players shouldn't know who the Ghost is possessing
- **Server authority**: Master Client knows ground truth (`GhostGame.hostIdx`), clients only know if THEY are possessed

**How it works**:
1. `GhostGame.hostIdx` is the secret — only Master Client tracks this
2. When Ghost transfers (on kill or timer expiry), Master Client picks new random host
3. Master Client sends targeted RPC to new host: "You are now possessed" (only that player receives it)
4. Other clients never learn who's possessed — they must deduce from behavior and audio cues (transfer bell plays from possessed player's position)

**Result**: Paranoia gameplay — you can't tell who's the Ghost until they act suspiciously.

---

## 7. Common Pitfalls for Beginners

### 7.1 Plane vs Cube Floor
**Wrong**:
```
GameObject → 3D Object → Plane
```
- **Problem**: Plane uses MeshCollider
- **CharacterController.isGrounded** fails on MeshCollider (needs BoxCollider/SphereCollider/CapsuleCollider)
- **Result**: `isGrounded` always false, can't jump properly

**Right**:
```
GameObject → 3D Object → Cube
Scale: (30, 0.1, 30)
```
- **Why**: Cube uses BoxCollider by default
- **CharacterController.isGrounded** works correctly

### 7.2 Camera.main Every Frame
**Wrong**:
```csharp
void Update()
{
    Camera cam = Camera.main; // SLOW: searches scene every frame
    Vector3 dir = cam.transform.forward;
}
```

**Right**:
```csharp
private Camera cam;
void Start()
{
    cam = Camera.main; // Cache once
    if (cam == null) Debug.LogError("No MainCamera!");
}
void Update()
{
    Vector3 dir = cam.transform.forward; // Fast: cached reference
}
```

**In POSSESSED**: Camera is under Actor → LocalOnly, so you cache it from there:
```csharp
void Start()
{
    cam = localOnlyRoot.GetComponentInChildren<Camera>();
}
```

### 7.3 NavMeshAgent + CharacterController Conflict
**Problem**: Both components try to control the same Transform.
- **NavMeshAgent**: Moves the GameObject along the NavMesh
- **CharacterController**: Moves the GameObject via `.Move()` or `.SimpleMove()`

**Result**: Jittering, stuttering movement.

**Solution**: Only enable one at a time.
```csharp
// Bot mode: NavMesh AI
navAgent.enabled = true;
controller.enabled = false;

// Player mode: Manual control
navAgent.enabled = false;
controller.enabled = true;
```

### 7.4 Prefabs Not in Resources/ Folder
**Problem**: `PhotonNetwork.Instantiate("Actor", ...)` fails silently or throws error.

**Why**: Photon PUN 2 requires networked prefabs to be in `Assets/Resources/` (or subfolder like `Assets/Resources/Prefabs/`).

**Solution**: Move Actor.prefab to `Assets/Resources/Actor.prefab`.

### 7.5 Forgetting PhotonView Component
**Problem**: Created prefab, added scripts, but `PhotonNetwork.Instantiate()` fails.

**Why**: Networked GameObjects must have a PhotonView component.

**Solution**: Select prefab → Add Component → Photon View.

### 7.6 RPC Method Name Typos
**Problem**: `photonView.RPC("TakeDammage", ...)` → no error, but method never called.

**Why**: RPC uses **string name**. Typo = silently fails.

**Solution**: Use `nameof(TakeDamage)` instead of `"TakeDamage"`:
```csharp
photonView.RPC(nameof(TakeDamage), RpcTarget.All, amount);
```

### 7.7 Modifying Non-Owned Objects
**Problem**: Trying to move another player's character directly:
```csharp
void Update()
{
    // This only moves the object on YOUR machine, not others
    otherPlayer.transform.position += Vector3.forward * Time.deltaTime;
}
```

**Why**: Each client only sends **their own** owned objects' state. Modifying someone else's object doesn't sync.

**Solution**: Use RPCs to notify the owner to move themselves, or use PhotonTransformView to sync the owner's movement.

---

## 8. Workflow Summary

**Scene setup** (Block 1-3):
1. Create Cube floor (BoxCollider), scale (30, 0.1, 30)
2. Bake NavMesh (AI → Navigation → Bake, or NavMesh Surface component)
3. Create Actor prefab in Resources/ with:
   - CapsuleBody (visual mesh)
   - CharacterController (movement + collision)
   - NavMeshAgent (bot pathfinding, disabled for players)
   - PhotonView (network sync)
   - PhotonTransformView (position/rotation sync)
   - LocalOnly empty (Camera, FocusLight, PeriphLight children)

**Core scripts** (Block 4-5):
1. Write plain C# (GhostGame.cs, Sight.cs, IVoiceLayer.cs) — no Unity dependencies
2. Write MonoBehaviours (Actor.cs, VisionSystem.cs, ArenaDirector.cs) — Unity integration

**Bot AI** (Block 6):
1. BotBrain.cs: NavMeshAgent pathfinding, random target selection
2. Enable NavMeshAgent, disable CharacterController for bots

**Audio** (Block 7):
1. Add AudioSource to Actor (footsteps, ambient)
2. AudioRouter.cs: Toggle AudioLowPassFilter for wall muffling (800Hz vs 22000Hz)

**Multiplayer** (Block 8):
1. PhotonView + PhotonTransformView on Actor
2. ArenaDirector: Connect → Join Room → Spawn Actor
3. TopDownController: `if (photonView.IsMine)` for input
4. Kill validation RPC: Client sends request, Master Client validates, broadcasts result

**Voice** (Block 9):
1. Vivox SDK setup (App ID)
2. VivoxVoice.cs: Join voice channel, set 3D position per Actor

**Build** (Block 11):
1. File → Build Settings → WebGL
2. Player Settings → WebGL Template: Minimal
3. Build → Upload to itch.io

---

**Read GUIDE-FULL.md for complete implementation details. This document explains the "why" and "what"; GUIDE-FULL.md explains the "how".**
