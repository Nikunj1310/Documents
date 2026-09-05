# Photon PUN 2 Architecture

**Category:** Multiplayer Concept
**Package:** PUN 2 FREE (Photon Unity Networking 2)

---

## What Photon PUN 2 Is

A multiplayer networking framework for Unity. It handles four things:

1. **Connecting players** into a shared session (a Room)
2. **Syncing GameObjects** across every client in that Room
3. **Remote Procedure Calls** — invoking a method on other clients' machines
4. **Ownership** — tracking which client controls which GameObject

You do not run a server. Photon Cloud relays messages between clients.

---

## Why PUN 2 for POSSESSED

| | PUN 2 | Unity Netcode (NGO) |
|---|---|---|
| WebGL builds | ✅ WebSocket fallback works | ❌ No UDP socket in browser |
| Room codes | ✅ Built in | ❌ You build the lobby UI |
| Over the internet | ✅ Photon Cloud relay | ❌ Needs Unity Relay setup (~2h) |
| Concurrent users (free) | 20 CCU | Unlimited |

The jam ships to **itch.io as WebGL**. That single requirement eliminates NGO. The 20 CCU cap is irrelevant for a jam.

---

## Topology

**Client-server relay, not peer-to-peer.** Every message goes client → Photon Cloud → other clients.

**Photon is not authoritative by default.** Photon relays whatever a client sends. If you want validation, you write it. In POSSESSED, the [[Master Client]] performs that validation — see [[Kill Validation Pattern]].

---

## The Pieces

| Piece | Kind | Role |
|---|---|---|
| [[PhotonNetwork API]] | static class | Connect, join rooms, spawn, destroy |
| [[PhotonView]] | Component | Marks a GameObject as networked, holds ViewID + owner |
| [[PhotonTransformView]] | Component | Syncs position/rotation automatically |
| [[RPC]] | method attribute | Calls a method on other clients |
| [[Master Client]] | role | The one client that validates and decides |
| [[Room and Lobby]] | server-side concept | Where players gather and play |
| [[Ownership]] | per-object state | Who is allowed to drive this object |

---

## Connection Flow

```
ConnectUsingSettings()
      ↓ OnConnectedToMaster()
JoinLobby()
      ↓ OnJoinedLobby()
JoinOrCreateRoom("Arena", opts, TypedLobby.Default)
      ↓ OnJoinedRoom()
PhotonNetwork.Instantiate("Actor", spawnPos, identity)
      ↓
Master Client only: new GhostGame(); game.Begin(playerCount)
```

Implemented in [[ArenaDirector]]. Full callback list in [[Room and Lobby]].

---

## What Syncs and What Does Not

**Synced across the network:**
- Actor position and rotation — via [[PhotonTransformView]]
- Kill claims and kill results — via [[RPC]]
- Actor spawn and destroy — via `PhotonNetwork.Instantiate` / `PhotonNetwork.Destroy`

**Never synced (deliberately):**
- `GhostGame.hostIdx` — the possessed player's identity. Only Master Client holds it; the possessed player learns via a targeted RPC. This is the entire deception layer.
- Camera and lights — each client enables only their own. See [[Local-Only Rendering]].
- Vision tiers — every client recomputes locally from synced positions. [[Sight]] is deterministic, so the result matches everywhere.

**Why recompute vision instead of syncing it?** 8 actors × 8 observers × 60fps of tier data is network spam. The math is identical on every client given the same positions, so there is nothing to send.

---

## Ownership Model in POSSESSED

Ownership is assigned once, at spawn, and **never transferred**.

Each client calls `PhotonNetwork.Instantiate("Actor", ...)` for themselves, so each client owns exactly one Actor for the whole match. `photonView.IsMine` is true for that one Actor and false for every other.

The Ghost is a **virtual tag** — an `int hostIdx` inside [[GhostGame]]. When it transfers, no GameObject changes hands. Nothing calls `TransferOwnership`. See [[Ownership]] for why this matters and when transfer would be used instead.

---

## Authority Model

```
Client                    Master Client
  │
  │ "I killed actor 3"  (RPC → MasterClient)
  ├───────────────────────────►│
  │                            │ re-check range
  │                            │ re-check vision tier
  │                            │ re-check line of sight
  │                            │ re-check cooldown
  │                            │ GhostGame.TryKill(...)
  │  "actor 3 died"  (RPC → All)
  │◄───────────────────────────┤
  │                            │
  ▼ destroy actor 3            ▼ destroy actor 3
```

**Clients never self-declare a kill.** A client asks; the Master Client decides. Details in [[Kill Validation Pattern]].

---

## Hard Requirements

- Networked prefabs **must** live in `Assets/Resources/` — [[Resources Folder Requirement]]
- Networked prefabs **must** have a [[PhotonView]] component
- Use `PhotonNetwork.Instantiate` / `PhotonNetwork.Destroy`, never plain `Instantiate` / `Destroy`
- RPC methods **must** carry `[PunRPC]`
- RPC parameters must be serializable — int, float, bool, string, Vector3, Quaternion. Never a GameObject or Component; pass a ViewID or actor index instead.

---

## Related Notes

- [[PhotonNetwork API]] — the static entry point
- [[PhotonView]] — the core networked component
- [[PhotonTransformView]] — position/rotation sync
- [[RPC]] — remote method calls
- [[Master Client]] — the validating client
- [[Room and Lobby]] — connection lifecycle
- [[Ownership]] — IsMine and why we never transfer
- [[Kill Validation Pattern]] — the claim/validate/broadcast loop
- [[Local-Only Rendering]] — why camera and lights are not synced
- [[ArenaDirector]] — where the connection flow lives
- [[Block 1 - Project Setup]] — installing and configuring PUN 2
