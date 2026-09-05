# Block 3: Actor Prefab

**Time Budget:** 30 minutes (1:00–1:30)  
**Goal:** Create Actor prefab with all components (visual, movement, network, local-only camera/lights)

## What You're Building

The player/bot character template. Every actor in the game is an instance of this [[Prefab]]. This single prefab handles both players (with input) and bots (with AI).

## Prerequisites

- [ ] Scene setup complete ([[Block-02-Scene-Setup]])
- [ ] Resources/ folder exists
- [ ] Understanding of [[GameObject]], [[Component]], [[Prefab]]

## Hierarchy Structure

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

## Critical Camera Setup

**IMPORTANT:** [[Camera]] is a **child of Actor → LocalOnly**, NOT a separate GameObject in scene. No CameraRig. No CameraFollow script. Camera follows Actor automatically via parent-child hierarchy.

## Step-by-Step Instructions

### 1. Create Root GameObject (2 min)

- [ ] Hierarchy → Create Empty
- [ ] Name: `Actor`
- [ ] Position: `(0, 1, 0)`

### 2. Add CapsuleBody Visual (3 min)

- [ ] Right-click Actor → 3D Object → Capsule
- [ ] Name: `CapsuleBody`
- [ ] Position: `(0, 0, 0)` relative to Actor
- [ ] Remove [[CapsuleCollider]] component (CharacterController handles collision)

**Why remove CapsuleCollider:**
- [[CharacterController]] already provides collision
- Double colliders cause physics conflicts
- CapsuleCollider on mesh is redundant

### 3. Add EyeAnchor (2 min)

- [ ] Right-click Actor → Create Empty
- [ ] Name: `EyeAnchor`
- [ ] Position: `(0, 1.5, 0)` relative to Actor

**What is EyeAnchor:**
Empty [[Transform]] at eye level. All vision linecasts and range checks use `EyeAnchor.position`, never `transform.position`.

**Why needed:**
Ground-level linecasts (y=0) hit the floor and always return "blocked". Eye-level linecasts work correctly.

### 4. Add NameTag (3 min)

- [ ] Right-click Actor → UI → Text - TextMeshPro (world-space)
- [ ] Name: `NameTag`
- [ ] Position: `(0, 2.5, 0)` relative to Actor
- [ ] Set Font Size: `2`
- [ ] Set Alignment: Center/Middle
- [ ] Add Billboard script (see below)

**Billboard Script:**
```csharp
using UnityEngine;
public class Billboard : MonoBehaviour {
    Camera cam;
    void Start() { cam = Camera.main; }
    void LateUpdate() { 
        if (cam) { 
            transform.LookAt(cam); 
            transform.Rotate(0, 180, 0); 
        }
    }
}
```

Save as `Scripts/Billboard.cs`, attach to NameTag.

**What it does:** Rotates text to always face camera (readable from any angle).

### 5. Add Movement Components (5 min)

#### CharacterController:
- [ ] Select Actor root → Add Component → [[CharacterController]]
- [ ] Set Radius: `0.5`
- [ ] Set Height: `2.0`
- [ ] Set Center: `(0, 1, 0)`

**What is CharacterController:**
Unity component for player movement with collision, without full physics simulation. Better for player control than [[Rigidbody]].

#### NavMeshAgent:
- [ ] Add Component → [[NavMeshAgent]]
- [ ] Set Speed: `3.5`
- [ ] Set Radius: `0.5`
- [ ] Set Height: `2.0`
- [ ] **DISABLE the component** (uncheck checkbox)

**CRITICAL:** [[NavMeshAgent]] + [[CharacterController]] conflict. Only enable one at a time:
- **Bots:** NavMeshAgent ON, CharacterController OFF
- **Players:** CharacterController ON, NavMeshAgent OFF

See [[Common-Mistake-NavMeshAgent-CharacterController-Conflict]]

### 6. Add Audio Components (3 min)

- [ ] Add Component → [[AudioSource]]
- [ ] Set Spatial Blend: `1.0` (3D)
- [ ] Set Play On Awake: `OFF`
- [ ] Add Component → [[AudioLowPassFilter]]
- [ ] Set Cutoff Frequency: `22000` (default)

**What is AudioSource:**
Plays audio clips in 3D space. Spatial Blend 1.0 means sound comes from actor position (closer = louder).

**What is AudioLowPassFilter:**
Cuts high frequencies above cutoff. 22000 Hz = no filtering. 800 Hz = heavy muffling (through walls).

### 7. Add Photon Network Components (4 min)

- [ ] Add Component → [[PhotonView]]
- [ ] Add Component → [[PhotonTransformView]]
- [ ] In [[PhotonView]] Inspector:
  - Observables: Add [[PhotonTransformView]]
  - Set Sync Position: `ON`
  - Set Sync Rotation: `ON`
  - Set Interpolate: `ON`

**What is PhotonView:**
Core component for networked GameObjects. Assigns unique ViewID, tracks ownership, routes RPCs.

**What is PhotonTransformView:**
Auto-syncs Transform (position/rotation) across network. Owner sends updates, other clients interpolate.

### 8. Create LocalOnly Branch (3 min)

- [ ] Right-click Actor → Create Empty
- [ ] Name: `LocalOnly`
- [ ] Position: `(0, 0, 0)` relative to Actor

**What is LocalOnly:**
Container for rendering only the owning client should see (camera, lights). At runtime: `localOnlyRoot.SetActive(photonView.IsMine)`.

**Why needed:**
- Performance: 1 actor with 2 shadow lights = fine. 8 actors with 16 shadow lights = slideshow in WebGL.
- Logic: Only you need to see YOUR vision cone. Other players are just lit capsules.

