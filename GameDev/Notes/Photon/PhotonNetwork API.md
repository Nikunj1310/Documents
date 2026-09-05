# PhotonNetwork API

## Overview

**PhotonNetwork** is a static class providing global multiplayer operations in [[Photon PUN 2]]. It handles connection, room management, and networked object spawning.

**Think of it as:** The control center for all multiplayer functionality. You call `PhotonNetwork.Something()` to connect, join rooms, spawn objects, etc.

## Key Concepts

### Client-Server Topology
- Not peer-to-peer
- **Photon Cloud** relays messages between clients
- No dedicated server (players connect directly to Photon's servers)

### Master Client
- First player who joined the room
- Makes authoritative decisions (validate kills, spawn enemies)
- **Not a dedicated server** — just another player with extra responsibility
- Migrates automatically if Master Client disconnects

## Connection Flow

```
Start Game
    ↓
PhotonNetwork.ConnectUsingSettings()
    ↓
OnConnectedToMaster() callback
    ↓
PhotonNetwork.JoinLobby()
    ↓
OnJoinedLobby() callback
    ↓
PhotonNetwork.JoinOrCreateRoom(roomName, options, lobby)
    ↓
OnJoinedRoom() callback
    ↓
PhotonNetwork.Instantiate("Actor", pos, rot)
    ↓
Play Game
    ↓
PhotonNetwork.LeaveRoom()
    ↓
OnLeftRoom() callback
```

## Connection Methods

### Connect to Photon Cloud
```csharp
// Connect using settings from PhotonServerSettings asset
PhotonNetwork.ConnectUsingSettings();

// Connect with custom settings
PhotonNetwork.ConnectToMaster(ip, port, appId);
```

**When to use:** At game start, before joining any room.

### Join Lobby
```csharp
// Join default lobby
PhotonNetwork.JoinLobby();

// Join specific lobby
PhotonNetwork.JoinLobby(TypedLobby.Default);
```

**What is Lobby:** Waiting area before joining a room. Can list available rooms.

### Join or Create Room
```csharp
// Join room "Arena", create if doesn't exist
RoomOptions opts = new RoomOptions { MaxPlayers = 8 };
PhotonNetwork.JoinOrCreateRoom("Arena", opts, TypedLobby.Default);

// Join random room
PhotonNetwork.JoinRandomRoom();

// Create room with specific name
PhotonNetwork.CreateRoom("MyRoom", opts);

// Join room by name
PhotonNetwork.JoinRoom("Arena");
```

**Room:** The actual game session. All players in same room see each other.

### Leave Room
```csharp
// Leave current room, return to lobby
PhotonNetwork.LeaveRoom();

// Disconnect completely
PhotonNetwork.Disconnect();
```

## Room State Properties

```csharp
// Are we in a room?
bool inRoom = PhotonNetwork.InRoom;

// Are we connected to Photon Cloud?
bool connected = PhotonNetwork.IsConnected;

// Are we in a lobby?
bool inLobby = PhotonNetwork.InLobby;

// Are we the Master Client?
bool isMaster = PhotonNetwork.IsMasterClient;

// Current room
Room currentRoom = PhotonNetwork.CurrentRoom;

// Local player
Player localPlayer = PhotonNetwork.LocalPlayer;

// All players in room
Player[] players = PhotonNetwork.PlayerList;

// Number of players
int count = PhotonNetwork.CurrentRoom.PlayerCount;
```

## Network Object Methods

### Instantiate (Spawn Networked Object)
```csharp
// Spawn Actor prefab at position/rotation
GameObject obj = PhotonNetwork.Instantiate("Actor", position, rotation);

// With group parameter (advanced)
GameObject obj = PhotonNetwork.Instantiate("Actor", position, rotation, 0);
```

**CRITICAL:**
- Prefab **must be in Resources/ folder** (or subfolder)
- Prefab **must have [[PhotonView]] component**
- String name must match prefab name exactly (case-sensitive)
- Spawns on **all clients** in the room
- Caller becomes **owner** (`photonView.IsMine == true`)

### Destroy (Remove Networked Object)
```csharp
// Destroy networked GameObject
PhotonNetwork.Destroy(gameObject);

// Destroy by PhotonView
PhotonNetwork.Destroy(photonView);
```

**CRITICAL:**
- Use `PhotonNetwork.Destroy`, **NOT** `GameObject.Destroy`
- Only **owner** or **Master Client** can destroy
- Destroys on all clients automatically

## Master Client Authority

### Check if Master Client
```csharp
if (PhotonNetwork.IsMasterClient) {
    // Only Master Client executes this
    game.Tick(Time.deltaTime);
    ValidateKill(killerIdx, victimIdx);
}
```

### Master Client in POSSESSED
- Owns `GhostGame` instance (tracks possession state)
- Validates kill requests (range, vision, cooldown)
- Broadcasts kill results to all clients
- Picks Ghost transfer targets randomly

**Why Master Client:**
- Someone must make authoritative decisions
- Prevents cheating (client can't self-declare kills)
- Single source of truth for game state

## Time Synchronization

```csharp
// Server time (synced across all clients)
double serverTime = PhotonNetwork.Time;

// Time since level loaded (local)
float localTime = Time.time;

// Ping to server (milliseconds)
int ping = PhotonNetwork.GetPing();
```

**Use case:** Timestamp events for interpolation, lag compensation.

## Network Statistics

```csharp
// Current region
string region = PhotonNetwork.CloudRegion;

// Send rate (packets per second)
int sendRate = PhotonNetwork.SendRate;

// Serialization rate (state updates per second)
int serializationRate = PhotonNetwork.SerializationRate;

// Network statistics
LoadBalancingClient.LoadBalancingPeer.BytesIn
LoadBalancingClient.LoadBalancingPeer.BytesOut
```

## Room Options

```csharp
RoomOptions opts = new RoomOptions {
    MaxPlayers = 8,               // Max players allowed
    IsVisible = true,             // Show in lobby room list
    IsOpen = true,                // Allow new players to join
    PublishUserId = true,         // Share player IDs
    CleanupCacheOnLeave = true,   // Remove player data on disconnect
    
    // Custom properties (searchable)
    CustomRoomProperties = new Hashtable {
        { "map", "Arena" },
        { "mode", "Hunt" }
    },
    CustomRomPropertiesForLobby = new string[] { "map", "mode" }
};

PhotonNetwork.JoinOrCreateRoom("Arena", opts, TypedLobby.Default);
```

## Player Properties

```csharp
// Set custom properties on local player
Hashtable props = new Hashtable {
    { "team", "Red" },
    { "score", 0 }
};
PhotonNetwork.LocalPlayer.SetCustomProperties(props);

// Read properties from any player
object team = player.CustomProperties["team"];
```

## Callbacks (MonoBehaviourPunCallbacks)

To receive Photon events, inherit from `MonoBehaviourPunCallbacks`:

```csharp
using Photon.Pun;
using Photon.Realtime;

public class ArenaDirector : MonoBehaviourPunCallbacks {
    
    public override void OnConnectedToMaster() {
        Debug.Log("Connected to Photon Cloud");
        PhotonNetwork.JoinLobby();
    }
    
    public override void OnJoinedLobby() {
        Debug.Log("Joined lobby, can now join/create rooms");
        RoomOptions opts = new RoomOptions { MaxPlayers = 8 };
        PhotonNetwork.JoinOrCreateRoom("Arena", opts, TypedLobby.Default);
    }
    
    public override void OnJoinedRoom() {
        Debug.Log("Joined room, spawn player now");
        Vector3 spawnPos = new Vector3(Random.Range(-12f, 12f), 0.5f, Random.Range(-12f, 12f));
        PhotonNetwork.Instantiate("Actor", spawnPos, Quaternion.identity);
    }
    
    public override void OnPlayerEnteredRoom(Player newPlayer) {
        Debug.Log($"Player {newPlayer.NickName} joined");
    }
    
    public override void OnPlayerLeftRoom(Player otherPlayer) {
        Debug.Log($"Player {otherPlayer.NickName} left");
    }
    
    public override void OnMasterClientSwitched(Player newMasterClient) {
        Debug.Log($"New Master Client: {newMasterClient.NickName}");
        if (PhotonNetwork.IsMasterClient) {
            // I'm now Master Client, take over authority
        }
    }
}
```

### Common Callbacks

| Callback | When | Use |
|---|---|---|
| `OnConnectedToMaster()` | Connected to Photon Cloud | Join lobby |
| `OnJoinedLobby()` | Entered lobby | Join/create room |
| `OnJoinedRoom()` | Entered room | Spawn player |
| `OnLeftRoom()` | Left room | Cleanup |
| `OnPlayerEnteredRoom(Player)` | Another player joined | Update player list |
| `OnPlayerLeftRoom(Player)` | Another player left | Remove their objects |
| `OnMasterClientSwitched(Player)` | Master Client changed | Take over authority if you're new Master |

## POSSESSED Implementation

### ArenaDirector Connection Flow

```csharp
public class ArenaDirector : MonoBehaviourPunCallbacks {
    GhostGame game;
    Actor localActor;
    
    void Start() {
        PhotonNetwork.ConnectUsingSettings();
    }
    
    public override void OnConnectedToMaster() {
        PhotonNetwork.JoinLobby();
    }
    
    public override void OnJoinedLobby() {
        RoomOptions opts = new RoomOptions { MaxPlayers = 8 };
        PhotonNetwork.JoinOrCreateRoom("Arena", opts, TypedLobby.Default);
    }
    
    public override void OnJoinedRoom() {
        // Spawn local actor
        Vector3 spawnPos = new Vector3(Random.Range(-12f, 12f), 0.5f, Random.Range(-12f, 12f));
        GameObject actorObj = PhotonNetwork.Instantiate("Actor", spawnPos, Quaternion.identity);
        localActor = actorObj.GetComponent<Actor>();
        localActor.actorIdx = PhotonNetwork.LocalPlayer.ActorNumber - 1;
        
        // Master Client initializes game
        if (PhotonNetwork.IsMasterClient) {
            game = new GhostGame();
            game.OnEvent += OnGameEvent;
            game.Begin(PhotonNetwork.PlayerList.Length);
        }
    }
    
    void Update() {
        // Only Master Client runs game logic
        if (game != null && PhotonNetwork.IsMasterClient) {
            game.Tick(Time.deltaTime);
        }
    }
}
```

## Common Mistakes

### 1. Forgetting Resources/ Folder
```csharp
// FAILS: Prefab not in Resources/
PhotonNetwork.Instantiate("Actor", pos, rot);

// Solution: Move Actor.prefab to Assets/Resources/Actor.prefab
```

See [[Common-Mistake-Prefabs-Not-In-Resources]]

### 2. Using GameObject.Destroy Instead of PhotonNetwork.Destroy
```csharp
// WRONG: Only destroys locally
Destroy(gameObject);

// RIGHT: Destroys on all clients
PhotonNetwork.Destroy(gameObject);
```

### 3. Not Checking IsMasterClient for Authority
```csharp
// WRONG: All clients pick different Ghost hosts (desync)
void OnKill() {
    hostIdx = Random.Range(0, playerCount);
}

// RIGHT: Only Master Client picks
void OnKill() {
    if (PhotonNetwork.IsMasterClient) {
        hostIdx = Random.Range(0, playerCount);
    }
}
```

### 4. Connecting Multiple Times
```csharp
// WRONG: Connects every frame
void Update() {
    PhotonNetwork.ConnectUsingSettings();
}

// RIGHT: Connect once at start
void Start() {
    if (!PhotonNetwork.IsConnected) {
        PhotonNetwork.ConnectUsingSettings();
    }
}
```

## Performance Tips

1. **Reduce SerializationRate** for slower-paced games (default: 10/sec)
2. **Use RPCs sparingly** — Not every frame (use PhotonTransformView for continuous sync)
3. **Batch RPCs** — Send one RPC with array, not multiple RPCs
4. **Check IsConnected** before operations

## Related Notes

- [[Photon PUN 2]] — Overview
- [[PhotonView]] — Component for networked objects
- [[PhotonTransformView]] — Transform synchronization
- [[RPC]] — Remote procedure calls
- [[Master Client]] — Authority pattern
- [[Room and Lobby]] — Room system details
- [[ArenaDirector]] — POSSESSED implementation

## Quick Reference

```csharp
// Connection
PhotonNetwork.ConnectUsingSettings();
PhotonNetwork.JoinLobby();
PhotonNetwork.JoinOrCreateRoom("Arena", new RoomOptions { MaxPlayers = 8 }, TypedLobby.Default);

// State
bool inRoom = PhotonNetwork.InRoom;
bool isMaster = PhotonNetwork.IsMasterClient;
Player localPlayer = PhotonNetwork.LocalPlayer;
Player[] players = PhotonNetwork.PlayerList;

// Objects
GameObject obj = PhotonNetwork.Instantiate("Actor", pos, rot);
PhotonNetwork.Destroy(obj);

// Time
double serverTime = PhotonNetwork.Time;
int ping = PhotonNetwork.GetPing();
```
