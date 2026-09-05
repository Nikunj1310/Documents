# Block 11 - UI System

**Time Budget:** 60 minutes (8:30–9:30)
**Goal:** Build Canvas + write UIRouter.cs — HUD, possession panel, defeat screen, results screen

---

## What You're Building

Visual feedback. The possession panel is the **most important UI in the game** — it is the only thing that tells you the Ghost is inside you and the timer is running.

---

## Prerequisites

- [ ] [[Block 6 - ArenaDirector]] complete (event routing exists)
- [ ] TextMeshPro imported (Unity prompts on first use — click "Import TMP Essentials")

---

## Checklist

- [ ] Create Canvas (Screen Space - Overlay)
- [ ] Add alive-count text (top-left)
- [ ] Add possession panel (red tint, title + timer, disabled by default)
- [ ] Add defeat screen (disabled by default)
- [ ] Add results screen (disabled by default)
- [ ] Create `Scripts/UI/UIRouter.cs`
- [ ] Add UIRouter to scene, wire all references
- [ ] Test each screen appears at the right moment

---

## 1. Canvas Setup (20 min)

**Hierarchy → UI → Canvas**

| Setting | Value |
|---|---|
| Render Mode | **Screen Space - Overlay** |
| UI Scale Mode (Canvas Scaler) | Scale With Screen Size |
| Reference Resolution | 1920 × 1080 |

**What Screen Space - Overlay means:** UI draws on top of everything, always visible, ignores cameras. Correct choice here because our camera lives on the Actor prefab and gets destroyed/replaced — world-space UI would vanish with it. See [[Canvas]].

### Alive Count (top-left)

- [ ] Right-click Canvas → UI → Text - TextMeshPro
- [ ] Name: `AliveCountText`
- [ ] Anchor: top-left, Pos (120, -40)
- [ ] Text: `Alive: 8`
- [ ] Font Size: 28

### Possession Panel

- [ ] Right-click Canvas → UI → Panel
- [ ] Name: `PossessionPanel`
- [ ] Color: red, Alpha ~60 (subtle tint, not opaque — you must still see the game)
- [ ] Child text `PossessionTitle`: "YOU ARE POSSESSED", font 48, centered, top third
- [ ] Child text `PossessionTimer`: "90", font 64, centered, middle
- [ ] **Set GameObject inactive** (uncheck the box next to its name)

**Why alpha 60, not 255:** You are still playing while possessed. An opaque overlay blinds you and makes possession unplayable.

### Defeat Screen

- [ ] Right-click Canvas → UI → Panel
- [ ] Name: `DefeatScreen`
- [ ] Color: dark red, Alpha ~230 (nearly opaque — you are out)
- [ ] Child text: "YOU DIED", font 72, centered
- [ ] **Set inactive**

### Results Screen

- [ ] Right-click Canvas → UI → Panel
- [ ] Name: `ResultsScreen`
- [ ] Color: black, Alpha ~230
- [ ] Child text `ResultsText`: "SURVIVORS WIN", font 64, centered
- [ ] **Set inactive**

---

## 2. UIRouter.cs (25 min)

**Location:** `Assets/Scripts/UI/UIRouter.cs`

```csharp
using UnityEngine;
using TMPro;
using Possessed;

public class UIRouter : MonoBehaviour {
    public static UIRouter Instance { get; private set; }

    [Header("HUD")]
    public TextMeshProUGUI aliveCountText;

    [Header("Possession")]
    public GameObject possessionPanel;
    public TextMeshProUGUI possessionTimerText;

    [Header("Screens")]
    public GameObject defeatScreen;
    public GameObject resultsScreen;
    public TextMeshProUGUI resultsText;

    void Awake() {
        Instance = this;
        possessionPanel.SetActive(false);
        defeatScreen.SetActive(false);
        resultsScreen.SetActive(false);
    }

    public void SetAliveCount(int count) {
        aliveCountText.text = $"Alive: {count}";
    }

    public void ShowPossessionUI(bool show) {
        possessionPanel.SetActive(show);
    }

    public void SetPossessionTimer(float seconds) {
        possessionTimerText.text = Mathf.CeilToInt(seconds).ToString();

        // Red flash under 15s (matches TimerWarning audio cue)
        possessionTimerText.color = seconds <= 15f
            ? Color.Lerp(Color.white, Color.red, Mathf.PingPong(Time.time * 3f, 1f))
            : Color.white;
    }

    public void ShowDefeatScreen() {
        possessionPanel.SetActive(false);   // Clear possession UI on death
        defeatScreen.SetActive(true);
    }

    public void ShowResults(Ending ending) {
        resultsScreen.SetActive(true);
        resultsText.text = ending switch {
            Ending.SurvivorsWin => "GHOST DESTROYED\nSURVIVORS WIN",
            Ending.GhostWins    => "THE GHOST SURVIVES",
            Ending.NoWinner     => "NO WINNER",
            _                   => "GAME OVER"
        };
    }
}
```

