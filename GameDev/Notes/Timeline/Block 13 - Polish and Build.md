# Block 13 - Polish and Build

**Time Budget:** 60 minutes (10:00–11:00)
**Goal:** Tune feel, build WebGL, upload to itch.io, submit

---

## Hard Rule for This Block

**Stop feature work now.** Every minute here is tuning, building, or uploading. A polished game that never uploads scores zero.

Reserve the **final 25 minutes** for build + upload. Set a timer.

---

## Prerequisites

- [ ] [[Block 12 - Multiplayer Testing]] passed
- [ ] Green commit exists

---

## 1. Tuning Pass (20 min)

Change values only. No new scripts.

### Feel

| Value | Where | Default | Tune if |
|---|---|---|---|
| Move Speed | TopDownController | 5 | Too slow to escape → 6. Too twitchy → 4.5 |
| Rot Speed | TopDownController | 10 | Aiming feels laggy → 14 |
| Possession Timer | GhostGame | 90s | Rounds drag → 70s. Rushed → 110s |
| Survivor Kill Radius | ArenaDirector | 2.0m | Kills whiff constantly → 2.3 |
| Ghost Kill Radius | ArenaDirector | 1.0m | Host cannot ever land a kill → 1.2 |
| Swing Cooldown | ArenaDirector | 3s | Spam-clicking wins → keep 3 |

**Rule:** change one value, play once, keep or revert. Do not stack five changes and guess.

### Visual (URP, 5 min max)

- [ ] Hierarchy → Volume → Global Volume
- [ ] Add Override → Post-processing → **Vignette**, Intensity ~0.35
- [ ] Add Override → **Film Grain**, Intensity ~0.25

Two overrides, 80% of the horror look. Stop there.

### Audio Levels

- [ ] Scream loudest (it is the kill confirm)
- [ ] Bell clearly audible but quieter than scream
- [ ] Crescendo audible but not overpowering

---

## 2. Player Settings (5 min)

**Edit → Project Settings → Player**

- [ ] Company Name: your name / team name
- [ ] Product Name: `POSSESSED`
- [ ] **Resolution and Presentation → WebGL Template: Minimal**
- [ ] **Publishing Settings → Compression Format: Disabled**

**Why Compression Disabled, not Brotli:** Brotli needs server headers itch.io does not always send, producing a blank screen on load. Disabled builds are larger but load reliably. Reliability beats file size on submission day.

---

## 3. WebGL Build (15 min)

- [ ] File → Build Settings
- [ ] Confirm `Arena.unity` is in Scenes In Build (checked)
- [ ] Platform: WebGL
- [ ] Build → `Builds/WebGL/`
- [ ] Wait (5–12 min)

**While it builds:** write your itch.io description (template below). Do not sit idle.

### Test the Build

- [ ] Open `Builds/WebGL/index.html` in a browser
- [ ] Loads without a blank screen
- [ ] Connects to Photon
- [ ] Actor spawns, WASD works
- [ ] Open a second tab → two players see each other
- [ ] Land one kill

If the browser blocks audio until you click — that is normal WebGL autoplay policy, not a bug.

---

## 4. Upload to itch.io (10 min)

- [ ] Zip the **contents** of `Builds/WebGL/` (so `index.html` sits at the zip root, not inside a nested folder)
- [ ] itch.io → Dashboard → Create New Project
- [ ] Title: `POSSESSED`
- [ ] Kind of project: **HTML**
- [ ] Upload the zip → check **"This file will be played in the browser"**
- [ ] Embed options: Click to launch in fullscreen
- [ ] Viewport: 1280 × 720
- [ ] Upload 2–3 screenshots (Editor: Ctrl+Shift+S or a screen capture of the build)
- [ ] Visibility: **Public**
- [ ] Save → open the public page → confirm it loads and plays

**A common failure:** zipping the folder itself instead of its contents, so itch.io cannot find `index.html`. Verify the zip root.

### Description Template

```
POSSESSED — top-down social deception horror

One of you carries a Ghost. You cannot see it.
Every kill relocates it — randomly, never to the killer.

Kill the current host and the Ghost is destroyed: survivors win.
Kill anyone else and it simply moves somewhere worse.
If only two remain and the Ghost still lives, the Ghost wins.

If you are possessed, you have 90 seconds.
Kill someone, or the Ghost kills you.

CONTROLS
WASD — move
Mouse — aim
Left click — strike (2m reach, 3s cooldown; only 1m while possessed)

VISION
Bright cone — you can see them, and you can kill them
Dim ring — something is near, but you cannot strike it
Darkness — nothing

Listen. A bell means the Ghost moved. Muffled means it moved behind a wall.

Multiplayer — open in two tabs or send a friend the link.
```

---

## 5. Submit to the Jam (5 min)

- [ ] Copy the itch.io page URL
- [ ] Paste into the jam submission form
- [ ] Confirm the submission is saved
- [ ] Final commit:

```bash
git add .
git commit -m "Jam submission: POSSESSED WebGL build"
```

---

## Time Breakdown

| Task | Time | Hard deadline |
|---|---|---|
| Tuning | 20 min | must end by 10:20 |
| Player Settings | 5 min | by 10:25 |
| WebGL build | 15 min | by 10:40 |
| Test build | 5 min | by 10:45 |
| itch.io upload | 10 min | by 10:55 |
| Jam submit | 5 min | by 11:00 |

**At 10:35, stop everything and build.** An unbuilt masterpiece scores nothing.

---

## If Things Break Late

| Situation | Action |
|---|---|
| Build fails with script errors | `git checkout .` back to the green commit, rebuild |
| Blank screen in browser | Compression Format → Disabled, rebuild |
| Photon does not connect in build | Confirm App ID in PhotonServerSettings (Resources folder ships with the build) |
| Out of time, build broken | Upload the **last working build** you tested, even if unpolished |

---

## Final Submission Checklist

- [ ] WebGL build loads in a browser
- [ ] Two clients can see each other
- [ ] Kills work and sync
- [ ] Possession UI appears
- [ ] itch.io page is Public
- [ ] Jam submission saved
- [ ] Controls listed in the description

---

## Related Notes

- [[WebGL Build Settings]]
- [[itch.io Upload]]
- [[Game Rules]]
- [[Block 12 - Multiplayer Testing]]
- [[00 - Index]]
