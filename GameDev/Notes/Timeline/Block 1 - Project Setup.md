# Block 1 - Project Setup

**Time:** 0:00 – 0:30 (30 minutes)  
**Goal:** Create Unity project, import Photon PUN 2, set up folder structure

---

## Checklist

- [ ] Create Unity project (3D URP template)
- [ ] Import Photon PUN 2 from Asset Store
- [ ] Register at photonengine.com and get App ID
- [ ] Setup Photon in Unity (PUN Wizard)
- [ ] Install AI Navigation package
- [ ] Create folder structure (Resources/, Scripts/Core/, Scripts/Game/, etc.)
- [ ] Set Ambient Intensity to 0 (Lighting settings)
- [ ] Add custom layers: Wall, Ground
- [ ] Test WebGL build (Switch Platform + Build once)
- [ ] Initialize git repository (optional but recommended)

---

## Step-by-Step Instructions

### 1. Create Unity Project

**Unity Hub → New Project:**
- Template: **3D URP** (Universal Render Pipeline)
- Project Name: `GreedIncarnate`
- Location: Choose your projects folder

**Why URP?**
- Better post-processing (vignette, film grain for horror aesthetic)
- Better WebGL performance
- Built-in Volume system (easier than Post Processing Stack v2)

**Time:** ~2 minutes (project creation)

---

### 2. Import Photon PUN 2

**Window → Asset Store (or Package Manager → Asset Store tab):**
- Search: "Photon PUN 2 FREE"
- Click Download → Import
- Accept all files

**What is Photon PUN 2?**
Multiplayer networking framework. Handles connecting players to rooms, syncing GameObjects across clients, and Remote Procedure Calls (RPCs).

**Why Photon PUN 2 (not Unity Netcode)?**
- ✅ WebGL support (works in browser)
- ✅ Room codes built-in
- ✅ Works over internet (Photon Cloud relay)
- ❌ Unity Netcode doesn't support WebGL hosting, requires Relay setup

**Time:** ~5 minutes (download + import)

**Reference:** [[PhotonNetwork API]], [[Photon PUN 2 Architecture]]

---

### 3. Register for Photon Account

**Go to:** https://www.photonengine.com/pun

1. Create account (or log in)
2. Dashboard → Create a New App
3. Photon Type: **Pun**
4. Name: `GreedIncarnate` (or anything)
5. Copy **App ID** (long string like "a1b2c3d4-e5f6-7890-g1h2-i3j4k5l6m7n8")

**Time:** ~3 minutes

---

### 4. Setup Photon in Unity

**Window → Photon Unity Networking → PUN Wizard:**
- Click "Setup Project"
- Paste your App ID
- Click "Setup"

**Verify it worked:**
- Check `Assets/Photon/PhotonUnityNetworking/Resources/PhotonServerSettings` exists
- AppIdRealtime field should have your App ID

**Time:** ~1 minute

---

### 5. Install AI Navigation Package

**Window → Package Manager:**
- Dropdown (top-left): Unity Registry
- Search: "AI Navigation"
- Click Install

**What is AI Navigation?**
Provides NavMesh baking system for bot pathfinding. Without this, bots can't navigate around walls.

**Time:** ~2 minutes

**Reference:** [[NavMesh]], [[NavMesh Baking]]

---

### 6. Create Folder Structure

**Project window → Assets → Right-click → Create → Folder:**

```
Assets/
├─ Resources/           ← CRITICAL: Photon prefabs MUST be here
├─ Scripts/
│  ├─ Core/            ← Plain C# (GhostGame, Sight, IVoiceLayer)
│  ├─ Game/            ← MonoBehaviours (Actor, ArenaDirector, etc.)
│  ├─ Multiplayer/
│  ├─ Audio/
│  └─ UI/
├─ Scenes/
├─ Audio/
└─ Materials/
```

**Why Resources/ folder?**
`PhotonNetwork.Instantiate("Actor", ...)` only works if the prefab is in Resources/ folder. Photon limitation.

**Time:** ~2 minutes

**Reference:** [[Resources Folder Requirement]]

---

### 7. Project Settings - Lighting

**Window → Rendering → Lighting:**
- Click **Environment** tab
- **Ambient Intensity Multiplier: 0** (or very low like 0.1)

**Why?**
Horror aesthetic requires pitch-black shadows. Unlit areas should be completely dark. Default ambient light makes everything grey.

