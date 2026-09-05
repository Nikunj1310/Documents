# POSSESSED — Learning Notes Index

**Welcome to your Obsidian vault for learning Unity while building POSSESSED.**

This vault is organized by topic, with detailed explanations of every component, method, and concept you'll use. Each note explains:
- **What it is** (beginner-friendly explanation)
- **Why we use it** (not alternatives)
- **How to use it** (methods, properties, common patterns)
- **Common mistakes** (what to avoid)

---

## 🎮 Game Design

- [[Game Rules]] — What you're building (Ghost mechanics, win conditions, vision tiers)
- [[Architecture Decisions]] — Why we structured the code this way

---

## 🔧 Unity Core Concepts

### Hierarchy & Components
- [[GameObject]] — The building block of Unity scenes
- [[Component]] — Adding functionality to GameObjects
- [[Transform]] — Position, rotation, scale, parent-child hierarchy
- [[Prefab]] — Reusable templates

### Scripting
- [[MonoBehaviour]] — Unity's script base class
- [[Lifecycle Methods]] — Awake, Start, Update, FixedUpdate, LateUpdate, OnDestroy
- [[Attributes]] — SerializeField, Header, RequireComponent, Tooltip, PunRPC

### Common Components
- [[CharacterController]] — Player movement with collision (no physics)
- [[NavMeshAgent]] — AI pathfinding
- [[Camera]] — Rendering the scene
- [[Light]] — Spot Light, Point Light, Directional Light
- [[AudioSource]] — Playing 3D audio
- [[AudioLowPassFilter]] — Wall muffling effect
- [[MeshRenderer]] — Making objects visible
- [[Collider]] — BoxCollider, SphereCollider, CapsuleCollider, MeshCollider

---

## 🌐 Photon PUN 2 Multiplayer

### Core Concepts
- [[PhotonNetwork API]] — Global multiplayer operations
- [[PhotonView]] — Network sync component
- [[PhotonTransformView]] — Auto-sync position/rotation
- [[Master Client]] — Server authority pattern
- [[RPC]] — Remote Procedure Calls
- [[Room and Lobby]] — Connection flow

### Multiplayer Patterns
- [[Ownership]] — Who controls which GameObject
- [[Kill Validation Pattern]] — Client claims, Master validates, broadcasts result
- [[Local-Only Rendering]] — Camera/lights enabled only for your actor

---

## 🧠 AI & Pathfinding

- [[NavMesh]] — Walkable surface for AI
- [[NavMesh Baking]] — Creating the NavMesh (two workflows)
- [[NavMeshAgent Methods]] — SetDestination, pathPending, remainingDistance

---

## 👁️ Vision System

- [[Vision Tiers]] — Hidden, Silhouette, Full
- [[Raycast]] — Line-of-sight checks
- [[Physics.Linecast]] — Wall occlusion detection
- [[Sight Evaluation]] — Cone angles, distances, peripheral vision

---

## 🎵 Audio System

- [[3D Audio]] — Spatial Blend, Volume Rolloff
- [[Audio Events]] — Scream, TransferBell, TimerWarning, etc.
- [[Wall Muffling]] — AudioLowPassFilter cutoff frequency

---

## 🖥️ UI System

- [[Canvas]] — Screen Space Overlay vs World Space
- [[TextMeshPro]] — Text rendering
- [[UI Panels]] — HUD, Possession UI, Defeat Screen, Results Screen

---

## 📝 Scripts Reference

### Core Scripts (Plain C#)
- [[GhostGame]] — Game state machine (possession, kills, timers)
- [[Sight]] — Vision tier math
- [[IVoiceLayer]] — Voice chat interface

### Game Layer Scripts (MonoBehaviours)
- [[Actor Script]] — Per-actor state and cached components
- [[ActorRegistry]] — Global actor list singleton
- [[ArenaDirector]] — Game lifecycle, Photon connection, event routing
- [[TopDownController]] — WASD input, mouse aiming, attacking
- [[VisionSystem]] — Tier evaluation and rendering
- [[BotBrain]] — AI pathfinding logic
- [[AudioRouter]] — Event-to-sound mapping, wall muffling
- [[UIRouter]] — HUD updates, screen management

---

## 🛠️ Unity APIs

- [[Time API]] — deltaTime, time, fixedDeltaTime
- [[Input API]] — GetKey, GetKeyDown, GetAxis, GetMouseButtonDown
- [[Physics API]] — Raycast, Linecast, LayerMask
- [[Vector3]] — Position math, distance, direction, dot product

---

## ⚠️ Common Mistakes

- [[Plane vs Cube Floor]] — Why Cube works with CharacterController
- [[Camera.main Performance]] — Cache it, don't call every frame
- [[NavMeshAgent + CharacterController Conflict]] — Only enable one
- [[Resources Folder Requirement]] — Photon prefabs must be here
- [[RPC Method Name Typos]] — Use nameof() to avoid silent failures

---

## 🚀 Build & Deploy

- [[WebGL Build Settings]] — Compression, template, player settings
- [[itch.io Upload]] — Creating HTML project, embed settings

---

## ✅ 11-Hour Timeline Checklist

- [[Block 1 - Project Setup]] — Photon, AI Navigation, folders (0:00-0:30)
- [[Block 2 - Scene Setup]] — Floor, walls, NavMesh, lighting (0:30-1:00)
- [[Block 3 - Actor Prefab]] — Components, hierarchy, LocalOnly (1:00-1:30)
- [[Block 4 - Core Scripts]] — GhostGame, Sight (1:30-2:30)
- [[Block 5 - Actor and Registry]] — Actor.cs, ActorRegistry.cs (2:30-3:30)
- [[Block 6 - ArenaDirector]] — Photon connection, GhostGame ownership (3:30-4:30)
- [[Block 7 - Player Input]] — TopDownController (4:30-5:30)
- [[Block 8 - Vision System]] — VisionSystem, tier rendering (5:30-7:00)
- [[Block 9 - Bot AI]] — BotBrain (7:00-8:00)
- [[Block 10 - Audio]] — AudioRouter (8:00-8:30)
- [[Block 11 - UI]] — Canvas, UIRouter (8:30-9:30)
- [[Block 12 - Multiplayer Testing]] — 2+ clients (9:30-10:00)
- [[Block 13 - Polish and Build]] — WebGL, itch.io (10:00-11:00)

---

**Start with [[Game Rules]] to understand what you're building, then follow the timeline blocks in order.**
