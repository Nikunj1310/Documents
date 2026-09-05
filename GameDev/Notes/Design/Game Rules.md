# Game Rules

**What you're building:** A top-down 3D social deception horror game where one player is secretly possessed by a Ghost.

---

## Core Concept

- **3-10 players** in a match
- **One Ghost exists** — it's a virtual tag (variable `int hostIdx`), NOT a physical character
- **Possessed player knows they're possessed** — sees "YOU ARE POSSESSED" UI + countdown timer
- **Other players don't know** — must deduce from behavior and audio cues
- **Every kill relocates the Ghost randomly**
- **Goal:** Kill the currently possessed player to destroy the Ghost, OR survive until you're one of the last 2 standing

---

## The Ghost (Virtual Entity)

The Ghost is NOT a separate character you control. It's just a **tag** on one player.

**What happens when you're possessed:**
1. UI appears: "YOU ARE POSSESSED"
2. 90-second countdown timer starts (configurable)
3. Warning sound at 15 seconds remaining
4. If timer expires: You die, Ghost transfers to someone else randomly

**What other players see:**
- Nothing different about you visually
- They hear a **bell sound from your position** when Ghost transfers to you
- They must deduce who's possessed from behavior

---

## Kill Mechanics

### Range
- **Survivor kill range:** 2.0 meters
- **Ghost kill range:** 1.0 meters (possessed player must get closer)
- **Cooldown:** 3 seconds between attacks