**Time:** ~1 minute

---

### 8. Project Settings - Layers

**Edit → Project Settings → Tags and Layers:**

**Layers section → Add two custom layers:**
- **Wall** (e.g., User Layer 8)
- **Ground** (e.g., User Layer 9)

**What are layers used for?**
- **Wall:** Vision linecasts check if Wall layer blocks line-of-sight
- **Ground:** Mouse raycasts hit Ground layer to find where player is aiming

**Time:** ~1 minute

---

### 9. Test WebGL Build Pipeline

**CRITICAL:** Test the build pipeline NOW, not at hour 10.

**File → Build Settings:**
- Platform: **WebGL**
- Click "Switch Platform" (takes ~2 min to reimport assets)
- Click "Build"
- Choose empty folder (e.g., `Builds/WebGL/`)
- Wait for build (~5 min first time)

**After build completes:**
- Open `index.html` in browser
- Confirm empty scene loads (no errors)

**Why test now?**
A broken build pipeline discovered at hour 10:30 = failed submission. Fix it when you have time.

**Time:** ~8 minutes (2 min switch + 5 min build + 1 min test)

---

### 10. Initialize Git (Optional but Recommended)

**Terminal/Command Prompt:**
```bash
cd "path/to/GreedIncarnate"
git init

# Download Unity .gitignore
curl -o .gitignore https://raw.githubusercontent.com/github/gitignore/main/Unity.gitignore

# Initial commit
git add .
git commit -m "Initial commit: URP project + Photon PUN 2 + AI Navigation"
```

**Why?**
- Block 8 (Photon multiplayer) is where things break
- Having a clean commit at hour 6 saves your jam if things go wrong
- You can rollback to working state

**Time:** ~2 minutes

---

## Verification Checklist

Before moving to Block 2, verify:

- [ ] Unity project opens without errors
- [ ] Photon PUN 2 menu appears (Window → Photon Unity Networking)
- [ ] PhotonServerSettings asset exists with your App ID
- [ ] AI Navigation package installed (check Package Manager)
- [ ] Resources/ folder exists in Assets/
- [ ] All script folders created (Core/, Game/, Multiplayer/, Audio/, UI/)
- [ ] Ambient Intensity set to 0
- [ ] Layers added: Wall, Ground
- [ ] WebGL build completed successfully
- [ ] Browser loads index.html without errors

---

## Common Issues

### "Photon PUN 2 not found in Asset Store"
- Use Package Manager instead → Asset Store tab
- Or download from https://www.photonengine.com/pun

### "AI Navigation package not found"
- Older Unity: It's built-in, skip this step
- Newer Unity (2022.2+): It's a separate package

### "WebGL build fails"
- Check Console for errors
- Verify no syntax errors in existing scripts
- Try switching to Standalone first, then WebGL

### "App ID not saving in PhotonServerSettings"
- Manually edit PhotonServerSettings asset
- Set AppIdRealtime field to your App ID

---

## Time Budget

| Task | Time |
|---|---|
| Create project | 2 min |
| Import Photon | 5 min |
| Register account | 3 min |
| Setup Photon | 1 min |
| Install AI Nav | 2 min |
| Create folders | 2 min |
| Lighting settings | 1 min |
| Add layers | 1 min |
| Test WebGL | 8 min |
| Git init | 2 min |
| **Total** | **27 min** |
| **Buffer** | **3 min** |

---

## What You've Built

By end of Block 1, you have:
- Empty Unity project with URP
- Photon PUN 2 ready to connect to cloud
- AI Navigation ready for NavMesh baking
- Folder structure for organizing scripts
- Project settings configured (dark shadows, custom layers)
- Verified WebGL build pipeline works
- Clean git commit (optional)

**No actual gameplay yet** — that starts in Block 2 (scene setup).

---

## Next Steps

**→ [[Block 2 - Scene Setup]]** — Create arena with floor, walls, NavMesh, lighting

---

## Reference Documentation

- [[Photon PUN 2 Architecture]] — How multiplayer works
- [[PhotonNetwork API]] — Global Photon operations
- [[NavMesh]] — AI pathfinding system
- [[Resources Folder Requirement]] — Why prefabs must be here
- [[WebGL Build Settings]] — Build configuration details

---

**Remember:** If you're behind schedule, DON'T skip testing the WebGL build. That's the only hard requirement for submission. Everything else can be cut if needed.
