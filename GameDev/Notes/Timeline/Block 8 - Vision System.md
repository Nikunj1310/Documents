# Block 8 - Vision System

**Time Budget:** 90 minutes (5:30–7:00)
**Goal:** Write VisionSystem.cs — evaluates vision tiers, applies to renderers, animates cone

---

## What You're Building

The gameplay truth layer. Vision tier determines:
- **Hidden:** Cannot see actor, cannot kill
- **Silhouette:** See anonymous black shape, cannot kill
- **Full:** See actor + name, CAN kill

**This is not cosmetic.** Lighting/shadows are aesthetic. Vision tier is validated server-side for kill authorization.

---

## Prerequisites

- [ ] [[Block 7 - Player Input]] complete
- [ ] Understanding of [[Vision Tiers]]
- [ ] Understanding of [[Sight Script]]
- [ ] Created Silhouette material (unlit black)

---

## Checklist

- [ ] Create Silhouette material (Shader: Unlit/Color, Color: Black)
- [ ] Create `Scripts/Game/VisionSystem.cs`
- [ ] Attach to Actor.prefab
- [ ] Test: Other actors hidden behind walls
- [ ] Test: Other actors appear as silhouettes within 4.5m peripheral
- [ ] Test: Other actors appear with names in focus cone
- [ ] Test: Cone widens when you stop moving

---

## Vision Tier Rules (from Sight.cs)

```
Focus Cone (moving):  40° half-angle, 14m range → VisTier.Full
Focus Cone (still):   62° half-angle, 16m range → VisTier.Full
Peripheral Sphere:    4.5m radius (360°)        → VisTier.Silhouette
Outside both:                                    → VisTier.Hidden
Wall blocks LOS:                                 → VisTier.Hidden (override)
```

See [[Vision Tiers]] for full rules.

---

## Key Responsibilities

### 1. Evaluate Vision Tiers (Every Frame)

```csharp
void Update() {
    if (!myActor.photonView.IsMine) return;  // Only owner evaluates
    
    // Detect movement
    float speed = (transform.position - lastPos).magnitude / Time.deltaTime;
    lastPos = transform.position;
    bool moving = speed > 0.15f;  // Threshold: small jitter doesn't count
    
    // Animate cone (slow open, fast close)
    float targetHalfAngle = moving ? narrowHalfAngle : wideHalfAngle;
    float targetRange = moving ? narrowRange : wideRange;
    float lerpSpeed = moving ? closeSpeed : openSpeed;
    
    currentHalfAngle = Mathf.Lerp(currentHalfAngle, targetHalfAngle, lerpSpeed * Time.deltaTime);
    currentRange = Mathf.Lerp(currentRange, targetRange, lerpSpeed * Time.deltaTime);
    
    // Update spotlight (cosmetic, matches gameplay cone)
    focusLight.spotAngle = currentHalfAngle * 2f;  // Unity uses full angle, not half
    focusLight.range = currentRange;
    
    // Evaluate tier for every other actor
    foreach (Actor other in ActorRegistry.Instance.AllActors) {
        if (other == myActor) continue;
        
        VisTier tier = Sight.Evaluate(
            myActor.EyePosition,
            transform.forward,
            other.EyePosition,
            moving
        );
        
        // Override: wall blocks = Hidden (even if math says Silhouette/Full)
        if (tier != VisTier.Hidden) {
            if (Physics.Linecast(myActor.EyePosition, other.EyePosition, wallLayerMask)) {
                tier = VisTier.Hidden;
            }
        }
        
        // Store tier (used by TopDownController for kill validation)
        other.currentTier = tier;
        
        // Apply to renderer
        ApplyTier(other, tier);
    }
}
```

**Why slow open, fast close:**
- Peeking around corner = commitment (must wait ~0.5s for cone to open)
- If instant, players spam crouch-peek with no cost
- Fast close = responsive when moving again

**Why check movement threshold (0.15f):**
- Avoids cone flickering from tiny position jitter
- CharacterController has small drift even when idle

### 2. Apply Vision Tier to Renderer

```csharp
void ApplyTier(Actor actor, VisTier tier) {
    switch (tier) {
        case VisTier.Hidden:
            actor.bodyRenderer.enabled = false;
            actor.nameTag.gameObject.SetActive(false);
            break;
            
        case VisTier.Silhouette:
            actor.bodyRenderer.enabled = true;
            actor.bodyRenderer.material = silhouetteMaterial;
            actor.nameTag.gameObject.SetActive(false);
            break;
            
        case VisTier.Full:
            actor.bodyRenderer.enabled = true;
            actor.bodyRenderer.material = actor.originalMaterial;
            actor.nameTag.gameObject.SetActive(true);
            break;
    }
}
```

**Why disable renderer, not rely on lighting:**
- Lighting is cosmetic. Player can crank monitor gamma to see through darkness.
- Disabling renderer is cheat-proof (nothing to render = invisible).

**Why swap material:**
- Silhouette must be flat black (no texture, no shading)
- Unlit shader = ignores lighting (always pure black)

---

## Create Silhouette Material

1. Project → Assets/Materials/ → Right-click → Create → Material
2. Name: `Silhouette`
3. Shader: Unlit/Color (dropdown at top of Inspector)
4. Color: Black (RGB: 0, 0, 0)
5. Save

**What is Unlit shader:** Ignores all lights, displays pure color. See [[Shader]].

---

## Full Script Structure