### How to Kill
1. Left-click while targeting someone
2. Target must be in **Full vision tier** (inside your cone, within range, clear line-of-sight)
3. Within kill range (2m for normal, 1m if you're Ghost)
4. Client sends kill request to Master Client
5. Master Client validates (range, vision, cooldown)
6. If valid: Kill applies, broadcasts to all clients

### What Happens on Kill

| You Kill | Result |
|---|---|
| **Non-possessed player** | Victim dies (scream) → Ghost transfers **randomly** to another living player (excludes you, the killer) → New host sees possession UI + timer starts → Bell sound plays from new host's position |
| **Possessed player** | Victim dies → **Ghost is DESTROYED** (shatter sound) → All alive players WIN |

**Key point:** Killing a non-possessed player makes things WORSE for everyone — Ghost relocates randomly, could be closer to you now.

---

## Possession Timer

- **Default duration:** 90 seconds (host-configurable)
- **Starts:** The moment you become possessed
- **Warning:** Rising crescendo sound at 15 seconds remaining
- **If expires:** You are killed by the Ghost itself → Scream from your position → Ghost transfers randomly to another living player

**Important:** Letting the timer expire does NOT destroy the Ghost. It just kills you and moves the Ghost to someone else. You lose either way if you're possessed.

---

## Win Conditions

| Situation | Outcome | Winners |
|---|---|---|
| Possessed player killed by another player | **Survivors Win** | All currently alive players split prize pool |
| A kill leaves exactly **2 alive** (one is Ghost) | **Ghost Wins** | The player who made that final kill gets 100% of prize pool |
| Timer expires leaving exactly **2 alive** | **No Winner** | Nobody gets paid (punishes passive play) |

**Dead players always lose** — even if your team wins later, you don't get anything if you died earlier.

### Example Scenarios

**Scenario 1: Good ending**
- 5 players alive
- Player A kills Player B (who is possessed)
- Ghost destroyed → Players A, C, D, E win (split prize)
- Player B loses (was possessed and died)

**Scenario 2: Bad ending**
- 3 players alive: A, B (possessed), C
- Player A kills Player C (innocent)
- Now 2 alive: A and B (Ghost still alive)
- **Ghost wins** → Player A gets prize pool (even though B is the Ghost)
- Why? Because A's kill reduced the count to 2, satisfying Ghost's win condition

**Scenario 3: Passive play**
- 3 players alive: A, B (possessed), C
- B's timer expires → B dies
- Now 2 alive: A and C (Ghost randomly transferred to either A or C)
- **No winner** → Nobody gets paid (game punishes letting timer expire at critical moment)

---

## Vision Tiers (Targeting Rules)

You can only kill targets in **Full** vision tier. The three tiers:

| Tier | Condition | What You See | Can Kill? |
|---|---|---|---|
| **Full** | Inside focus cone + within range + clear line-of-sight | Full colored model + nametag floating above | ✅ YES |
| **Silhouette** | Within 4.5m sphere (any direction) + clear LOS | Black silhouette, no nametag | ❌ NO |
| **Hidden** | Outside range OR wall blocking | Completely invisible | ❌ NO |

### Focus Cone (Where You're Looking)

- **Moving:** 40° half-angle (80° total), 14m range
- **Standing still:** Opens to 62° half-angle (124° total), 16m range after ~0.5 seconds
- **Closes fast** when you start moving (lerp speed 12/s)
- **Opens slowly** when you stop (lerp speed 4/s)

**Why peeking is risky:** It takes time to open the cone. If you stop to peek around a corner, you're vulnerable for half a second before your vision expands.

### Peripheral Vision (360° Around You)

- **4.5m radius sphere**
- **Any direction** (doesn't matter where you're looking)
- Shows **Silhouette tier** only (can't kill, but warns you someone is close)

**Why it matters:** You hear footsteps, see a black silhouette nearby, but can't kill them yet. Turn to face them → they enter your cone → Full tier → can kill.

### Wall Occlusion

Even if someone is in your cone + range, **walls block vision**.

**How it works:**
1. Sight.Evaluate() returns Full or Silhouette
2. VisionSystem does Physics.Linecast(your eyes, target eyes)
3. If linecast hits Wall layer → downgrade to Hidden
4. Target becomes invisible, can't kill them

**Audio also muffles:** AudioLowPassFilter drops to 800Hz when wall blocks LOS (sounds like hearing through a wall).

---

## Audio Cues (How You Track the Ghost)

Sound is your primary information source. Listen carefully:

| Event | Sound | Source | What It Means |
|---|---|---|---|
| **Kill** | Scream | Victim's position | Someone just died there |
| **Ghost Transfer** | Bell toll | New host's position | Ghost just moved to someone at that location |
| **Timer Warning** | Rising crescendo | Host's position | Possessed player has 15s left |
| **Timer Expiry** | Scream | Host's position | Possessed player's timer expired, they died |
| **Ghost Destroyed** | Shatter chime | All clients | Someone killed the Ghost, survivors win |

**Key strategy:** When you hear a bell, note the direction. The Ghost is now at that location. Track them before they relocate again.

---

## Strategy Tips

### If You're NOT Possessed:
1. **Don't kill randomly** — every innocent kill relocates Ghost randomly (could be worse)
2. **Track bell sounds** — note where Ghost transfers, hunt that location
3. **Stay mobile** — standing still opens your cone but makes you predictable
4. **Use walls** — break line-of-sight to escape
5. **Listen for crescendo** — if you hear timer warning nearby, the possessed player is close

### If You're Possessed:
1. **You WILL die** — either kill someone (Ghost moves, you survive) or timer expires (you die, Ghost moves)
2. **Get close to someone** — your kill range is only 1m (half of normal)
3. **Use confusion** — bells don't reveal your name, blend in with others
4. **Timer pressure** — you have 90s, use it wisely
5. **Sacrifice is NOT heroic** — letting timer expire doesn't destroy Ghost, just kills you and relocates it

### Advanced: The 3-Player Dilemma
- 3 alive: You, Player B, Player C
- You don't know who's possessed
- If you kill innocent → 2 left, Ghost wins, you just handed victory to Ghost
- If you kill Ghost → Ghost destroyed, you + remaining player win
- **Hesitation is death** — but so is guessing wrong

---

## Next Steps

- Read [[Architecture Decisions]] to understand how code is structured
- Start [[Block 1 - Project Setup]] to begin building
- Reference [[Vision Tiers]] for detailed cone math
- Check [[Kill Validation Pattern]] for multiplayer implementation

---

**Remember:** The Ghost is a virtual tag tracked by Master Client in `GhostGame.hostIdx`. It's NOT a physical entity. Each player controls their own Actor the entire game. The Ghost tag just marks "who is currently possessed."
