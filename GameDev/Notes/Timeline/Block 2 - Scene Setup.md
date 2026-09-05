# Block 2 - Scene Setup

**Time Budget:** 30 minutes (0:30–1:00)
**Goal:** Create arena with floor, walls, NavMesh, lighting

---

## What You're Building

The physical game world: collision geometry players cannot walk through, walls that block vision linecasts, and a baked NavMesh so bots can pathfind.

---

## Prerequisites

- [ ] Unity project created
- [ ] Photon PUN 2 imported
- [ ] AI Navigation package installed
- [ ] Layers created: `Wall`, `Ground`

All from [[Block 1 - Project Setup]].

---

## Step-by-Step Instructions

### 1. Create Arena Scene (2 min)

- [ ] File → Save Scene As → `Scenes/Arena.unity`

---

### 2. Create Floor (5 min)

- [ ] GameObject → 3D Object → **Cube** (NOT Plane)
- [ ] Name it `Floor`
- [ ] Set Transform:
  - Position: `(0, 0, 0)`
  - Scale: `(30, 0.1, 30)`
- [ ] Set Layer: `Ground`
- [ ] Set Material: Dark grey / near-black

**Why Cube not Plane:**
Plane ships with a [[MeshCollider]]. `CharacterController.isGrounded` is unreliable against MeshCollider. Cube ships with a [[BoxCollider]], which works correctly. Full explanation: [[Plane vs Cube Floor]].

**What is BoxCollider:**
A [[Collider]] component defining a box-shaped collision boundary. [[CharacterController]] uses it to detect ground contact for the `isGrounded` property.

Scale (30, 0.1, 30) gives a 30×30m arena with a 10cm slab. Surface sits at y = 0.05.

---

### 3. Create Walls (8 min)

- [ ] GameObject → 3D Object → Cube (create 4–8 walls, duplicate with Ctrl+D)
- [ ] For each wall:
  - Scale: `(10, 2.5, 0.2)` — long axis varies per wall
  - Layer: `Wall`
  - Material: Mid-grey (contrast against floor)
- [ ] Position 4 walls around the 30×30 perimeter
- [ ] Position 4–6 interior walls forming **4–5 rooms**
- [ ] Create doorways: leave **2–3 unit gaps**, at least **2 doorways per room**
- [ ] Add 2–4 pillars: Cube scaled ~`(1, 2.5, 1)` inside rooms

**Why 2.5m tall:**
Eye height is 1.5m ([[EyeAnchor]]), so 2.5m walls reliably block vision linecasts. Shorter walls let players see over them.

**Why loops matter:**
Aim for **2–3 loops** (multiple paths between rooms). A tree layout (one path per room) creates dead ends with no escape. Kill range is only 2m, so escape must be possible. Pillars create ambush corners and break line-of-sight at close range.

---

### 4. Bake NavMesh (10 min)

**What is [[NavMesh]]:**
A baked, simplified walkable surface. [[NavMeshAgent]] uses it to pathfind around obstacles. Without a baked NavMesh, `SetDestination()` silently fails and bots stand still.

Open **Window → AI → Navigation** and check which tabs exist.

#### Path A — Unity 2022.2+ (AI Navigation package installed)

- [ ] Create empty GameObject → Name: `NavMesh`
- [ ] Add Component → **NavMesh Surface**
- [ ] Set Settings:
  - Collect Objects: `All`
  - Use Geometry: `Render Meshes`
  - Agent Type: `Humanoid`
- [ ] Click **Bake** button
- [ ] **Verify:** Blue overlay on floor in Scene view

Static flags are **not used** in this workflow. Skip them.

#### Path B — Older Unity (a "Bake" tab exists)

- [ ] Select Floor + All Walls (click first, Ctrl+click rest)
- [ ] Inspector top-right → Static dropdown → **Navigation Static**
- [ ] Window → AI → Navigation → **Bake** tab
- [ ] Leave defaults: Agent Radius 0.5, Height 2.0, Max Slope 45, Step Height 0.4
- [ ] Click **Bake**
- [ ] **Verify:** Blue overlay on floor in Scene view