**What `Mathf.CeilToInt` does:** Rounds up. Timer shows "1" until it truly hits zero, instead of flashing "0" for a full second.

**What `Mathf.PingPong` does:** Bounces 0→1→0 forever. Used here to pulse the timer red as a panic cue.

**What `switch` expression is:** C# 8 shorthand for mapping one value to another. Equivalent to a switch statement that returns a value.

---

## 3. Wire to ArenaDirector (10 min)

In [[ArenaDirector]] `Update()`:

```csharp
void Update() {
    if (game == null) return;

    if (PhotonNetwork.IsMasterClient) {
        game.Tick(Time.deltaTime);
    }

    // HUD refresh
    UIRouter.Instance.SetAliveCount(game.AliveCount());

    // Possession timer (only if local player is the host)
    if (game.hostIdx == localActorIdx) {
        UIRouter.Instance.SetPossessionTimer(game.possessTimer);
    }
}
```

In `HandleGameEvent()`:

```csharp
case GameEvent.TransferBell:
    if (actorIdx == localActorIdx) UIRouter.Instance.ShowPossessionUI(true);
    break;

case GameEvent.Scream:
case GameEvent.HostExecuted:
    if (actorIdx == localActorIdx) UIRouter.Instance.ShowDefeatScreen();
    break;
```

---

## Add to Scene

1. Hierarchy → Create Empty → Name: `UIRouter`
2. Add Component → UI Router
3. Drag every reference from the Canvas into its Inspector slot:
   - Alive Count Text → `AliveCountText`
   - Possession Panel → `PossessionPanel`
   - Possession Timer Text → `PossessionTimer`
   - Defeat Screen → `DefeatScreen`
   - Results Screen → `ResultsScreen`
   - Results Text → `ResultsText`

**Leaving a slot empty causes a NullReferenceException at runtime.** Check every one.

---

## Verification Checklist

- [ ] UIRouter.cs compiles
- [ ] All 6 Inspector references assigned (none say "None")
- [ ] Play mode: alive count shows and decreases on kills
- [ ] Play mode: possession panel appears when you become host
- [ ] Play mode: timer counts down, pulses red under 15s
- [ ] Play mode: defeat screen appears when you die
- [ ] Play mode: results screen appears at game end with correct text

---

## Common Mistakes

### 1. Panels Left Active in Editor
**Symptom:** Defeat screen covers the game from frame 1.
**Fix:** Uncheck the active box on all three panels. `Awake()` also forces them off as a safety net.

### 2. Possession Panel Fully Opaque
**Symptom:** You cannot play while possessed.
**Fix:** Alpha ~60 on the panel Image, not 255.

### 3. Using Legacy Text Instead of TextMeshPro
**Symptom:** Blurry text at high resolution.
**Fix:** Use `Text - TextMeshPro`, field type `TextMeshProUGUI`. See [[TextMeshPro]].

### 4. Updating HUD Only on Events
**Symptom:** Alive count goes stale.
**Fix:** Refresh in `Update()`, as shown above.

---

## Time Breakdown

| Task | Time |
|---|---|
| Canvas + alive count | 8 min |
| Possession panel | 7 min |
| Defeat + results screens | 5 min |
| Write UIRouter.cs | 25 min |
| Wire references | 5 min |
| Test all four states | 10 min |
| **Total** | **60 min** |

**Behind schedule?** Ship the **possession panel only** — it is the one non-negotiable piece, because possession is invisible without it. Cut defeat and results screens (use `Debug.Log` instead). Saves ~20 min.

---

## What You Built

- Alive-count HUD
- Possession panel with pulsing countdown
- Defeat screen
- Results screen for all three endings

**Next:** [[Block 12 - Multiplayer Testing]]

---

## Related Notes

- [[Canvas]]
- [[TextMeshPro]]
- [[UI Panels]]
- [[GameEvent]]
- [[Game Rules]] — the three endings this screen reports
- [[Block 10 - Audio System]]
- [[Block 12 - Multiplayer Testing]]