See [[Pattern-Local-Only-Rendering]]

### 9. Add Camera Under LocalOnly (4 min)

- [ ] Right-click LocalOnly → [[Camera]]
- [ ] Set Position: `(0, 18, -8)` relative to Actor
- [ ] Set Rotation: `(70, 0, 0)` — pitch down 70°
- [ ] Set Tag: **Untagged** (NOT MainCamera)
- [ ] Set Clear Flags: `Skybox`
- [ ] Set Field of View: `60`

**CRITICAL:** Camera is **child of Actor**, NOT separate GameObject. Moves automatically with Actor (parent-child hierarchy).

**Why child of Actor:**
- Zero scripts needed for camera following
- Camera moves/rotates with Actor automatically
- Multiplayer-safe: Each client only enables their own camera

### 10. Add FocusLight Under LocalOnly (2 min)

- [ ] Right-click LocalOnly → Light → Spot Light
- [ ] Name: `FocusLight`
- [ ] Set Position: `(0, 1.5, 0)` relative to Actor
- [ ] Set Type: `Spot`
- [ ] Set Range: `14`
- [ ] Set Spot Angle: `80` (animated by [[VisionSystem]])
- [ ] Set Intensity: `3-5`
- [ ] Set Shadows: `Soft Shadows`
- [ ] Set Color: `White`

**What is Spot Light:**
Cone-shaped light (like flashlight). Matches vision cone angle/range. Shadows show wall occlusion.

### 11. Add PeriphLight Under LocalOnly (2 min)

- [ ] Right-click LocalOnly → Light → Point Light
- [ ] Name: `PeriphLight`
- [ ] Set Position: `(0, 1.5, 0)` relative to Actor
- [ ] Set Range: `4.5`
- [ ] Set Intensity: `1-2`
- [ ] Set Shadows: `Soft Shadows`

**What is Point Light:**
Omnidirectional light (like light bulb). Shows 4.5m peripheral vision sphere (silhouette tier).

### 12. Save as Prefab (1 min)

- [ ] Drag Actor from Hierarchy → **Assets/Resources/** folder
- [ ] **MUST be in Resources/** — `PhotonNetwork.Instantiate("Actor", ...)` only works here
- [ ] Delete Actor from Hierarchy (spawned at runtime)

**Why Resources/ folder:**
[[Photon PUN 2]] limitation. Networked prefabs must be in Resources/ or subfolder. See [[Common-Mistake-Prefabs-Not-In-Resources]]

## Component Summary Table

| Component | Settings | Purpose |
|---|---|---|
| [[CharacterController]] | Radius 0.5, Height 2.0 | Player movement + collision |
| [[NavMeshAgent]] | Speed 3.5, **Disabled** | Bot pathfinding (enable for bots only) |
| [[AudioSource]] | Spatial Blend 1.0 | 3D positional audio |
| [[AudioLowPassFilter]] | Cutoff 22000 Hz | Wall muffling effect |
| [[PhotonView]] | Observables: PhotonTransformView | Network sync, ownership |
| [[PhotonTransformView]] | Sync Position/Rotation | Auto-sync transform |

## Verification Checklist

- [ ] Actor.prefab exists in Resources/ folder
- [ ] Hierarchy matches structure above
- [ ] Camera is child of LocalOnly (not separate GameObject)
- [ ] CapsuleBody has NO CapsuleCollider
- [ ] NavMeshAgent component is DISABLED by default
- [ ] PhotonView has PhotonTransformView in Observables list
- [ ] EyeAnchor at y=1.5 (eye level)
- [ ] NameTag at y=2.5 (above head)
- [ ] FocusLight and PeriphLight have shadows enabled

## Time Breakdown

| Task | Time |
|---|---|
| Root + CapsuleBody | 5 min |
| EyeAnchor + NameTag | 5 min |
| Movement components | 5 min |
| Audio components | 3 min |
| Photon components | 4 min |
| LocalOnly + Camera | 7 min |
| Lights | 4 min |
| Save prefab | 1 min |
| **Total** | **30 min** |

## Common Mistakes

- [[Common-Mistake-Camera-Separate-GameObject]] — Camera should be child of Actor
- [[Common-Mistake-NavMeshAgent-CharacterController-Conflict]] — Both enabled causes jittering
- [[Common-Mistake-Prefabs-Not-In-Resources]] — PhotonNetwork.Instantiate fails
- [[Common-Mistake-Forgetting-PhotonView]] — Networked objects need PhotonView

## Related Notes

- [[GameObject]] — Building blocks of Unity
- [[Component]] — Functionality attached to GameObjects
- [[Prefab]] — Reusable templates
- [[Transform]] — Position, rotation, scale
- [[CharacterController]] — Player movement
- [[NavMeshAgent]] — AI pathfinding
- [[Camera]] — Rendering viewpoint
- [[Light]] — Scene illumination
- [[AudioSource]] — Sound playback
- [[AudioLowPassFilter]] — Audio filtering
- [[PhotonView]] — Network synchronization
- [[PhotonTransformView]] — Transform syncing
- [[Pattern-Local-Only-Rendering]] — Performance optimization
- [[Block-04-Core-Scripts]] — Next block

## Quick Reference

```csharp
// Enable LocalOnly for owner only
localOnlyRoot.SetActive(photonView.IsMine);

// Get eye position for vision checks
Vector3 eyePos = eyeAnchor.position;

// Toggle movement mode
if (isBot) {
    navAgent.enabled = true;
    controller.enabled = false;
} else {
    navAgent.enabled = false;
    controller.enabled = true;
}
```
