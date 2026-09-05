# Block 10 - Audio System

**Time Budget:** 30 minutes (8:00–8:30)
**Goal:** Write AudioRouter.cs — spatial event sounds with wall muffling

---

## What You're Building

3D positional audio feedback. Events (kills, Ghost transfers) play from actor positions. Wall muffling creates information asymmetry: "I heard a bell to my left, but it's muffled → Ghost transferred to someone behind that wall."

---

## Prerequisites

- [ ] [[Block 6 - ArenaDirector]] complete (event routing)
- [ ] Downloaded 4 audio clips (scream, bell, crescendo, shatter)
- [ ] Understanding of [[AudioSource 3D]]
- [ ] Understanding of [[AudioLowPassFilter]]

---

## Checklist

- [ ] Download/create 4 audio clips
- [ ] Create `Scripts/Audio/AudioRouter.cs`
- [ ] Add AudioRouter to scene
- [ ] Test: Kill plays scream from victim position
- [ ] Test: Ghost transfer plays bell from new host position
- [ ] Test: Audio muffles when wall blocks LOS

---

## Audio Clip Sources (10 min)

**Download free horror sounds:**
- [freesound.org](https://freesound.org) — search "scream horror", "bell toll", "crescendo tension", "glass shatter"
- [OpenGameArt.org](https://opengameart.org) — Public domain assets
- Unity Asset Store → "Horror Sound Effects" (many free packs)

**Required clips:**
1. **scream.wav** — short, sharp (played on kill)
2. **bell.wav** — low toll or whisper (played on Ghost transfer)
3. **crescendo.wav** — rising tension (played at possession timer warning, 15s remaining)
4. **shatter.wav** — glass breaking or chime (played when Ghost destroyed)

**Import:**
1. Create folder: Assets/Audio/
2. Drag clips into folder
3. Select each clip → Inspector:
   - Load Type: Decompress On Load (for short clips)
   - Preload Audio Data: ON

---

## Key Responsibilities

### 1. Map Events to Sounds

```csharp
public void PlayEvent(GameEvent evt, int actorIdx) {
    Actor actor = ActorRegistry.Instance.GetActorByIndex(actorIdx);
    if (actor == null) return;
    
    AudioClip clip = null;
    
    switch (evt) {
        case GameEvent.Scream:
            clip = screamClip;
            break;
        case GameEvent.TransferBell:
            clip = bellClip;
            break;
        case GameEvent.TimerWarning:
            clip = crescendoClip;
            break;
        case GameEvent.HostExecuted:
            clip = screamClip;  // Same as kill scream
            break;
        case GameEvent.GhostShattered:
            clip = shatterClip;
            break;
    }
    
    if (clip != null) {
        PlayAtPosition(actor, clip);
    }
}
```

### 2. Play 3D Audio from Actor Position

```csharp
void PlayAtPosition(Actor actor, AudioClip clip) {
    if (actor.audioSource == null) return;
    
    // Apply wall muffling BEFORE playing
    ApplyMuffling(actor);
    
    actor.audioSource.PlayOneShot(clip);
}
```

**What is PlayOneShot:**
- Plays clip once without interrupting currently playing sounds
- Multiple screams can overlap (unlike `Play()` which stops previous sound)
- See [[AudioSource API]]

### 3. Wall Muffling (Core Information Mechanic)

```csharp
void ApplyMuffling(Actor speaker) {
    Actor listener = ActorRegistry.Instance.AllActors.FirstOrDefault(a => a.photonView.IsMine);
    if (listener == null) return;
    
    AudioLowPassFilter filter = speaker.lowPassFilter;
    if (filter == null) return;
    
    // Linecast from listener to speaker
    bool wallBlocks = Physics.Linecast(
        listener.EyePosition,
        speaker.EyePosition,
        wallLayerMask,
        QueryTriggerInteraction.Ignore
    );
    
    // 800 Hz = muffled (like through wall)
    // 22000 Hz = clear (no filtering, full frequency range)
    filter.cutoffFrequency = wallBlocks ? 800f : 22000f;
}
```

**What is AudioLowPassFilter:**
- Cuts frequencies above cutoff Hz
- 22000 Hz = human hearing limit, effectively no filtering
- 800 Hz = heavy muffling (sounds like through a thick wall)
- See [[AudioLowPassFilter]]

**Why this matters:**
- "I heard a bell" → Ghost transferred somewhere
- "I heard a **muffled** bell to my left" → Ghost transferred behind that wall to the left
- **Sound literacy is the skill ceiling** of this game

### 4. Update Muffling Every Frame (Optional)

For dynamic muffling (sound starts clear, becomes muffled as speaker moves behind wall):

```csharp
void Update() {
    Actor listener = ActorRegistry.Instance.AllActors.FirstOrDefault(a => a.photonView.IsMine);
    if (listener == null) return;
    
    foreach (Actor actor in ActorRegistry.Instance.AllActors) {
        if (actor == listener) continue;
        ApplyMuffling(actor);
    }
}
```

**For 11-hour jam:** Skip continuous update. Only muffle at PlayOneShot time. Saves 10 min.

---

## Full Script Structure

```csharp
using UnityEngine;
using Possessed;
using System.Linq;

public class AudioRouter : MonoBehaviour {
    public static AudioRouter Instance { get; private set; }
    
    [Header("Audio Clips")]
    public AudioClip screamClip;
    public AudioClip bellClip;
    public AudioClip crescendoClip;
    public AudioClip shatterClip;
    
    [Header("Layers")]
    public LayerMask wallLayerMask;
    
    void Awake() {
        Instance = this;
    }
    
    public void PlayEvent(GameEvent evt, int actorIdx) { /* ... */ }
    void PlayAtPosition(Actor actor, AudioClip clip) { /* ... */ }
    void ApplyMuffling(Actor speaker) { /* ... */ }
}
```

See **GUIDE-FULL.md §9.1** for complete implementation.

---

## Add to Scene

1. Hierarchy → Create Empty
2. Name: `AudioRouter`
3. Add Component → Audio Router
4. Inspector:
   - Scream Clip: Drag scream.wav
   - Bell Clip: Drag bell.wav
   - Crescendo Clip: Drag crescendo.wav
   - Shatter Clip: Drag shatter.wav
   - Wall Layer Mask: Select "Wall" layer

---

## Connect to ArenaDirector

In `ArenaDirector.HandleGameEvent()`:

```csharp
void HandleGameEvent(GameEvent evt, int actorIdx) {
    // Route to audio
    if (AudioRouter.Instance != null) {
        AudioRouter.Instance.PlayEvent(evt, actorIdx);
    }
    
    // ... UI routing ...
}
```

---

## Verification Checklist

- [ ] AudioRouter.cs compiles
- [ ] 4 audio clips imported to Assets/Audio/
- [ ] AudioRouter in scene, clips assigned
- [ ] Play mode (2 clients): Kill victim → scream plays from victim position
- [ ] Play mode (2 clients): Ghost transfers → bell plays from new host position
- [ ] Play mode: Bell sounds muffled when wall blocks LOS to speaker
- [ ] Play mode: Bell sounds clear when no wall between you and speaker

**Test setup:**
1. Two clients (Editor + WebGL build)
2. Editor: stand behind wall
3. Browser: trigger Ghost transfer (kill someone)
4. Editor: listen for muffled bell
5. Editor: move into open space (no walls)
6. Browser: trigger transfer again
7. Editor: bell should sound clear now

---

## Common Mistakes

### 1. AudioSource Spatial Blend = 0

**Symptom:** All sounds play at same volume regardless of position (sounds like 2D audio)
**Fix:** Actor.audioSource.spatialBlend = 1.0 (set in Actor.prefab). See [[Block 3 - Actor Prefab]].

### 2. Forgetting to Apply Muffling Before PlayOneShot

**Wrong:**
```csharp
actor.audioSource.PlayOneShot(clip);
ApplyMuffling(actor);  // Too late, sound already started
```

**Right:**
```csharp
ApplyMuffling(actor);  // Set filter first
actor.audioSource.PlayOneShot(clip);  // Then play
```

### 3. Using Play() Instead of PlayOneShot()

**Play():** Stops current sound, starts new one
**PlayOneShot():** Overlaps sounds (multiple screams can play simultaneously)

For event sounds, PlayOneShot is correct.

### 4. Not Setting Layer Mask for Linecast

**Symptom:** Linecast hits everything (ground, actors) → always returns "blocked"
**Fix:**
```csharp
Physics.Linecast(a, b, wallLayerMask);  // Only hits Wall layer
```

---

## Time Breakdown

| Task | Time |
|---|---|
| Download/import audio clips | 10 min |
| Write AudioRouter.cs | 15 min |
| Add to scene, connect to ArenaDirector | 5 min |
| Test audio playback | 5 min |
| Test wall muffling | 5 min |
| **Total** | **40 min** |

**Behind schedule?** This is **soft feature**. Cut if 30+ min behind. Silent game still works (visual feedback sufficient). But audio adds immersion and tests well, so worth including if on schedule.

---

## What You Built

Spatial audio system:
- Event-to-sound mapping
- 3D positional audio (sounds come from actor positions)
- Wall muffling (information asymmetry: muffled = behind wall)

**Next:** [[Block 11 - UI System]] — visual feedback (HUD, screens)

---

## Related Notes

- [[AudioSource 3D]]
- [[AudioSource API]]
- [[AudioLowPassFilter]]
- [[Physics API]] (Linecast)
- [[GameEvent]]
- [[Block 6 - ArenaDirector]]
- [[Block 11 - UI System]]
