# Block 12 - Multiplayer Testing

**Time Budget:** 30 minutes (9:30–10:00)
**Goal:** Verify every networked path with 2+ clients before polish

---

## What You're Doing

Hunting the bugs that only exist with two clients. Everything up to now was testable solo. Ownership, RPCs, and sync bugs surface only here.

---

## Prerequisites

- [ ] Blocks 1–11 complete
- [ ] WebGL build pipeline verified in [[Block 1 - Project Setup]]

---

## How to Run Two Clients

**Option A — Editor + WebGL build (recommended)**
1. File → Build Settings → Build → `Builds/WebGL/`
2. Open `Builds/WebGL/index.html` in a browser
3. Press Play in the Editor
4. Both connect to Photon room `"Arena"`

**Option B — Two browser tabs**
Open `index.html` twice. Works, but you cannot inspect the Console or set breakpoints.

**Option C — ParrelSync** (Editor clone package)
Best debugging, but installation costs ~10 min. Skip during a jam unless already installed.

---

## Test Matrix

Work down this list in order. Each row builds on the one above.

### Connection & Spawn
- [ ] Both clients log "Connected to Photon Cloud"
- [ ] Both clients log "Joined room"
- [ ] Two actors exist in each client's Hierarchy
- [ ] `PhotonNetwork.CurrentRoom.PlayerCount == 2` on both

### Ownership
- [ ] WASD moves **only your own** actor on each client
- [ ] Your actor's camera is active; the other actor's camera is not
- [ ] Your actor's FocusLight/PeriphLight are on; the other's are off
- [ ] The other actor wanders on its own ([[BotBrain]] running because `!IsMine`)

### Transform Sync
- [ ] Moving on client A visibly moves that actor on client B
- [ ] Motion is smooth, not teleporting (Interpolate is on in [[PhotonTransformView]])
- [ ] Rotation syncs (actor faces the direction its owner is aiming)

### Vision Tiers
- [ ] Actor behind a wall is fully invisible
- [ ] Actor within 4.5m off to the side shows as a black silhouette with no name
- [ ] Actor inside the front cone shows full color plus a name tag
- [ ] Standing still visibly widens the spotlight over ~0.5s

### Kill Validation
- [ ] Left-click on a `Full`-tier actor within 2m kills it
- [ ] Victim disappears on **both** clients (not just the killer's)
- [ ] Clicking a `Silhouette`-tier actor does **nothing**
- [ ] Clicking a `Hidden`-tier actor does **nothing**
- [ ] Clicking a target 5m away does **nothing** (out of range)
- [ ] Rapid clicking is rate-limited by the 3s cooldown

### Ghost Mechanics
- [ ] One client sees "YOU ARE POSSESSED" at match start
- [ ] Its timer counts down from 90
- [ ] Killing a **non-possessed** actor moves the Ghost — a bell plays and a different client now shows the possession panel
- [ ] The killer never becomes the new host
- [ ] Killing the **possessed** actor ends the match with "GHOST DESTROYED / SURVIVORS WIN"
- [ ] Letting the timer expire kills the host and relocates the Ghost

### Endings
- [ ] Killing the host → `SurvivorsWin` on all clients
- [ ] Reducing to 2 alive by killing an innocent → `GhostWins`
- [ ] Timer expiry that leaves 2 alive → `NoWinner`

### Audio
- [ ] Kill scream plays from the victim's position (directional)
- [ ] Transfer bell plays from the new host's position
- [ ] Sounds are muffled when a wall sits between you and the source

---

## Bug Triage Table

| Symptom | Likely Cause | Fix |
|---|---|---|
| Actor does not spawn | Prefab outside `Resources/` | [[Resources Folder Requirement]] |
| Both actors respond to your WASD | Missing `IsMine` guard | [[Ownership]] |
| Remote actor never moves | `PhotonTransformView` not in Observables | [[PhotonTransformView]] |
| Remote actor teleports in steps | Interpolate disabled | [[PhotonTransformView]] |
| Kill does nothing, no error | RPC name typo, or `[PunRPC]` missing | [[RPC]] |
| Kill applies only on killer's screen | Broadcast RPC not sent to `RpcTarget.All` | [[Kill Validation Pattern]] |
| Everything reads `Hidden` | Passing `transform.position` instead of `EyePosition` | [[Vision Tiers]] |
| Actor jitters violently | NavMeshAgent + CharacterController both on | [[NavMeshAgent vs CharacterController]] |
| Nobody is ever possessed | `game.Begin()` not called, or called on a non-Master client | [[Master Client]] |
| Two clients both act as Master | Both created their own `GhostGame` | [[Master Client]] |
| Bots frozen, NavMesh warning in Console | NavMesh not baked, or spawn Y too high | [[NavMesh Baking]] |

---

## Commit When Green

The moment the Kill Validation and Ghost Mechanics sections pass:

```bash
git add .
git commit -m "Working multiplayer: kills, ghost transfer, endings verified"
```

This is your fallback. [[Block 13 - Polish and Build]] changes tuning values and can break things — a green commit means you can always ship *something*.

---

## Time Breakdown

| Task | Time |
|---|---|
| WebGL build | 8 min |
| Connection + ownership + sync | 7 min |
| Vision + kill validation | 8 min |
| Ghost transfer + endings | 5 min |
| Commit | 2 min |
| **Total** | **30 min** |

**Behind schedule?** Test in this priority order: Connection → Ownership → Kill Validation → Ghost Transfer. Skip Audio and the ending-matrix rows. An untested ending is survivable; an untested kill path is not.

---

## Related Notes

- [[Ownership]]
- [[PhotonTransformView]]
- [[RPC]]
- [[Kill Validation Pattern]]
- [[Master Client]]
- [[Vision Tiers]]
- [[Game Rules]]
- [[Block 11 - UI System]]
- [[Block 13 - Polish and Build]]