The default **Humanoid** agent type is already Radius 0.5 / Height 2.0, matching the Actor prefab. Change nothing.

**Common Issue:**
Blue overlay insets ~0.5m from every wall → **This is normal.** That is agent radius. Bots cannot hug walls exactly.

---

### 5. Configure Lighting (5 min)

#### Directional Light (already in scene)

- [ ] Set Rotation: `(50, -30, 0)` — angled top-down light
- [ ] Set Intensity: `1`
- [ ] Set Color: Slightly warm white

#### Environment Settings

- [ ] Window → Rendering → Lighting → Environment tab
- [ ] Set Skybox: Default or black
- [ ] Set Ambient Intensity: `0` — **pitch-black shadows**

**Why Ambient Intensity 0:**
Horror aesthetic requires darkness. Unlit areas should be completely black, which makes the spotlight and point light on each Actor matter. Default ambient makes everything readable grey.

**Do not delete the scene Main Camera.** [[ArenaDirector]] disables it at runtime; each player renders through the camera on their own Actor. See [[Local-Only Rendering]].

---

### 6. Walk Test (4 min)

- [ ] Drop a temporary Cube in the scene
- [ ] Add a [[CharacterController]] component
- [ ] Attach a throwaway WASD script
- [ ] Press Play and verify:
  - [ ] You stand on the floor (do not fall through)
  - [ ] You cannot walk through walls
  - [ ] You can pass through every doorway
  - [ ] No room is a dead end
- [ ] Delete the test cube

---

## Verification Checklist

- [ ] Floor exists with BoxCollider, layer is `Ground`
- [ ] 4–8 walls exist with layer `Wall`
- [ ] Walls form 4–5 rooms with doorways
- [ ] 2–3 loops exist (multiple paths between rooms)
- [ ] NavMesh blue overlay visible on floor
- [ ] Blue overlay insets from walls (expected)
- [ ] Ambient lighting is 0
- [ ] Scene is playable, no clipping through geometry

---

## Time Breakdown

| Task | Time |
|---|---|
| Create scene | 2 min |
| Floor setup | 5 min |
| Walls + pillars | 8 min |
| NavMesh baking | 10 min |
| Lighting | 5 min |
| Walk test | 4 min |
| **Total** | **34 min** |

**Behind schedule?** Ship 4 boundary walls + 2 interior walls + 2 pillars. A smaller arena still plays. Do **not** skip the NavMesh bake — bots depend on it.

---

## Troubleshooting

| Problem | Solution |
|---|---|
| Blue overlay not appearing | Select the `NavMesh` object (Path A) or open the Navigation window (Path B) — the overlay only draws while navigation display is active |
| Blue overlay covers wall tops | Walls lack colliders, or Use Geometry is set to Physics Colliders |
| NavMesh doesn't avoid walls | Ensure walls have colliders enabled |
| Floor too dark | Increase Directional Light intensity, keep Ambient at 0 |
| CharacterController falls through floor | Use Cube not Plane, ensure BoxCollider exists |

---

## Common Mistakes

- [[Plane vs Cube Floor]] — using Plane breaks `CharacterController.isGrounded`
- Forgetting to bake NavMesh — bots stand still, `SetDestination` silently fails
- Not setting Wall/Ground layers — vision system and mouse aiming break

---

## Quick Reference

```csharp
// Check if a NavMesh position is valid near a point
if (NavMesh.SamplePosition(randomPos, out NavMeshHit hit, 5f, NavMesh.AllAreas)) {
    // Valid NavMesh position found in hit.position
}

// Get layer index from name
int wallLayer = LayerMask.NameToLayer("Wall");
```

---

## Related Notes

- [[Transform]] — GameObject positioning
- [[Collider]] — collision detection, BoxCollider vs MeshCollider
- [[NavMesh]] — AI pathfinding surface
- [[NavMesh Baking]] — both workflows in detail
- [[Light]] — Directional, Spot, Point
- [[Plane vs Cube Floor]] — why the floor must be a Cube
- [[Game Rules]] — why arena size and loops matter
- [[Block 1 - Project Setup]] — previous block
- [[Block 3 - Actor Prefab]] — next block
