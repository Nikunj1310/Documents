# Block 4 - Core Scripts

**Time Budget:** 60 minutes (1:30–2:30)
**Goal:** Write GhostGame.cs, Sight.cs, IVoiceLayer.cs — plain C# game logic with zero Unity dependencies

---

## What You're Building

The **game rules engine**. These scripts handle possession state, kill validation, timer, win conditions, and vision tier math. Pure C# = testable without Unity, same logic works for mobile version later.

---

## Prerequisites

- [ ] Scripts/Core/ folder exists
- [ ] Understanding of [[Game Rules]]

---

## Checklist

- [ ] Create `Scripts/Core/GhostGame.cs` — game state machine
- [ ] Create `Scripts/Core/Sight.cs` — vision tier math
- [ ] Create `Scripts/Core/IVoiceLayer.cs` — voice chat interface
- [ ] All three files compile with no errors
- [ ] Test compilation: return to Unity, check Console

---

## 1. Create GhostGame.cs (30 min)

**Location:** `Assets/Scripts/Core/GhostGame.cs`

This is the core state machine. Copy the complete implementation from **GUIDE-FULL.md §6.1**.

**Key Types:**
```csharp
public enum GameEvent { Scream, TransferBell, TimerWarning, HostExecuted, GhostShattered }
public enum Phase { Lobby, Hunt, Results }
public enum Ending { None, SurvivorsWin, GhostWins, NoWinner }
```

**Key State:**
```csharp
public int hostIdx = -1;              // who is possessed (-1 = nobody)
public bool[] alive;                  // 8 players = 8 bools
public float possessTimer;            // countdown (90s default)
public int payoutKillerIdx = -1;      // for GhostWins outcome
```

**Key Methods:**
- `Begin(int playerCount)` — Start match, pick first Ghost randomly
- `Tick(float dt)` — Update timer, fire warning/expiry events
- `TryKill(killerIdx, victimIdx, distance)` — Validate kill, apply result, return bool
- `TransferGhost(excludeIdx)` — Pick random living player, reset timer, fire TransferBell event

**Event System:**
```csharp
public event Action<GameEvent, int> OnEvent;
```
[[ArenaDirector]] subscribes to this. When GhostGame fires `OnEvent(GameEvent.Scream, victimIdx)`, ArenaDirector routes it to [[AudioRouter]] (play scream from victim position) and [[UIRouter]] (update HUD).

**Why plain C#:**
- Testable without Unity (unit tests in another project)
- Same rules for PC and mobile versions
- Easier to reason about (state changes in one place)

**Reference:** [[GhostGame Script]], [[Game Rules]]

---

## 2. Create Sight.cs (15 min)

**Location:** `Assets/Scripts/Core/Sight.cs`

Pure math for vision tier evaluation. Copy from **GUIDE-FULL.md §6.2**.

**Key Method:**
```csharp
public static VisTier Evaluate(
    Vector3 observerPos, 
    Vector3 observerFwd, 
    Vector3 targetPos, 
    bool observerMoving
)
```

**Logic:**
1. Check **focus cone** (40° / 14m moving, 62° / 16m still) → return Full if inside
2. Check **peripheral sphere** (4.5m radius, any direction) → return Silhouette if inside
3. Otherwise → return Hidden

**Why pure Vector3 math:**
- No Unity dependencies (works in unit tests)
- Deterministic (same inputs = same output on all clients)
- Fast (just dot product + magnitude checks)

[[VisionSystem]] calls this for every other actor every frame, then does `Physics.Linecast` to check walls.

**Reference:** [[Sight Evaluation]], [[Vision Tiers]]

---

## 3. Create IVoiceLayer.cs (5 min)

**Location:** `Assets/Scripts/Core/IVoiceLayer.cs`

Interface for voice chat systems. Copy from **GUIDE-FULL.md §6.3**.

```csharp
public interface IVoiceLayer {
    void MuteLocal(bool muted);
}

public class VoiceStub : IVoiceLayer {
    public void MuteLocal(bool muted) { /* no-op */ }
}
```

[[GhostGame]] calls `MuteLocal(true)` when you die. For now, use the stub implementation (does nothing). [[Block 11 - UI]] optionally integrates Vivox for real voice chat.

---

## Verification

- [ ] All three files exist in `Assets/Scripts/Core/`
- [ ] Return to Unity editor
- [ ] Check Console for compilation errors
- [ ] If errors: fix syntax, save, wait for recompile

**Common errors:**
- Missing `using System;` for `Action<>`
- Missing `using System.Linq;` for `.Where()` / `.Count()`
- Missing namespace declaration (wrap in `namespace Possessed { ... }`)

---

## Time Breakdown

| Task | Time |
|---|---|
| GhostGame.cs | 30 min |
| Sight.cs | 15 min |
| IVoiceLayer.cs | 5 min |
| Verification | 10 min |
| **Total** | **60 min** |

**Behind schedule?** These three files are **hard gates**. You cannot skip them. Cut time from [[Block 10 - Audio]] or [[Block 11 - UI]] instead.

---

## What You Built

The game rules engine:
- GhostGame tracks possession, validates kills, fires events
- Sight determines vision tiers (who can see whom)
- IVoiceLayer provides voice chat contract

**No Unity integration yet** — that starts in [[Block 5 - Actor and Registry]].

---

## Next

**→ [[Block 5 - Actor and Registry]]** — MonoBehaviours that bridge core logic to Unity scene

---

## Related Notes

- [[GhostGame Script]] — detailed API reference
- [[Sight Evaluation]] — cone math explained
- [[Game Rules]] — the game design these scripts implement
- [[Vision Tiers]] — Hidden, Silhouette, Full
- [[MonoBehaviour]] — the Unity bridge layer (next block)