```csharp
using UnityEngine;
using Possessed;

public class VisionSystem : MonoBehaviour {
    [Header("Cone Animation")]
    public float narrowHalfAngle = 40f;
    public float wideHalfAngle = 62f;
    public float narrowRange = 14f;
    public float wideRange = 16f;
    public float openSpeed = 4f;   // Slow open
    public float closeSpeed = 12f;  // Fast close
    
    [Header("Peripheral")]
    public float periphRange = 4.5f;
    
    [Header("References")]
    public Material silhouetteMaterial;
    public Light focusLight;
    public Light periphLight;
    public LayerMask wallLayerMask;
    
    Actor myActor;
    Vector3 lastPos;
    float currentHalfAngle;
    float currentRange;
    
    void Awake() {
        myActor = GetComponent<Actor>();
        currentHalfAngle = narrowHalfAngle;
        currentRange = narrowRange;
    }
    
    void Start() {
        lastPos = transform.position;
        focusLight = myActor.focusLight;
        periphLight = myActor.periphLight;
    }
    
    void Update() {
        if (!myActor.photonView.IsMine) return;
        
        // ... cone animation, tier evaluation, ApplyTier ...
    }
    
    void ApplyTier(Actor actor, VisTier tier) { /* ... */ }
}
```

See **GUIDE-FULL.md §8.1** for complete implementation.

---

## Attach to Actor.prefab

1. Open Actor.prefab
2. Select root
3. Add Component → Vision System
4. Inspector:
   - Narrow Half Angle: 40
   - Wide Half Angle: 62
   - Narrow Range: 14
   - Wide Range: 16
   - Open Speed: 4
   - Close Speed: 12
   - Periph Range: 4.5
   - Silhouette Material: Drag Silhouette.mat from Project
   - Wall Layer Mask: Select "Wall" layer
5. Save prefab

---

## Verification Checklist

- [ ] VisionSystem.cs compiles
- [ ] Silhouette.mat exists (Unlit/Color, Black)
- [ ] VisionSystem attached to Actor.prefab
- [ ] Play mode (solo): Spawn 3 dummy actors in scene
- [ ] Test: Actor behind wall is invisible (renderer disabled)
- [ ] Test: Actor within 4.5m (any direction) appears as black silhouette
- [ ] Test: Actor in front cone (<40°) appears with full color + name
- [ ] Test: Stand still → cone widens over ~0.5s (spotlight angle increases)
- [ ] Test: Move → cone narrows immediately (spotlight angle decreases)

**Test setup for solo verification:**
1. Temporarily disable `if (!myActor.photonView.IsMine) return;` check
2. Manually place 3 Actors in scene at (5,1,0), (0,1,5), (-5,1,0)
3. Disable their PhotonView components (so they don't network spawn)
4. Play mode → observe vision tiers
5. Re-enable the check after testing

---

## Common Mistakes

### 1. Using Full Spotlight Angle Instead of Half

**Wrong:**
```csharp
focusLight.spotAngle = currentHalfAngle;  // Cone half as wide as intended
```

**Right:**
```csharp
focusLight.spotAngle = currentHalfAngle * 2f;  // Unity uses full cone angle
```

Unity Spot Light `spotAngle` is the full cone angle (tip to opposite edge), not half-angle.

### 2. Not Overriding with Wall Linecast

**Wrong:**
```csharp
VisTier tier = Sight.Evaluate(...);
other.currentTier = tier;  // Wall doesn't block, you can kill through walls
```

**Right:**
```csharp
VisTier tier = Sight.Evaluate(...);
if (tier != VisTier.Hidden && Physics.Linecast(...)) {
    tier = VisTier.Hidden;  // Wall always blocks
}
other.currentTier = tier;
```

### 3. Applying Tier to Self

**Wrong:**
```csharp
foreach (Actor other in AllActors) {
    ApplyTier(other, ...);  // Self becomes invisible
}
```

**Right:**
```csharp
foreach (Actor other in AllActors) {
    if (other == myActor) continue;  // Skip self
    ApplyTier(other, ...);
}
```

### 4. Setting material Instead of sharedMaterial

For runtime changes per-instance, use `material` (creates instance). For prefab default, use `sharedMaterial`. Here we want per-instance (each actor sees different tiers), so `material` is correct.

```csharp
actor.bodyRenderer.material = silhouetteMaterial;  // Correct: per-instance
```

---

## Performance Notes

**Cost per frame:**
- N actors × (1 Sight.Evaluate + 1 Linecast) = 9 actors × 10 ops = ~90 ops/frame
- Acceptable for jam scope (≤10 players)

**Optimization (post-jam):**
- Throttle to 10Hz (every 6th frame): `if (Time.frameCount % 6 != 0) return;`
- Spatial hash: only evaluate actors within max vision range (16m)

---

## Time Breakdown

| Task | Time |
|---|---|
| Create Silhouette material | 5 min |
| Write cone animation | 20 min |
| Write tier evaluation loop | 20 min |
| Write ApplyTier method | 15 min |
| Attach to prefab, configure | 10 min |
| Test with dummy actors | 15 min |
| Fix bugs | 5 min |
| **Total** | **90 min** |

**Behind schedule?** Cut cone animation (ship with fixed 40° cone, no widen-when-still). Saves 20 min. Vision tiers themselves are **hard gate**.

---

## What You Built

The gameplay truth layer:
- Vision tier evaluation (Hidden/Silhouette/Full)
- Renderer manipulation (disable/swap material/restore)
- Cone animation (narrow moving, wide still)
- Wall occlusion (linecast override)

**Next:** [[Block 9 - Bot AI]] — autonomous actors

---

## Related Notes

- [[Vision Tiers]]
- [[Sight Script]]
- [[Physics API]] (Linecast)
- [[Light Component]] (Spot Light)
- [[Shader]] (Unlit)
- [[Material]]
- [[Block 7 - Player Input]]
- [[Block 9 - Bot AI]]
