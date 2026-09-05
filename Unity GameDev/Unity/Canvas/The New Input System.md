> This guide explains Unity's New Input System from scratch.
> It walks through core concepts, your actual code, and answers every common question.
> Read it top to bottom the first time, then use it as a reference.



![[Recording 20260809195931.m4a]]

![[Recording 20260809201306.m4a]]
Alright, focusing specifically on the asset itself (the `.inputactions` file) — here are the distinct ways to consume it:

### 1. `PlayerInput` component (asset dragged into Inspector)

Attach `PlayerInput`, drag your `.inputactions` asset into its **Actions** field. This is the "no manual enable/disable" path — the component handles enabling the active action map and routing callbacks for you. Pick a **Behavior**:

|Behavior|Usage|
|---|---|
|Send Messages|Unity calls `On<ActionName>(InputValue value)` on the same GameObject, by naming convention|
|Broadcast Messages|Same, but also calls on children|
|Invoke Unity Events|Wire a method in the Inspector via `UnityEvent`, zero code|
|Invoke C# Events|Subscribe in code: `playerInput.onActionTriggered += ctx => {...}`|

csharp

```csharp
public void OnMove(InputValue value) // Send Messages behavior
{
    Vector2 move = value.Get<Vector2>();
}
```

Easiest to set up, but least flexible for things like manually toggling action maps.

### 2. Generated C# class (the recommended workflow)

In the asset's Inspector, tick **Generate C# Class**. Unity creates a strongly-typed wrapper class (usually matching your asset's name, e.g. `PlayerControls`) with a property per action map and per action — no string lookups, no typos, full autocomplete.

csharp

```csharp
public class PlayerMovement : MonoBehaviour
{
    private PlayerControls controls;

    void Awake()  { controls = new PlayerControls(); }
    void OnEnable()  { controls.Gameplay.Enable(); }
    void OnDisable() { controls.Gameplay.Disable(); }

    void OnEnable2()
    {
        controls.Gameplay.Jump.performed += OnJump; // event style
    }

    void Update()
    {
        Vector2 move = controls.Gameplay.Move.ReadValue<Vector2>(); // polling style
    }

    void OnJump(InputAction.CallbackContext ctx) { }
}
```

This is what most real projects use — you get both polling (`ReadValue`) and event subscription (`.performed +=`, `.started +=`, `.canceled +=`) on the same generated action, and you control exactly when action maps are enabled (e.g. swap `Gameplay` for `UI` on pause).

### 3. Raw `InputActionAsset` reference — no codegen

Drag the asset into a public field, then look up maps/actions by string name at runtime. No generated class needed, but you lose compile-time safety.

csharp

```csharp
public InputActionAsset inputActions;
private InputAction moveAction;

void Awake()
{
    var map = inputActions.FindActionMap("Gameplay");
    moveAction = map.FindAction("Move");
}

void OnEnable()  { moveAction.Enable(); }
void OnDisable() { moveAction.Disable(); }

void Update() { Vector2 move = moveAction.ReadValue<Vector2>(); }
```

### 4. `InputActionReference` — reference a single action directly

Instead of referencing the whole asset, expose a field of type `InputActionReference` and drag **one specific action** from the asset onto it in the Inspector. Good for reusable components that only care about one action (e.g. a generic "Interact" button component you drop on many prefabs).

csharp

```csharp
public InputActionReference jumpActionRef;

void OnEnable()  { jumpActionRef.action.Enable(); jumpActionRef.action.performed += OnJump; }
void OnDisable() { jumpActionRef.action.Disable(); jumpActionRef.action.performed -= OnJump; }

void OnJump(InputAction.CallbackContext ctx) { }
```

### 5. `InputActionMap.actionTriggered` — one subscription for a whole map

Instead of subscribing per-action, subscribe once at the **map** level and branch on `ctx.action.name` inside. Useful when you want a single funnel/log point for everything in a map.

csharp

```csharp
void OnEnable()
{
    controls.Gameplay.Enable();
    controls.Gameplay.Get().actionTriggered += OnAnyAction;
}
void OnAnyAction(InputAction.CallbackContext ctx)
{
    if (ctx.action.name == "Move") { /* ... */ }
}
```

### Quick decision table

| Approach                      | Type safety               | Setup effort          | Best for                         |
| ----------------------------- | ------------------------- | --------------------- | -------------------------------- |
| `PlayerInput` (Send Messages) | Low (string method names) | Lowest                | Quick prototypes                 |
| `PlayerInput` (C# Events)     | Medium                    | Low                   | Prototypes needing code control  |
| Generated C# class            | Highest                   | Medium (one checkbox) | Most real projects               |
| Raw `InputActionAsset`        | Low (string lookups)      | Medium                | Dynamic/data-driven input setups |
| `InputActionReference`        | Medium                    | Low per-field         | Reusable, decoupled components   |
| `actionTriggered` on a map    | Low                       | Low                   | Central logging/debug funnels    |

---

## Table of Contents

- [[#Part 1 Core Concepts]]
- [[#Part 2 The PlayerInput Component]]
- [[#Part 3 How Your Code Works]]
- [[#Part 4 Common Questions Answered]]
- [[#Part 5 Quick Reference]]

---

# Part 1: Core Concepts

## What Is The New Input System?

The New Input System is a **Unity Package** — it is not built into the Unity engine core.
You install it from the Package Manager.
It replaces the old `Input.GetKey()` approach.

It has four layers:

```
┌─────────────────────────────────────────────────────────┐
│  LAYER 4 — Visual Editor (UI)                           │
│  The three-column editor you use to set up inputs       │
├─────────────────────────────────────────────────────────┤
│  LAYER 3 — JSON Data (.inputactions file)               │
│  Stores your action maps, actions, and bindings         │
├─────────────────────────────────────────────────────────┤
│  LAYER 2 — C# Package (UnityEngine.InputSystem)         │
│  All the classes you work with in code                  │
├─────────────────────────────────────────────────────────┤
│  LAYER 1 — C++ Native Engine (Hidden)                   │
│  Polls the physical hardware every frame                │
└─────────────────────────────────────────────────────────┘
```

You only ever touch **Layers 2, 3, and 4**.
Layer 1 is invisible and automatic.


---

## The Class Hierarchy

Everything in the Input System follows this containment hierarchy:

```
InputActionAsset          ← ScriptableObject, the .inputactions file
  └── InputActionMap      ← Groups actions by context ("Player", "UI")
        └── InputAction   ← One logical game event ("Jump", "Move", "Link")
              └── InputBinding  ← Links the action to a physical key/button
                    └── InputControl  ← The actual key/button on the device
```

**Plain English:**
- An **Asset** holds everything.
- A **Map** is a group of related actions (like a "mode" of your game).
- An **Action** is something your player can do ("Jump").
- A **Binding** says which physical button triggers that action (`<Keyboard>/space`).
- A **Control** is the runtime representation of that physical button.

When you create an Input Action Asset via UI, Unity creates a `.inputactions` file that looks like this internally:
```json
{
  "name": "PlayerControls",
  "maps": [
    {
      "name": "Player",
      "actions": [
        {
          "name": "Move",
          "type": "Value",
          "expectedControlType": "Vector2"
        },
        {
          "name": "Jump",
          "type": "Button"
        }
      ],
      "bindings": [
        {
          "path": "<Keyboard>/space",
          "action": "Jump",
          "groups": "Keyboard&Mouse"
        },
        {
          "path": "<Keyboard>/w",
          "action": "Move/up",
          "groups": "Keyboard&Mouse"
        }
      ]
    }
  ],
  "controlSchemes": [
    {
      "name": "Keyboard&Mouse",
      "devices": [
        {"devicePath": "<Keyboard>"},
        {"devicePath": "<Mouse>"}
      ]
    }
  ]
}

```
*This JSON is loaded at runtime and converted into C# objects* (`InputActionMap`, `InputAction`, `InputBinding`).

---

## The Core Classes Explained

### `InputActionAsset`
**What it is:** A `ScriptableObject` that stores your entire input configuration.
**Where it lives:** As a `.inputactions` file in your Project Assets.
**What it contains:** Multiple `InputActionMap` objects.

```csharp
// Reference it in Inspector or load it
public InputActionAsset controls;

// Enable/disable all maps at once
controls.Enable();
controls.Disable();

// Find a specific map
InputActionMap playerMap = controls.FindActionMap("Player");
```

---

### `InputActionMap`
**What it is:** A named container for a group of `InputAction` objects.
**Why it exists:** Different game states need different inputs.
  - "Player" map: Move, Jump, Attack
  - "UI" map: Navigate, Submit, Cancel
  - "Vehicle" map: Accelerate, Steer, Brake

Only **one map should be active** at a time in most games.

```csharp
InputActionMap playerMap = controls.FindActionMap("Player");
playerMap.Enable();   // activate all actions in this group
playerMap.Disable();  // deactivate all actions in this group
```

---

### [[InputAction in New Input System]]
**What it is:** Represents one logical thing a player can do.
**Three types:**

| Type | Use case | Example |
|---|---|---|
| `Button` | Press/release events | Jump, Fire |
| `Value` | Continuous value with change | Move (Vector2), Aim |
| `PassThrough` | Like Value but allows multiple devices at once | Multiplayer |

**Three phases of an action:**

```
started   → The input begins    (e.g., key starts being held)
performed → The interaction completes (e.g., key is pressed fully)
canceled  → The input ends      (e.g., key released)
```

**You subscribe to phases:**
```csharp
jumpAction.performed += OnJump;   // subscribe
jumpAction.performed -= OnJump;   // unsubscribe (important in OnDisable!)
```

---

### `InputBinding`
**What it is:** A struct that stores the **path string** linking an action to a control.

```
Action "Jump"
  ├── Binding: "<Keyboard>/space"      ← keyboard spacebar
  └── Binding: "<Gamepad>/buttonSouth" ← gamepad A button
```

The path `"<Keyboard>/space"` is just a string at design time.
At runtime, Unity resolves it into an actual `InputControl` object.

---

### `InputControl`
**What it is:** The runtime object representing one physical input (one key, one stick axis, one button).

You rarely use `InputControl` directly.
It is what the Input System queries to read hardware state.

Examples:
- `Keyboard.current.spaceKey` → the spacebar's `ButtonControl`
- `Gamepad.current.leftStick` → the left stick's `StickControl`
- `Mouse.current.position` → mouse position `Vector2Control`

---

### `InputDevice`
**What it is:** Represents a physical hardware device.

```csharp
// Access current devices directly
Keyboard keyboard = Keyboard.current;
Mouse mouse = Mouse.current;
Gamepad gamepad = Gamepad.current;

// Check a specific key without InputAction
if (Keyboard.current.spaceKey.wasPressedThisFrame)
    Debug.Log("Space pressed!");

// List all connected devices
foreach (InputDevice device in InputSystem.devices)
    Debug.Log(device.name);
```

`InputDevice` is the parent class. `Keyboard`, `Mouse`, `Gamepad` are subclasses.

---

### `InputSystem` (static class)
**What it is:** The global manager — think of it as the control room.
**It is static** — you call it like `InputSystem.AddDevice(...)`.

Key things you can do:
```csharp
// Get all connected devices
InputSystem.devices;

// React when any device connects or disconnects
InputSystem.onDeviceChange += (device, change) => {
    Debug.Log($"Device {device.name}: {change}");
};

// Add a virtual device (useful for testing)
var keyboard = InputSystem.AddDevice<Keyboard>();

// Remove a device
InputSystem.RemoveDevice(device);
```

In most games, you **never call `InputSystem` directly** — the package manages it for you.

---

## How Path Resolution Works

When you write `"<Keyboard>/space"` as a binding, here is exactly what Unity does at runtime:

```
Step 1: Parse "<Keyboard>"
        → Search InputSystem.devices for a device of type Keyboard
        → Found: Keyboard.current

Step 2: Parse "/space"
        → Query Keyboard.current for a child control named "space"
        → Found: ButtonControl (the spacebar)

Step 3: Each frame, read the ButtonControl's state
        → isPressed? wasPressedThisFrame? → trigger the action
```

This means:
- Bindings are **data** (strings), not hardcoded control references.
- You can **rebind at runtime** by changing the path string — no code change needed.
- If no matching device exists, the binding simply doesn't fire.

---

# Part 2: The PlayerInput Component

## What Is PlayerInput?

`PlayerInput` is a **Unity Component** — you add it to a `GameObject` in the Inspector.

In Unity's object hierarchy:
```
Object                ← Everything in Unity
  └── Component       ← Attached to GameObjects
        └── Behaviour ← Components that can be enabled/disabled
              └── MonoBehaviour     ← Standard Unity script base
                    └── PlayerInput ← THIS — a built-in Unity component
```

It is **not** a GameObject. It is **not** a Manager script you write.
It is a pre-built component from the Input System package.

---

## What Does PlayerInput Do?

`PlayerInput` is the **bridge** between your `.inputactions` asset and your code.

Without it, you must:
1. Load the asset manually
2. Find the right action
3. Subscribe to callbacks
4. Enable/disable everything yourself

So how will it all look without the PlayerInput?

With it, you just:
1. Drag the asset into the `Actions` field in the Inspector
2. PlayerInput handles enabling, subscribing, and routing callbacks

**But how would it look without any PlayerInput attached to the GameObject?**
[[Using Unity's New Input System Without PlayerInput]]

---

## PlayerInput Key Properties

| Property | Type | What It Does |
|---|---|---|
| `actions` | `InputActionAsset` | The .inputactions asset assigned in Inspector |
| `currentActionMap` | `InputActionMap` | The currently active action map |
| `notificationBehavior` | `PlayerNotifications` | How callbacks are sent to your code |
| `camera` | `Camera` | Used for split-screen multiplayer |
| `playerIndex` | `int` | Index when using PlayerInputManager for multiplayer |
| `splitScreenIndex` | `int` | Split-screen area index |
| `controlScheme` | `string` | Name of the active control scheme |
| `hasMissingRequiredDevices` | `bool` | True if required device is not connected |

---

## PlayerInput Key Methods

```csharp
// Switch to a different action map by name
playerInput.SwitchCurrentActionMap("UI");

// Activate a specific control scheme
playerInput.SwitchCurrentControlScheme("Gamepad", Gamepad.current);

// Pause/resume all input processing
playerInput.DeactivateInput();
playerInput.ActivateInput();
```

---

## PlayerNotifications — How Callbacks Reach Your Code

This is the `notificationBehavior` property. It controls **how** PlayerInput tells your code that an action fired.

There are four modes:

### Mode 1: `SendMessages` (Default)
PlayerInput calls `SendMessage("OnActionName", value)` on the **same GameObject**.
The PlayerInput gives the list of all the methods that it'll call as a callback. The callback method had to be on a component attached to the GameObject on which we have enabled the player input.

![[Pasted image 20260518214301.png]]

```
PlayerInput fires "Jump" action
  → Calls SendMessage("OnJump", value)
  → Unity searches every component on THIS GameObject
  → Finds your method: public void OnJump(InputValue value)
  → Calls it
```

**Rule:** Your method must be `public`, on the **same GameObject**, named `On` + ActionName.

```csharp
// This is automatically called — you don't connect it anywhere
public void OnJump(InputValue value)
{
    if (value.isPressed) Jump();
}

public void OnMove(InputValue value)
{
    moveDir = value.Get<Vector2>();
}
```

### Mode 2: `BroadcastMessages`
Same as `SendMessages` but also searches **child GameObjects**.

Use this when your handler script is on a child object, not the same object as PlayerInput.

### Mode 3: `InvokeUnityEvents`
PlayerInput exposes **UnityEvents** in the Inspector.
You drag your script's method into the event slot manually.

```
In Inspector:
PlayerInput component
  └── Events
        └── Player
              └── Jump: [drag script here] → OnLinkPerformed()
```

Your callback signature must be:
```csharp
public void OnLinkPerformed(InputAction.CallbackContext context)
{
    if (context.performed) ActivateLink();
}
```

### Mode 4: `InvokeCSharpEvents`
PlayerInput exposes C# events you subscribe to in code.

```csharp
playerInput.onActionTriggered += OnAnyAction;

void OnAnyAction(InputAction.CallbackContext context)
{
    if (context.action.name == "Jump") Jump();
}
```

---

## Multiple InputActionAssets

Can you have multiple `.inputactions` assets?
**Yes.** But only one `PlayerInput` component can have one asset at a time.

Common patterns:

**Pattern A — One asset, multiple maps (recommended)**
```
PlayerControls.inputactions
  ├── Player map    (Movement, Jump, Attack)
  ├── UI map        (Navigate, Submit, Cancel)
  └── Vehicle map   (Accelerate, Steer, Brake)
```
Switch with: `playerInput.SwitchCurrentActionMap("Vehicle")`

**Pattern B — Multiple assets, one PlayerInput**
You can swap the entire `actions` asset at runtime:
```csharp
playerInput.actions = newAsset;  // replace the whole asset
```

**Pattern C — Multiple PlayerInput components (multiplayer)**
Each player GameObject gets its own `PlayerInput` with its own asset.
Use `PlayerInputManager` to manage this automatically.

---

## Switching Action Maps

```csharp
// Via PlayerInput component
GetComponent<PlayerInput>().SwitchCurrentActionMap("UI");

// Via InputActionAsset directly
controls.FindActionMap("Player").Disable();
controls.FindActionMap("UI").Enable();

// Via generated C# class
controls.Player.Disable();
controls.UI.Enable();
```

**Important rule:** When using `PlayerInput`, call `SwitchCurrentActionMap()` — it handles enabling/disabling cleanly. Don't manually enable/disable maps when PlayerInput is managing them.

---

# Part 3: How Your Code Works

## The Setup

Your project uses two scripts:

```
┌───────────────────────────────────────────────┐
│ GameObject: "GameManager" (DontDestroyOnLoad) │
│   ├── InputManager.cs   (your script)         │
│   └── PlayerInput       (Unity component)     │
└───────────────────────────────────────────────┘

┌───────────────────────────────────────────────┐
│ GameObject: "Player"                          │
│   └── InputEventHandler.cs  (your script)     │
└───────────────────────────────────────────────┘
```

`InputManager` sits alongside `PlayerInput` on the same GameObject.
`InputEventHandler` is on the player — it reads clean state from `InputManager`.

---

## InputManager.cs — Line by Line

```csharp
public class InputManager : MonoBehaviour
{
    // Singleton — only one InputManager exists across all scenes
    public static InputManager Instance { get; private set; }

    // These are the "clean" input states other scripts can read
    public Vector2 MoveInput { get; private set; }
    public bool IsJumping { get; private set; }
    public bool IsLinkActive { get; private set; }
```

**Why a singleton?**
Any script anywhere needs to read input.
A singleton means you just call `InputManager.Instance.MoveInput` from anywhere.

---

```csharp
    private void Awake()
    {
        if (Instance != null && Instance != this) { Destroy(gameObject); return; }
        Instance = this;
        DontDestroyOnLoad(gameObject);
        playerInput = GetComponent<PlayerInput>();
    }
```

**What happens here:**
1. First time Awake runs: `Instance` is null → this becomes the singleton.
2. If a second InputManager ever loads: it destroys itself.
3. `DontDestroyOnLoad` keeps this alive across scene changes.
4. `GetComponent<PlayerInput>()` grabs the PlayerInput on this same GameObject.

---

```csharp
    public void OnMove(InputValue value)
    {
        MoveInput = value.Get<Vector2>();
    }

    public void OnJump(InputValue value)
    {
        IsJumping = value.isPressed;
    }

    public void OnLink(InputValue value)
    {
        IsLinkActive = value.isPressed;
    }
```

**What happens here — the key magic:**

1. PlayerInput component fires the "Move" action (player pressed WASD).
2. PlayerInput's `notificationBehavior` is set to `SendMessages`.
3. `SendMessage("OnMove", inputValue)` is called on this GameObject.
4. Unity finds `public void OnMove(InputValue value)` — because the name matches.
5. The method stores the clean `Vector2` in `MoveInput`.
6. Any other script can now read `InputManager.Instance.MoveInput`.

**The naming rule is non-negotiable with SendMessage:**
Action name `"Move"` → method must be `OnMove`
Action name `"Link"` → method must be `OnLink`
Action name `"Jump"` → method must be `OnJump`

---

```csharp
    public void SwitchToMap(string mapName)
    {
        playerInput.SwitchCurrentActionMap(mapName);
        currentMapName = mapName;
    }
```

**What happens:** PlayerInput disables the current map, enables the named map.
After this call, only actions from the new map will fire.

---

## InputEventHandler.cs — Line by Line

```csharp
public class InputEventHandler : MonoBehaviour
{
    private void Update()
    {
        HandleMovement();
        HandleLink();
    }
```

**Why Update() here, not a callback?**
Movement is continuous — the player holds a key and the character keeps moving.
Polling `MoveInput` every frame is correct for this.
Callbacks are better for one-shot events like jump or attack.

---

```csharp
    private void HandleMovement()
    {
        Vector2 moveDir = InputManager.Instance.MoveInput;
        rb.linearVelocity = new Vector2(moveDir.x * moveSpeed, rb.linearVelocity.y);
    }
```

**What happens:**
1. Every frame, read the current `MoveInput` from `InputManager`.
2. `InputManager.MoveInput` was set by `OnMove()` when the action fired.
3. Apply it as velocity.

No direct contact with the Input System at all — clean separation.

---

```csharp
    public void OnLinkPerformed(InputAction.CallbackContext context)
    {
        if (context.performed) ActivateLink();
    }
```

**This method is wired via UnityEvents (InvokeUnityEvents mode).**

In the Inspector, on the PlayerInput component:
```
Events → Player → Link (performed): [drag InputEventHandler here] → OnLinkPerformed
```

This method takes `CallbackContext` — explained in detail in Part 4.

---

## The Full Flow — What Happens When Player Presses Space

```
Frame N: Player presses Spacebar
│
├─ C++ layer detects key change
│
├─ Input Update Phase (before Update()):
│    InputSystem checks "Jump" action bindings
│    Finds: "<Keyboard>/space" → space is pressed
│    Triggers "Jump" action → phase: performed
│
├─ PlayerInput (on GameManager) receives the trigger
│    notificationBehavior = SendMessages
│    Calls: SendMessage("OnJump", inputValue)
│
├─ InputManager.OnJump(value) runs:
│    IsJumping = value.isPressed  → IsJumping = true
│
└─ Your Update() runs:
     Can now read InputManager.Instance.IsJumping == true
```

---

# Part 4: Common Questions Answered

## Q1: How does generating a C# class work? Does it stay in sync?

**Short answer:** No, it does NOT auto-sync. You must regenerate it manually.

**What happens when you click "Generate C# Class":**
1. Unity reads your current `.inputactions` JSON.
2. It generates a `.cs` file with wrapper properties.
3. Example output for a "Jump" action in "Player" map:

```csharp
// This is the GENERATED class — you did not write this
public class PlayerControls : IInputActionCollection2, IDisposable
{
    public InputActionAsset asset { get; }

    // Strongly-typed access to the "Player" map
    public @Player @Player => new @Player(this);

    public struct @Player
    {
        // Strongly-typed access to the "Jump" action
        public InputAction @Jump => m_Wrapper.asset["Player/Jump"];
    }
}
```

**Usage in your code:**
```csharp
private PlayerControls controls;

void Awake()
{
    controls = new PlayerControls();            // loads the JSON
    controls.Player.Jump.performed += OnJump;  // type-safe, IntelliSense works!
}

void OnEnable()  { controls.Enable(); }
void OnDisable() { controls.Disable(); }
```

**Why use it?**
- Auto-complete works — `controls.Player.Jump` instead of `FindAction("Jump")`
- Compile-time error if you mistype a name
- Cleaner code

**The catch — it does NOT auto-update:**
```
You add new "Dash" action via UI editor
↓
The .inputactions JSON is updated
↓
The generated .cs file is NOT updated (it's stale)
↓
controls.Player.Dash → compile error! Property doesn't exist yet
↓
You must click "Generate C# Class" again to regenerate
```

**So is the generated class its own InputActionMap?**
No. It is a **wrapper/facade** over the same underlying asset.
`new PlayerControls()` loads the JSON and creates the same `InputActionMap`/`InputAction` objects internally.
The generated class just gives you typed access to them.

---

## Q2: What is CallbackContext?

`InputAction.CallbackContext` is a **struct** passed to your callback method.
It is a snapshot of everything about the input event at the moment it fired.

```csharp
void OnJump(InputAction.CallbackContext context)
{
    // Which phase fired?
    bool started   = context.started;    // key just began pressing
    bool performed = context.performed;  // key fully pressed (or threshold met)
    bool canceled  = context.canceled;   // key released

    // Which action triggered this?
    InputAction action = context.action;   // the Jump action object
    string actionName  = context.action.name;  // "Jump"

    // Which device/control fired it?
    InputControl control = context.control;    // e.g., Keyboard.space
    InputDevice device   = context.control.device; // the Keyboard device

    // Read the value
    float value = context.ReadValue<float>();    // 0 or 1 for a button
    Vector2 vec = context.ReadValue<Vector2>();  // for Move action
}
```

**Why does it exist?**
A single callback method can handle all three phases.
Without `context`, you wouldn't know if the input started, completed, or stopped.

**Typical usage:**
```csharp
// Button action — act on press only
jumpAction.performed += ctx => Jump();

// Value action — read the value
moveAction.performed += ctx => {
    moveDir = ctx.ReadValue<Vector2>();
};

// Handle all phases in one method
jumpAction.started   += ctx => Debug.Log("jump started");
jumpAction.performed += ctx => Jump();
jumpAction.canceled  += ctx => Debug.Log("jump canceled");
```

---

## Q3: How does Unity know when to call `OnLinkPerformed`? It's never referenced anywhere.

**This depends on which notification mode you use.**

### If using SendMessages:
Unity uses C#'s **reflection** system.

When the "Link" action fires, PlayerInput does this internally:
```csharp
// Inside PlayerInput source code (simplified):
gameObject.SendMessage("OnLink", inputValue, SendMessageOptions.DontRequireReceiver);
```

`SendMessage` scans every component on the GameObject.
It looks for any method named `"OnLink"` using reflection.
If found — calls it. If not found — silently skips (DontRequireReceiver).

Your method is "discovered" automatically at runtime, not at compile time.
This is why you never need to wire it up — Unity finds it by name.

### If using InvokeUnityEvents:
You explicitly drag your method into the Inspector.
```
PlayerInput Inspector
  └── Events
        └── Player
              └── Link (performed): InputEventHandler → OnLinkPerformed
```

PlayerInput stores this reference and calls it directly.
No reflection involved — it's a stored delegate.

**Why is InvokeUnityEvents more reliable?**
- Compile-time safety: if you rename the method, the Inspector reference breaks (you'll see it).
- With SendMessages: if you rename `OnLink` to something else, it silently stops working.

---

## Q4: What does `InputValue` vs `CallbackContext` mean?

These are two different signatures for input callbacks.

| | `InputValue` | `CallbackContext` |
|---|---|---|
| Used with | `SendMessages` mode | `InvokeUnityEvents` or direct subscription |
| Phase info | Only fires at `performed` | Fires at started/performed/canceled |
| Read value | `value.Get<T>()` | `context.ReadValue<T>()` |
| Example | `OnJump(InputValue v)` | `OnJump(CallbackContext ctx)` |

```csharp
// SendMessages signature (InputValue)
public void OnJump(InputValue value)
{
    if (value.isPressed) Jump();  // only fires when performed
}

// InvokeUnityEvents / direct subscription signature (CallbackContext)
public void OnJump(InputAction.CallbackContext context)
{
    if (context.performed) Jump();
}
```

---

## Q5: How to switch between different action maps?

```csharp
// Method 1 — Via PlayerInput component (recommended when using PlayerInput)
GetComponent<PlayerInput>().SwitchCurrentActionMap("UI");

// Method 2 — Via the InputActionAsset directly
playerInput.actions.FindActionMap("Player").Disable();
playerInput.actions.FindActionMap("UI").Enable();

// Method 3 — Via generated C# class
controls.Player.Disable();
controls.UI.Enable();
```

**Practical example — opening a menu:**
```csharp
void OpenMenu()
{
    playerInput.SwitchCurrentActionMap("UI");
    // Now WASD moves menu cursor, not character
    // Jump button submits, not jumps
}

void CloseMenu()
{
    playerInput.SwitchCurrentActionMap("Player");
    // Back to normal player controls
}
```

---

## Q6: How do multiple InputActionAssets work?

**One asset is usually enough** — use multiple maps within one asset.

If you truly need multiple assets:

```csharp
[SerializeField] private InputActionAsset playerAsset;
[SerializeField] private InputActionAsset vehicleAsset;

private PlayerInput playerInput;

void SwitchToVehicle()
{
    playerInput.actions = vehicleAsset;  // swap the entire asset
    // PlayerInput re-subscribes to the new asset automatically
}

void SwitchToPlayer()
{
    playerInput.actions = playerAsset;
}
```

**When would you have two assets?**
- When two team members work on input simultaneously (merge conflicts)
- When modular DLC adds entirely new control schemes
- Almost never in a typical game

---

# Part 5: Quick Reference

## Class Summary

| Class | Type | Purpose |
|---|---|---|
| `InputActionAsset` | ScriptableObject | Holds all maps/actions/bindings |
| `InputActionMap` | Class | Groups related actions by context |
| `InputAction` | Class | One game input event |
| `InputBinding` | Struct | Path string linking action to control |
| `InputControl` | Class | Runtime physical button/axis |
| `InputDevice` | Class | Physical hardware device |
| `InputSystem` | Static class | Global manager and utilities |
| `PlayerInput` | Component | Bridge between asset and your code |
| `CallbackContext` | Struct | Snapshot of input event data |
| `InputValue` | Class | Simplified value (used with SendMessages) |

---

## Notification Behavior Comparison

| Mode | Signature | Auto-wired | Best for |
|---|---|---|---|
| `SendMessages` | `OnAction(InputValue v)` | Yes (by name) | Quick prototyping |
| `BroadcastMessages` | `OnAction(InputValue v)` | Yes (by name) | Child component handlers |
| `InvokeUnityEvents` | `OnAction(CallbackContext ctx)` | Inspector drag | Visual wiring, clarity |
| `InvokeCSharpEvents` | `OnAction(CallbackContext ctx)` | Code subscription | Full code control |

---

## Binding Path Format

```
"<DeviceType>/controlName"

Examples:
"<Keyboard>/space"           spacebar
"<Keyboard>/w"               W key
"<Mouse>/leftButton"         left mouse button
"<Mouse>/delta"              mouse movement (Vector2)
"<Gamepad>/buttonSouth"      A button (Xbox) / X button (PlayStation)
"<Gamepad>/leftStick"        left analog stick (Vector2)
"<Gamepad>/leftTrigger"      left trigger (float 0-1)
"<Touchscreen>/primaryTouch/position"  finger position (Vector2)
```

---

## CallbackContext — What You Can Read

```csharp
void OnAnyAction(InputAction.CallbackContext ctx)
{
    ctx.started           // bool: input just began
    ctx.performed         // bool: input completed
    ctx.canceled          // bool: input ended

    ctx.action            // InputAction: which action fired
    ctx.action.name       // string: "Jump", "Move", etc.
    ctx.control           // InputControl: which button/key fired it
    ctx.control.device    // InputDevice: which device

    ctx.ReadValue<float>()    // read float (button: 0 or 1)
    ctx.ReadValue<Vector2>()  // read Vector2 (stick, WASD composite)
    ctx.ReadValue<bool>()     // read bool
    ctx.time                  // double: timestamp when this fired
}
```

---

## Action Map Switching Cheat Sheet

```csharp
// Recommended — PlayerInput handles cleanup
playerInput.SwitchCurrentActionMap("MapName");

// Low-level — if not using PlayerInput
asset.FindActionMap("Player").Disable();
asset.FindActionMap("UI").Enable();

// Generated class
controls.Player.Disable();
controls.UI.Enable();
```

---

## The Three Ways to Connect Input to Your Code

```
Option 1: PlayerInput + SendMessages
  Pro: Least code, just write OnMove() and it works
  Con: Silent failure if you misspell the method name

Option 2: PlayerInput + InvokeUnityEvents  
  Pro: Visual wiring, clear in Inspector, CallbackContext gives full info
  Con: Requires Inspector setup per action

Option 3: Generated C# Class (no PlayerInput)
  Pro: Full type safety, no magic strings, IntelliSense
  Con: Must regenerate when you change the asset, more boilerplate

Rule of thumb:
  Prototyping / learning     → SendMessages
  Mid-size game              → InvokeUnityEvents
  Large game / team project  → Generated C# class
```

---

## Best Practices

1. **Always unsubscribe in OnDisable.**
   ```csharp
   void OnEnable()  { jumpAction.performed += OnJump; }
   void OnDisable() { jumpAction.performed -= OnJump; }  // prevents memory leaks
   ```

2. **Don't subscribe in Update.** Subscribe once in Awake/OnEnable.

3. **Use SwitchCurrentActionMap instead of manually enabling/disabling** when using PlayerInput.

4. **Don't read input in FixedUpdate.** Input callbacks run before Update, not FixedUpdate.
   Store the value in a field in the callback, apply it in FixedUpdate.

5. **Re-generate the C# class after every change** to your .inputactions file.

6. **One InputActionMap active at a time** keeps logic clean and prevents conflicting inputs.

---

*This guide was written to companion the files: `input_manager.cs` and `input_event_handler.cs`*

````
# Unity's New Input System — Complete Guide

**A from-scratch explanation of Unity's New Input System with entirely new code examples, architecture patterns, and practical implementation.**

---

## Table of Contents

1. [Part 1: Why the New Input System Exists](#part-1-why-the-new-input-system-exists)
2. [Part 2: Core Concepts & Architecture](#part-2-core-concepts--architecture)
3. [Part 3: The Class Hierarchy](#part-3-the-class-hierarchy)
4. [Part 4: PlayerInput Component](#part-4-playerinput-component)
5. [Part 5: Action Types & Callback Context](#part-5-action-types--callback-context)
6. [Part 6: Notification Patterns](#part-6-notification-patterns)
7. [Part 7: The InputManager Pattern](#part-7-the-inputmanager-pattern)
8. [Part 8: Action Map Switching](#part-8-action-map-switching)
9. [Part 9: Real Code Walkthrough - Spaceship Game](#part-9-real-code-walkthrough---spaceship-game)
10. [Part 10: Advanced Topics](#part-10-advanced-topics)
11. [Part 11: Quick Reference](#part-11-quick-reference)

---

## Part 1: Why the New Input System Exists

### The Old System's Problems

Unity's old `Input.GetKey()` approach had critical limitations:

```csharp
// ❌ OLD WAY — Everything hardcoded and messy
public class OldSpaceshipController : MonoBehaviour
{
    void Update()
    {
        // Hardcoded keys scattered everywhere
        if (Input.GetKeyDown(KeyCode.W))
            Thrust();
        
        if (Input.GetKeyDown(KeyCode.Space))
            FireWeapon();
        
        float rotation = Input.GetAxis("Horizontal"); // hidden in Project Settings
        RotateShip(rotation);
        
        // Want to support gamepad? Write it all again with different code
        float gamepadThrust = Input.GetAxis("GamepadThrottle");
        
        // Want players to rebind? Nightmare - edit code or write complex rebinding system
    }
}
````

**Problems:**

- **Hardcoded bindings** — Change W to Shift? Edit code everywhere.
- **No rebinding** — Players can't remap Thrust from W to E.
- **Single device** — Keyboard OR gamepad, awkward to support both.
- **No VR/Touch** — Mobile/VR required completely separate code paths.
- **Hidden configuration** — "Horizontal" axis buried in Project Settings.
- **Polling-based** — You check every frame whether input happened or not (wasteful).

### What the New System Solves

Code

```
✅ Action-based      — Define "Thrust" once, bind to W / RB trigger / VR squeeze
✅ Rebindable        — Players change controls at runtime with UI
✅ Multi-device      — Keyboard, gamepad, touch, VR use identical code
✅ Event-driven      — Input fires events only when needed
✅ Type-safe         — Actions defined in assets, can generate C# code
✅ Platform-agnostic — One "FireWeapon" action works everywhere
✅ Organized         — All bindings visible in Inspector, not scattered in code
```

---

## Part 2: Core Concepts & Architecture

### The Data Flow

Code

```
┌─────────────────────────────────────────────────────────────┐
│ INPUT ACTIONS ASSET (SpaceshipControls.inputactions)       │
│                                                             │
│ ├─ Combat (Action Map)                                    │
│ │  ├─ FirePrimary (Action) → W key / RB / Trigger         │
│ │  ├─ FireSecondary (Action) → Q key / LB / Shield btn    │
│ │  └─ Aim (Action) → Mouse movement / Gamepad stick       │
│ │                                                          │
│ └─ Navigation (Action Map)                                │
│    ├─ Throttle (Action) → Arrow keys / Gamepad triggers   │
│    ├─ Rotate (Action) → A/D keys / Gamepad stick          │
│    └─ ScanForEnemies (Action) → Space / Start button      │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ PLAYERINPUT COMPONENT                                       │
│ ├─ Loads the asset                                         │
│ ├─ Listens to all devices (keyboard, gamepad, mouse)       │
│ ├─ Detects when a binding is triggered                     │
│ └─ Fires callbacks to your code                            │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ YOUR CODE                                                   │
│ ├─ SpaceshipController subscribes to "FirePrimary"         │
│ ├─ RadarSystem subscribes to "ScanForEnemies"              │
│ ├─ EngineController subscribes to "Throttle"               │
│ └─ All react independently and cleanly                     │
└─────────────────────────────────────────────────────────────┘
```

### Key Terms

|Term|Definition|Example|
|---|---|---|
|**Input Actions Asset**|`.inputactions` file. Stores all maps, actions, bindings.|`SpaceshipControls.inputactions`|
|**Action Map**|Contextual group of actions (Combat, Navigation). Only one active at a time.|"Combat" map for battle, "Navigation" for flight|
|**Action**|One logical input event (FireWeapon, Thrust, Rotate). Can be Button or Value type.|"FirePrimary" — player shoots|
|**Binding**|Links an action to physical input. Can have multiple per action.|`<Keyboard>/w` or `<Gamepad>/triggerRight`|
|**PlayerInput**|MonoBehaviour component that bridges the asset and your code.|Attached to GameManager GameObject|
|**Callback Context**|Data passed to your event handler (phase, value, device info).|"Fire was pressed" or "Stick angle is 45°"|

---

## Part 3: The Class Hierarchy

Everything in the Input System follows this containment structure:

Code

```
InputActionAsset          ← ScriptableObject, the .inputactions file
  └── InputActionMap      ← Groups actions by context ("Combat", "Navigation")
        └── InputAction   ← One logical input event ("FirePrimary", "Thrust")
              └── InputBinding  ← Path linking action to control
                    └── InputControl  ← Runtime representation of physical button
```

### How It Works Under the Hood

When you create an Input Actions Asset in the Editor, Unity stores it as JSON internally:

JSON

```
{
  "name": "SpaceshipControls",
  "maps": [
    {
      "name": "Combat",
      "actions": [
        {
          "name": "FirePrimary",
          "type": "Button"
        },
        {
          "name": "Aim",
          "type": "Value",
          "expectedControlType": "Vector2"
        }
      ],
      "bindings": [
        {
          "path": "<Keyboard>/w",
          "action": "FirePrimary"
        },
        {
          "path": "<Gamepad>/triggerRight",
          "action": "FirePrimary"
        },
        {
          "path": "<Mouse>/delta",
          "action": "Aim"
        }
      ]
    }
  ]
}
```

At runtime, this JSON is deserialized into C# objects (`InputActionMap`, `InputAction`, `InputBinding`).

### Core Classes in Action

C#

```
// Load the asset
InputActionAsset spaceship = Resources.Load<InputActionAsset>("SpaceshipControls");

// Get a specific map
InputActionMap combatMap = spaceship.FindActionMap("Combat");

// Get a specific action
InputAction fireAction = spaceship.FindAction("FirePrimary");

// Enable the map (now actions fire when triggered)
combatMap.Enable();

// Check if an action is currently being held
if (fireAction.IsPressed())
    Debug.Log("Fire button is held down RIGHT NOW");

// Read the current value (for continuous inputs like Aim stick)
InputAction aimAction = spaceship.FindAction("Aim");
Vector2 aimDirection = aimAction.ReadValue<Vector2>();
```

---

## Part 4: PlayerInput Component

### What It Does

`PlayerInput` is a **MonoBehaviour** you attach to a GameObject. It's the bridge between your `.inputactions` asset and your code.

**Without PlayerInput (manual approach):**

C#

```
// You must do all this yourself
public class ManualInputSetup : MonoBehaviour
{
    private SpaceshipControlsInputActions _controls;

    void Awake()
    {
        _controls = new SpaceshipControlsInputActions();
        _controls.Enable();
        
        // Subscribe to every action manually
        _controls.Combat.FirePrimary.performed += ctx => FireWeapon();
        _controls.Combat.Aim.performed += ctx => UpdateAim(ctx.ReadValue<Vector2>());
        _controls.Navigation.Throttle.performed += ctx => UpdateThrottle(ctx.ReadValue<float>());
        
        // Handle map switching manually
        // Handle unsubscribing on destroy
        // Track which map is active
    }
}
```

**With PlayerInput (automatic):**

C#

```
// 1. Drag the asset into the Inspector
// 2. Choose a notification behavior
// 3. Wire up your methods (if using UnityEvents)
// Done! PlayerInput handles everything.
```

### Setup Steps

1. **Create a GameObject** (e.g., "GameManager")
2. **Add PlayerInput component** (Add Component → PlayerInput)
3. **Assign your `.inputactions` asset** to the Actions field
4. **Set Default Map** (which action map starts active, e.g., "Navigation")
5. **Choose Behavior** (how it notifies your code)
6. **Wire up in Inspector** (if using UnityEvents)

### PlayerInput Key Properties

C#

```
PlayerInput playerInput = GetComponent<PlayerInput>();

// Current state
InputActionMap currentMap = playerInput.currentActionMap;
string mapName = currentMap.name; // "Combat" or "Navigation"

// Get the asset
InputActionAsset asset = playerInput.actions;

// Check if a required device is missing
if (playerInput.hasMissingRequiredDevices)
    Debug.Log("Gamepad required but not connected!");

// Get player index (for multiplayer games)
int playerNum = playerInput.playerIndex; // 0, 1, 2, 3...

// Get active control scheme
string scheme = playerInput.controlScheme; // "Keyboard&Mouse" or "Gamepad"
```

### PlayerInput Methods

C#

```
// Switch to different action map
playerInput.SwitchCurrentActionMap("Combat");

// Switch control scheme
playerInput.SwitchCurrentControlScheme("Gamepad", Gamepad.current);

// Pause/resume all input processing
playerInput.DeactivateInput();   // stop listening to any input
playerInput.ActivateInput();     // resume listening

// Check what device triggered an action
void OnActionTriggered(InputAction.CallbackContext ctx)
{
    InputControl triggeredControl = ctx.control;
    InputDevice device = ctx.control.device;
    Debug.Log($"Action triggered by {device.name}: {triggeredControl.displayName}");
}
```

---

## Part 5: Action Types & Callback Context

### Action Types

Each action has a **type** that determines what data it produces:

#### Button — Discrete On/Off Input

C#

```
// Example: FirePrimary action
// In the asset: Action Type = Button
// Bindings: W key, Gamepad Right Trigger, Mouse left button

// In your code:
void OnFirePrimaryPerformed(InputAction.CallbackContext ctx)
{
    if (ctx.performed)  // ← only fires on full press, not release
    {
        GameObject projectile = Instantiate(
            projectilePrefab,
            firePoint.position,
            firePoint.rotation
        );
        projectile.GetComponent<Rigidbody>().velocity = firePoint.forward * bulletSpeed;
    }
}
```

**Phases for Button:**

- `started` — player just pressed the button
- `performed` — button is recognized as pressed (after debounce)
- `canceled` — button is released

#### Value — Continuous Data

C#

```
// Example: Throttle action (engine speed)
// In the asset: Action Type = Value, Control Type = Float
// Bindings: Gamepad Right Trigger (float 0-1), Arrow Up/Down (discrete)

// In your code - continuous value, use in Update
private float _throttleInput;

void OnThrottlePerformed(InputAction.CallbackContext ctx)
{
    _throttleInput = ctx.ReadValue<float>();  // 0 to 1
}

void Update()
{
    // Apply throttle every frame
    float speed = _throttleInput * maxEngineThrust;
    rb.velocity = transform.forward * speed;
}
```

Another Value example:

C#

```
// Example: Aim action (aiming crosshair)
// In the asset: Action Type = Value, Control Type = Vector2
// Bindings: Mouse delta movement, Gamepad Right Stick

private Vector2 _aimInput;

void OnAimPerformed(InputAction.CallbackContext ctx)
{
    _aimInput = ctx.ReadValue<Vector2>();  // -1 to 1, -1 to 1
}

void LateCamera()
{
    // Rotate camera based on aim input
    float horizontalRotation = _aimInput.x * aimSensitivity;
    float verticalRotation = _aimInput.y * aimSensitivity;
    
    cameraRig.Rotate(Vector3.up, horizontalRotation);
    cameraRig.Rotate(Vector3.right, verticalRotation);
}
```

#### PassThrough — Every Frame (Rare)

Used when you need to sample input every single frame without filtering. Almost never needed for games.

### CallbackContext — Everything You Receive

Every input callback receives an `InputAction.CallbackContext` struct with all the details:

C#

```
void OnAnyInput(InputAction.CallbackContext ctx)
{
    // ─── PHASE INFO ─────────────────────────────────
    bool started   = ctx.started;      // true when button first pressed
    bool performed = ctx.performed;    // true when full action recognized
    bool canceled  = ctx.canceled;     // true when button released
    
    // Example for FireSecondary (hold to charge):
    if (ctx.started)
        _chargeStartTime = Time.time;
    
    if (ctx.performed)
        _chargeAmount = Mathf.Clamp(Time.time - _chargeStartTime, 0, maxChargeTime);
    
    if (ctx.canceled)
        FireSecondaryWeapon(_chargeAmount);

    // ─── ACTION INFO ─────────────────────────────────
    InputAction action = ctx.action;          // the action object
    string actionName = ctx.action.name;      // "FirePrimary"

    // ─── DEVICE INFO ─────────────────────────────────
    InputControl control = ctx.control;              // which control triggered this
    InputDevice device = ctx.control.device;         // Keyboard, Mouse, Gamepad
    
    // Example: log what device fired the action
    Debug.Log($"Action '{actionName}' triggered on {device.displayName}");
    // Output: "Action 'FirePrimary' triggered on Keyboard"

    // ─── READ THE VALUE ─────────────────────────────
    float floatVal = ctx.ReadValue<float>();        // for single axis
    Vector2 vec = ctx.ReadValue<Vector2>();         // for 2D input
    bool pressed = ctx.ReadValueAsButton();          // bool interpretation
    
    // Example: read aiming direction
    Vector2 aimDir = ctx.ReadValue<Vector2>();
    Debug.DrawLine(shipCenter, shipCenter + (Vector3)aimDir * 10, Color.green);

    // ─── TIMING ─────────────────────────────────────
    double time = ctx.time;                          // when this fired (seconds)
    
    // Example: measure time between shots
    if (_lastShotTime > 0)
    {
        float timeSinceLastShot = (float)(ctx.time - _lastShotTime);
        Debug.Log($"Time since last shot: {timeSinceLastShot}s");
    }
    _lastShotTime = ctx.time;
}
```

---

## Part 6: Notification Patterns

`PlayerInput` has four ways to notify your code that an action fired. The `notificationBehavior` property controls which one.

### Pattern 1: Send Messages ❌ (Avoid)

PlayerInput calls `SendMessage("OnActionName", value)` on the same GameObject using reflection.

C#

```
// PlayerInput Behavior: Send Messages

public class SpaceshipOldWay : MonoBehaviour
{
    // Method name MUST match: On<ActionName>
    // PlayerInput finds it automatically via reflection at runtime
    
    public void OnFirePrimary(InputValue value)
    {
        if (value.isPressed)
        {
            // Only has the value, not the full context
            ShootMainWeapon();
        }
    }
    
    public void OnThrottle(InputValue value)
    {
        float throttle = value.Get<float>();
        ApplyEngineThrust(throttle);
    }
}
```

**Pros:**

- Automatic — no wiring needed

**Cons:**

- **Slow** — uses reflection every time
- **Silent failure** — typo the method name and it never calls
- **No type safety** — compile errors won't catch naming mistakes
- **Tight coupling** — must be on same GameObject as PlayerInput
- **Limited info** — no full CallbackContext access

**Avoid this.** Use it only for quick 5-minute prototyping.

### Pattern 2: Broadcast Messages ❌ (Avoid)

Same as Send Messages but searches child GameObjects too.

C#

```
// PlayerInput Behavior: Broadcast Messages

// PlayerInput will call SendMessage on THIS object AND all children
// Even slower due to extra GameObject hierarchy search
```

Even slower than Send Messages. Almost never use this.

### Pattern 3: Invoke Unity Events ✅ (Good)

PlayerInput exposes UnityEvents in the Inspector. You drag methods into slots manually.

C#

```
// PlayerInput Behavior: Invoke Unity Events

public class SpaceshipGameMode : MonoBehaviour
{
    // Wire these in the Inspector by dragging
    
    public void OnFirePrimaryPerformed(InputAction.CallbackContext ctx)
    {
        if (ctx.performed)
        {
            ShootMainWeapon();
        }
    }
    
    public void OnThrottlePerformed(InputAction.CallbackContext ctx)
    {
        if (ctx.performed)
        {
            float throttleAmount = ctx.ReadValue<float>();
            _currentThrottle = throttleAmount;
        }
    }
    
    public void OnScanForEnemiesPerformed(InputAction.CallbackContext ctx)
    {
        if (ctx.performed)
        {
            RadarSystem.instance.FullScan();
        }
    }
}
```

**In Inspector:**

Code

```
PlayerInput component
  └─ Events
       └─ Combat (Action Map)
            ├─ Fire Primary (performed): [SpaceshipGameMode] → OnFirePrimaryPerformed()
            ├─ Throttle (performed): [SpaceshipGameMode] → OnThrottlePerformed()
            └─ Scan For Enemies (performed): [SpaceshipGameMode] → OnScanForEnemiesPerformed()
```

**Pros:**

- Visual feedback — see what's wired in Inspector
- Full CallbackContext — all phase and value info available
- Type-safe — compile error if method deleted
- Clear at a glance — no magic method names

**Cons:**

- Inspector clutter — many actions = many event slots
- Requires manual wiring per action
- Hard to see from code who's handling what

**Best for:** Medium-sized projects, game modes, designers.

### Pattern 4: Invoke C# Events ✅ (Best)

Generate a C# class from your `.inputactions` asset. Subscribe directly in code.

C#

```
// First: In Input Actions asset Inspector, check "Generate C# Class"
// This generates: SpaceshipControlsInputActions.cs

public class SpaceshipController : MonoBehaviour
{
    private SpaceshipControlsInputActions _controls;

    void Awake()
    {
        _controls = new SpaceshipControlsInputActions();
        _controls.Enable();

        // Type-safe subscription — autocomplete works!
        _controls.Combat.FirePrimary.performed += OnFirePrimary;
        _controls.Combat.FireSecondary.performed += OnFireSecondary;
        _controls.Combat.Aim.performed += OnAim;
        _controls.Navigation.Throttle.performed += OnThrottle;
        _controls.Navigation.Rotate.performed += OnRotate;
    }

    void OnDisable()
    {
        // Always unsubscribe to prevent memory leaks
        _controls.Combat.FirePrimary.performed -= OnFirePrimary;
        _controls.Combat.FireSecondary.performed -= OnFireSecondary;
        _controls.Combat.Aim.performed -= OnAim;
        _controls.Navigation.Throttle.performed -= OnThrottle;
        _controls.Navigation.Rotate.performed -= OnRotate;
        _controls.Disable();
    }

    void OnFirePrimary(InputAction.CallbackContext ctx)
    {
        GameObject projectile = Instantiate(mainWeaponPrefab, firePoint.position, Quaternion.identity);
        projectile.GetComponent<Rigidbody>().velocity = fireDirection * bulletSpeed;
    }

    void OnFireSecondary(InputAction.CallbackContext ctx)
    {
        if (ctx.started)
            StartCharging();
        if (ctx.canceled)
            ReleaseCharge();
    }

    void OnAim(InputAction.CallbackContext ctx)
    {
        Vector2 aimInput = ctx.ReadValue<Vector2>();
        fireDirection = (transform.forward + (Vector3)aimInput * aimInfluence).normalized;
    }

    void OnThrottle(InputAction.CallbackContext ctx)
    {
        float throttle = ctx.ReadValue<float>();
        rb.velocity = transform.forward * (throttle * maxSpeed);
    }

    void OnRotate(InputAction.CallbackContext ctx)
    {
        Vector2 rotate = ctx.ReadValue<Vector2>();
        transform.Rotate(Vector3.up, rotate.x * rotationSpeed * Time.deltaTime);
    }
}
```

**Pros:**

- **Type-safe** — autocomplete works, typos are compile errors
- **Fast** — no reflection overhead
- **Full control** — subscribe/unsubscribe as needed
- **Clear** — no hidden Inspector wiring
- **Professional** — used in AAA games

**Cons:**

- Must regenerate class when asset changes
- More boilerplate code
- Less visual than Inspector wiring

**Best for:** Large projects, gameplay code, programming teams, production games.

### Comparison Table

|Pattern|Notification|Signature|Auto-wired|Phase Info|Best For|
|---|---|---|---|---|---|
|Send Messages|Reflection|`OnAction(InputValue)`|By name ✓|❌ Limited|5-min prototype|
|Broadcast Messages|Reflection|`OnAction(InputValue)`|By name ✓|❌ Limited|Never|
|Unity Events|Inspector drag|`OnAction(CallbackContext)`|Inspector ✓|✅ Full|Medium games|
|C# Events|Code subscribe|`OnAction(CallbackContext)`|Code ✓|✅ Full|Large games|

---

## Part 7: The InputManager Pattern

This is a **best practice architecture** used in professional projects. It centralizes all input handling into one singleton that exposes clean, high-level events.

### Why Use This Pattern?

**Problem:** If 10 scripts all subscribe directly to InputAction events, they must each:

- Store reference to the input system
- Subscribe in OnEnable / unsubscribe in OnDisable
- Know about action map switching
- Handle phase filtering (ctx.performed? ctx.started?)

**Solution:** One `InputManager` does all that. Other scripts just subscribe to simple events like `OnFirePrimary`.

### The Pattern — Complete InputManager

C#

```
using System;
using UnityEngine;
using UnityEngine.InputSystem;

public class InputManager : MonoBehaviour
{
    // ────────────────────────────────────────────────────────────
    // SINGLETON
    // ────────────────────────────────────────────────────────────
    
    public static InputManager instance { get; private set; }

    // ────────────────────────────────────────────────────────────
    // EVENTS — High-level, clean interface for other systems
    // ────────────────────────────────────────────────────────────
    
    // Combat
    public event Action OnFirePrimary;          // player shoots main weapon
    public event Action<float> OnFireSecondary; // player charges secondary (hold duration)
    public event Action<Vector2> OnAim;         // player aims (stick/mouse position)
    
    // Navigation
    public event Action<float> OnThrottle;      // engine speed (0-1)
    public event Action<Vector2> OnRotate;      // ship rotation (x=yaw, y=pitch)
    public event Action OnScanEnemies;          // player scans for hostile ships
    
    // Map switching (called by GameStateManager)
    public event Action<string> OnMapSwitched;  // fired when action map changes

    // ────────────────────────────────────────────────────────────
    // PRIVATE STATE
    // ────────────────────────────────────────────────────────────
    
    private PlayerInput _playerInput;
    private float _secondaryChargeStartTime;

    // ────────────────────────────────────────────────────────────
    // LIFECYCLE
    // ────────────────────────────────────────────────────────────

    void Awake()
    {
        // Singleton setup
        if (instance != null)
        {
            Destroy(gameObject);
            return;
        }
        instance = this;
        DontDestroyOnLoad(gameObject);

        // Get PlayerInput component and set behavior
        _playerInput = GetComponent<PlayerInput>();
        _playerInput.notificationBehavior = PlayerNotifications.InvokeUnityEvents;
    }

    // ────────────────────────────────────────────────────────────
    // CALLBACKS — Called by PlayerInput UnityEvents (wired in Inspector)
    // ────────────────────────────────────────────────────────────

    public void OnFirePrimaryPerformed(InputAction.CallbackContext ctx)
    {
        if (ctx.performed)
            OnFirePrimary?.Invoke();
    }

    public void OnFireSecondaryPerformed(InputAction.CallbackContext ctx)
    {
        if (ctx.started)
            _secondaryChargeStartTime = Time.time;
        
        if (ctx.canceled)
        {
            float chargeTime = Time.time - _secondaryChargeStartTime;
            OnFireSecondary?.Invoke(chargeTime);
        }
    }

    public void OnAimPerformed(InputAction.CallbackContext ctx)
    {
        if (ctx.performed)
        {
            Vector2 aimDir = ctx.ReadValue<Vector2>();
            OnAim?.Invoke(aimDir);
        }
    }

    public void OnThrottlePerformed(InputAction.CallbackContext ctx)
    {
        if (ctx.performed)
        {
            float throttleValue = ctx.ReadValue<float>();
            OnThrottle?.Invoke(throttleValue);
        }
    }

    public void OnRotatePerformed(InputAction.CallbackContext ctx)
    {
        if (ctx.performed)
        {
            Vector2 rotateInput = ctx.ReadValue<Vector2>();
            OnRotate?.Invoke(rotateInput);
        }
    }

    public void OnScanForEnemiesPerformed(InputAction.CallbackContext ctx)
    {
        if (ctx.performed)
            OnScanEnemies?.Invoke();
    }

    // ────────────────────────────────────────────────────────────
    // PUBLIC METHODS — For GameStateManager or menu systems
    // ────────────────────────────────────────────────────────────

    public void SwitchToMap(string mapName)
    {
        InputActionMap map = _playerInput.actions.FindActionMap(mapName);
        if (map == null)
        {
            Debug.LogWarning($"[InputManager] Map '{mapName}' not found in asset.");
            return;
        }
        _playerInput.SwitchCurrentActionMap(mapName);
        OnMapSwitched?.Invoke(mapName);
    }

    public bool IsMapActive(string mapName)
    {
        return _playerInput.currentActionMap.name == mapName;
    }

    public void PauseInput() => _playerInput.DeactivateInput();
    public void ResumeInput() => _playerInput.ActivateInput();
}
```

### Using the InputManager

Any script can now react to input cleanly without knowing about the Input System:

C#

```
public class WeaponSystem : MonoBehaviour
{
    [SerializeField] private GameObject mainProjectilePrefab;
    [SerializeField] private Transform firePoint;
    [SerializeField] private float mainBulletSpeed = 50f;

    void OnEnable()
    {
        // Subscribe to clean events — no Input System knowledge
        InputManager.instance.OnFirePrimary += ShootMainWeapon;
        InputManager.instance.OnFireSecondary += ShootSecondaryWeapon;
    }

    void OnDisable()
    {
        // ALWAYS unsubscribe
        InputManager.instance.OnFirePrimary -= ShootMainWeapon;
        InputManager.instance.OnFireSecondary -= ShootSecondaryWeapon;
    }

    void ShootMainWeapon()
    {
        GameObject projectile = Instantiate(mainProjectilePrefab, firePoint.position, Quaternion.identity);
        projectile.GetComponent<Rigidbody>().velocity = firePoint.forward * mainBulletSpeed;
    }

    void ShootSecondaryWeapon(float chargeTime)
    {
        float damage = Mathf.Lerp(50f, 500f, chargeTime / 3f); // charges up to 3 seconds
        Debug.Log($"Secondary weapon fired! Charge: {chargeTime:F2}s, Damage: {damage:F0}");
    }
}
```

Another example:

C#

```
public class ShipController : MonoBehaviour
{
    [SerializeField] private Rigidbody rb;
    [SerializeField] private float maxSpeed = 100f;
    [SerializeField] private float rotationSpeed = 45f;

    private Vector2 _currentRotation;
    private float _currentThrottle;

    void OnEnable()
    {
        InputManager.instance.OnThrottle += UpdateThrottle;
        InputManager.instance.OnRotate += UpdateRotation;
    }

    void OnDisable()
    {
        InputManager.instance.OnThrottle -= UpdateThrottle;
        InputManager.instance.OnRotate -= UpdateRotation;
    }

    void Update()
    {
        // Apply throttle every frame
        float speed = _currentThrottle * maxSpeed;
        rb.velocity = transform.forward * speed;

        // Apply rotation every frame
        transform.Rotate(Vector3.up, _currentRotation.x * rotationSpeed * Time.deltaTime);
        transform.Rotate(Vector3.right, _currentRotation.y * rotationSpeed * Time.deltaTime);
    }

    void UpdateThrottle(float value) => _currentThrottle = value;
    void UpdateRotation(Vector2 value) => _currentRotation = value;
}
```

### Benefits of This Pattern

- **Decoupling** — WeaponSystem doesn't know about InputSystem
- **Single responsibility** — InputManager handles input, systems handle gameplay
- **Easy map switching** — One place to control what's active
- **Reusability** — Use InputManager in any scene
- **Testability** — Can mock events for testing
- **Scalability** — Adding 50 more systems doesn't complicate input

---

## Part 8: Action Map Switching

Only one action map should be active at a time. Switching them lets you change what inputs are available based on game state.

### Example: Spaceship Game States

Code

```
Game State: SpaceExploration → Navigation map (Throttle, Rotate, Scan)
Game State: InCombat         → Combat map (FirePrimary, FireSecondary, Aim)
Game State: Docking          → Docking map (AlignShip, RequestEntry, CancelDocking)
Game State: PauseMenu        → UI map (Navigate, Confirm, Cancel, Inventory)
```

### GameStateManager Example

C#

```
public enum GameState { Exploring, Combat, Docking, Paused, Menu }

public class GameStateManager : MonoBehaviour
{
    public static GameStateManager instance { get; private set; }
    
    public event Action<GameState> OnStateChanged;
    
    private GameState _currentState;

    void Awake()
    {
        if (instance != null) { Destroy(gameObject); return; }
        instance = this;
    }

    public void SetState(GameState newState)
    {
        if (_currentState == newState) return;
        
        _currentState = newState;
        OnStateChanged?.Invoke(newState);

        // Switch input maps based on state
        switch (newState)
        {
            case GameState.Exploring:
                InputManager.instance.SwitchToMap("Navigation");
                Time.timeScale = 1f;
                Debug.Log("🚀 Exploring space");
                break;

            case GameState.Combat:
                InputManager.instance.SwitchToMap("Combat");
                Time.timeScale = 1f;
                Debug.Log("⚔️ Combat started!");
                break;

            case GameState.Docking:
                InputManager.instance.SwitchToMap("Docking");
                Time.timeScale = 0.1f; // slow for precision
                Debug.Log("🔧 Docking sequence");
                break;

            case GameState.Paused:
            case GameState.Menu:
                InputManager.instance.SwitchToMap("UI");
                Time.timeScale = 0f;
                Debug.Log("⏸️ Paused");
                break;
        }
    }

    public GameState GetCurrentState() => _currentState;
}
```

### Subscribe to State Changes

C#

```
public class UIManager : MonoBehaviour
{
    [SerializeField] private GameObject combatHUD;
    [SerializeField] private GameObject navigationHUD;
    [SerializeField] private GameObject pauseMenu;

    void Start()
    {
        GameStateManager.instance.OnStateChanged += HandleStateChange;
    }

    void HandleStateChange(GameState newState)
    {
        // Hide all HUDs
        combatHUD.SetActive(false);
        navigationHUD.SetActive(false);
        pauseMenu.SetActive(false);

        // Show appropriate HUD
        switch (newState)
        {
            case GameState.Exploring:
                navigationHUD.SetActive(true);
                break;
            case GameState.Combat:
                combatHUD.SetActive(true);
                break;
            case GameState.Paused:
                pauseMenu.SetActive(true);
                break;
        }
    }
}
```

---

## Part 9: Real Code Walkthrough - Spaceship Game

### Full Example: Complete Spaceship Combat

**Step 1: Create the Input Actions Asset**

In Unity Editor:

1. Right-click in Assets → Create → Input Actions
2. Name it: `SpaceshipControls`
3. Define maps and actions:

Code

```
SpaceshipControls.inputactions
├── Combat (Action Map)
│   ├── FirePrimary (Button)
│   │   ├── Keyboard: W
│   │   └── Gamepad: Right Trigger
│   ├── FireSecondary (Button)
│   │   ├── Keyboard: Q
│   │   └── Gamepad: Left Trigger
│   └── Aim (Value - Vector2)
│       ├── Keyboard: Mouse
│       └── Gamepad: Right Stick
│
└── Navigation (Action Map)
    ├── Throttle (Value - Float)
    │   ├── Keyboard: Up/Down arrows
    │   └── Gamepad: Left Stick Y
    ├── Rotate (Value - Vector2)
    │   ├── Keyboard: A/D keys
    │   └── Gamepad: Left Stick
    └── Scan (Button)
        ├── Keyboard: Space
        └── Gamepad: Start
```

**Step 2: InputManager.cs**

C#

```
using System;
using UnityEngine;
using UnityEngine.InputSystem;

public class InputManager : MonoBehaviour
{
    public static InputManager instance { get; private set; }

    public event Action OnFirePrimary;
    public event Action<float> OnFireSecondary;
    public event Action<Vector2> OnAim;
    public event Action<float> OnThrottle;
    public event Action<Vector2> OnRotate;
    public event Action OnScanRadar;

    private PlayerInput _playerInput;
    private float _chargeStart;

    void Awake()
    {
        if (instance != null) { Destroy(gameObject); return; }
        instance = this;
        DontDestroyOnLoad(gameObject);

        _playerInput = GetComponent<PlayerInput>();
        _playerInput.notificationBehavior = PlayerNotifications.InvokeUnityEvents;
    }

    // Combat Map Callbacks
    public void OnFirePrimaryPerformed(InputAction.CallbackContext ctx) 
        => OnFirePrimary?.Invoke();

    public void OnFireSecondaryPerformed(InputAction.CallbackContext ctx)
    {
        if (ctx.started) _chargeStart = Time.time;
        if (ctx.canceled) OnFireSecondary?.Invoke(Time.time - _chargeStart);
    }

    public void OnAimPerformed(InputAction.CallbackContext ctx)
        => OnAim?.Invoke(ctx.ReadValue<Vector2>());

    // Navigation Map Callbacks
    public void OnThrottlePerformed(InputAction.CallbackContext ctx)
        => OnThrottle?.Invoke(ctx.ReadValue<float>());

    public void OnRotatePerformed(InputAction.CallbackContext ctx)
        => OnRotate?.Invoke(ctx.ReadValue<Vector2>());

    public void OnScanRadarPerformed(InputAction.CallbackContext ctx)
        => OnScanRadar?.Invoke();

    public void SwitchMap(string mapName)
    {
        if (_playerInput.actions.FindActionMap(mapName) != null)
            _playerInput.SwitchCurrentActionMap(mapName);
    }
}
```

**Step 3: WeaponSystem.cs**

C#

```
using UnityEngine;

public class WeaponSystem : MonoBehaviour
{
    [SerializeField] private Transform firePoint;
    [SerializeField] private GameObject projectilePrefab;
    [SerializeField] private float bulletSpeed = 50f;
    [SerializeField] private float maxSecondaryDamage = 500f;

    void OnEnable()
    {
        InputManager.instance.OnFirePrimary += FireMain;
        InputManager.instance.OnFireSecondary += FireSecondary;
    }

    void OnDisable()
    {
        InputManager.instance.OnFirePrimary -= FireMain;
        InputManager.instance.OnFireSecondary -= FireSecondary;
    }

    void FireMain()
    {
        var projectile = Instantiate(
            projectilePrefab,
            firePoint.position,
            firePoint.rotation
        );
        
        projectile.GetComponent<Rigidbody>().velocity = firePoint.forward * bulletSpeed;
        
        Debug.Log("💥 Main weapon fired!");
    }

    void FireSecondary(float chargeTime)
    {
        float chargePercent = Mathf.Clamp01(chargeTime / 2f); // max 2 seconds
        float damage = Mathf.Lerp(50f, maxSecondaryDamage, chargePercent);
        
        Debug.Log($"⚡ Secondary weapon! Charge: {chargePercent * 100:F0}% Damage: {damage:F0}");
        
        // Spawn charged effect at firepoint
        Instantiate(chargeEffectPrefab, firePoint.position, Quaternion.identity);
    }
}
```

**Step 4: ShipController.cs**

C#

```
using UnityEngine;

public class ShipController : MonoBehaviour
{
    [SerializeField] private Rigidbody rb;
    [SerializeField] private float maxSpeed = 100f;
    [SerializeField] private float rotationSpeed = 45f;

    private Vector2 _moveInput;
    private Vector2 _rotateInput;
    private float _throttleInput;

    void OnEnable()
    {
        InputManager.instance.OnThrottle += SetThrottle;
        InputManager.instance.OnRotate += SetRotation;
        InputManager.instance.OnAim += SetAimDirection;
    }

    void OnDisable()
    {
        InputManager.instance.OnThrottle -= SetThrottle;
        InputManager.instance.OnRotate -= SetRotation;
        InputManager.instance.OnAim -= SetAimDirection;
    }

    void Update()
    {
        // Apply thrust
        float speed = _throttleInput * maxSpeed;
        rb.velocity = transform.forward * speed;

        // Apply rotation
        transform.Rotate(Vector3.up, _rotateInput.x * rotationSpeed * Time.deltaTime);
        transform.Rotate(Vector3.right, -_rotateInput.y * rotationSpeed * Time.deltaTime);
    }

    void SetThrottle(float value) => _throttleInput = Mathf.Clamp01(value);
    void SetRotation(Vector2 value) => _rotateInput = value;
    void SetAimDirection(Vector2 value) => _moveInput = value;
}
```

**Step 5: RadarSystem.cs**

C#

```
using UnityEngine;

public class RadarSystem : MonoBehaviour
{
    [SerializeField] private float scanRange = 500f;
    [SerializeField] private LayerMask enemyLayer;

    void OnEnable()
    {
        InputManager.instance.OnScanRadar += PerformScan;
    }

    void OnDisable()
    {
        InputManager.instance.OnScanRadar -= PerformScan;
    }

    void PerformScan()
    {
        Collider[] enemies = Physics.OverlapSphere(transform.position, scanRange, enemyLayer);
        
        Debug.Log($"🔍 Radar scan complete! Found {enemies.Length} targets.");
        
        foreach (var enemy in enemies)
        {
            Debug.Log($"  - Enemy detected: {enemy.name} at {Vector3.Distance(transform.position, enemy.transform.position):F0}m");
        }
    }
}
```

**Step 6: Inspector Setup**

1. Create GameObject "GameManager"
2. Add PlayerInput component
3. Assign SpaceshipControls asset
4. Set Behavior to "Invoke Unity Events"
5. Wire up all methods:

Code

```
PlayerInput Events
├─ Combat Map
│  ├─ Fire Primary → InputManager.OnFirePrimaryPerformed()
│  ├─ Fire Secondary → InputManager.OnFireSecondaryPerformed()
│  └─ Aim → InputManager.OnAimPerformed()
└─ Navigation Map
   ├─ Throttle → InputManager.OnThrottlePerformed()
   ├─ Rotate → InputManager.OnRotatePerformed()
   └─ Scan Radar → InputManager.OnScanRadarPerformed()
```

### Complete Data Flow

Code

```
Player presses W (or Gamepad RB)
    ↓
Input System detects Keyboard/w or Gamepad/triggerRight
    ↓
Input System finds "FirePrimary" action
    ↓
FirePrimary action fires: performed phase
    ↓
PlayerInput calls: InputManager.OnFirePrimaryPerformed(ctx)
    ↓
InputManager.OnFirePrimary event fires
    ↓
All subscribers called:
  - WeaponSystem.FireMain()
    ↓
    GameObject projectile created at firePoint
    Velocity set to forward * bulletSpeed
    Debug log: "💥 Main weapon fired!"
```

---

## Part 10: Advanced Topics

### Composite Bindings: Multiple Keys as One Input

Code

```
Navigate action (Value - Vector2)
  ├─ Composite: 2D Vector
  │   ├─ Up: W
  │   ├─ Down: S
  │   ├─ Left: A
  │   └─ Right: D
  │
  └─ Binding: Gamepad Left Stick
```

The Input System automatically converts WASD to a Vector2:

- W = `(0, 1)`
- A = `(-1, 0)`
- W + D = `(0.707, 0.707)` (normalized diagonal)
- Stick automatically outputs Vector2

**Code (same for both):**

C#

```
void OnNavigate(InputAction.CallbackContext ctx)
{
    Vector2 direction = ctx.ReadValue<Vector2>();
    
    // Works for both WASD and gamepad stick
    rb.velocity = direction * speed;
}
```

### Runtime Rebinding: Let Players Change Controls

C#

```
public class RebindingUI : MonoBehaviour
{
    public void StartRebinding(string actionName)
    {
        InputAction action = InputManager.instance.GetAction(actionName);
        
        var rebind = action.PerformInteractiveRebinding()
            .OnComplete(operation =>
            {
                string newPath = operation.action.bindings[0].effectivePath;
                Debug.Log($"{actionName} rebound to: {newPath}");
                
                // Save to PlayerPrefs
                PlayerPrefs.SetString($"Binding_{actionName}", newPath);
                
                operation.Dispose();
            })
            .Start();
    }
}
```

### Checking Current Input State

C#

```
// Is Fire button currently held?
if (InputManager.instance.IsFirePressed())
    Debug.Log("Holding fire");

// Get current rotation value
Vector2 currentRotate = InputManager.instance.GetRotationInput();

// Check which map is active
if (InputManager.instance.IsMapActive("Combat"))
    Debug.Log("In combat mode");
```

### Multiple Players (Multiplayer)

C#

```
// Each player gets its own PlayerInput
public class MultiplayerSetup : MonoBehaviour
{
    void Start()
    {
        // Player 1
        var player1Prefab = Instantiate(playerPrefab);
        player1Prefab.GetComponent<PlayerInput>().playerIndex = 0;

        // Player 2
        var player2Prefab = Instantiate(playerPrefab);
        player2Prefab.GetComponent<PlayerInput>().playerIndex = 1;
    }
}
```

---

## Part 11: Quick Reference

### Setup Checklist

Code

```
☐ Install Input System from Package Manager
☐ Create .inputactions asset (SpaceshipControls.inputactions)
☐ Define Action Maps (Combat, Navigation, UI)
☐ Define Actions in each map (FirePrimary, Throttle, etc.)
☐ Create Bindings (W, Gamepad RB, Mouse, etc.)
☐ Generate C# class (or skip if using PlayerInput)
☐ Create InputManager.cs
☐ Add PlayerInput to GameObject
☐ Assign .inputactions asset to PlayerInput
☐ Set Default Map (Navigation, Combat, etc.)
☐ Set Behavior to "Invoke Unity Events"
☐ Wire up methods in Inspector
☐ Create game systems that subscribe to InputManager events
```

### Quick Template

C#

```
// 1. Create InputManager singleton
public class InputManager : MonoBehaviour
{
    public static InputManager instance { get; private set; }
    
    public event Action OnFirePrimary;
    public event Action<float> OnThrottle;
    
    // Wire these in Inspector
    public void OnFirePrimaryPerformed(InputAction.CallbackContext ctx)
        => OnFirePrimary?.Invoke();
}

// 2. Subscribe in game systems
public class MySystem : MonoBehaviour
{
    void OnEnable()
        => InputManager.instance.OnFirePrimary += DoSomething;
    
    void OnDisable()
        => InputManager.instance.OnFirePrimary -= DoSomething;
    
    void DoSomething() { /* gameplay */ }
}
```

### Binding Paths

Code

```
<Keyboard>/space              Spacebar
<Keyboard>/w                  W key
<Keyboard>/a                  A key
<Mouse>/delta                 Mouse movement (Vector2)
<Mouse>/leftButton            Left click
<Gamepad>/triggerRight        Right trigger (0-1)
<Gamepad>/triggerLeft         Left trigger (0-1)
<Gamepad>/rightStick          Right stick (Vector2)
<Gamepad>/buttonSouth         A/X button
<Gamepad>/buttonNorth         Y/Triangle button
<Gamepad>/dpad/up             D-pad up
<Touchscreen>/primaryTouch    Touch position (Vector2)
```

### Decision Tree

**Should I use InputManager pattern?**

Code

```
Game size:
  ├─ Solo project (1-2 scripts)        → Optional, can subscribe directly
  ├─ Medium (5-10 scripts)             → Recommended
  ├─ Large (20+ scripts)               → Required
  └─ Team/production                   → Required
```

**Which notification pattern?**

Code

```
Requirements:
  ├─ Need all phase info (started/performed/canceled)  → UnityEvents or C# Events
  ├─ Want visual clarity in Inspector                   → UnityEvents
  ├─ Maximum code control                               → C# Events
  ├─ 5-minute prototype                                 → SendMessages
  └─ Professional game                                  → InputManager + UnityEvents
```

### Troubleshooting

|Problem|Cause|Fix|
|---|---|---|
|Action never fires|Wrong action map active|Call `SwitchToMap()`|
|Callbacks not called|Not wired in Inspector|Drag InputManager into event slots|
|Memory leak on scene change|Not unsubscribing|Add unsubscribe in OnDisable|
|Typo in method name (SendMessages)|Silent failure|Check method name: `On<ActionName>`|
|Gamepad doesn't work|No gamepad binding|Add binding: `<Gamepad>/triggerRight`|
|Value always 0|Wrong action type|Ensure action is "Value" not "Button"|
|Can't rebind|Need runtime rebinding code|Use `PerformInteractiveRebinding()`|

### Common Mistakes

❌ **Don't:**

C#

```
// Don't subscribe in Update
void Update()
{
    InputManager.instance.OnFire += DoThing;  // BAD: subscribes every frame!
}

// Don't forget to unsubscribe
void OnDisable() { } // BAD: memory leak!

// Don't access Input System directly if using InputManager
if (Input.GetKeyDown(KeyCode.W))  // BAD: bypasses InputManager
    MoveForward();
```

✅ **Do:**

C#

```
// Subscribe in OnEnable, unsubscribe in OnDisable
void OnEnable()
    => InputManager.instance.OnFire += DoThing;

void OnDisable()
    => InputManager.instance.OnFire -= DoThing;

// Use InputManager events consistently
InputManager.instance.OnThrottle += ApplyThrust;
```

---

## Summary

The New Input System provides:

- **Flexible, rebindable controls** via `.inputactions` assets
- **Event-driven architecture** instead of polling Input.GetKey()
- **Multi-device support** (keyboard, gamepad, touch, VR)
- **Clean separation** between input logic and gameplay logic

**Best Practice: InputManager Pattern**

- One singleton manages all input
- Exposes high-level C# events
- Other scripts subscribe to clean events
- Decoupled, testable, professional, scalable

**Key Insight:** Stop thinking about **keys and buttons**. Think about **player intentions**: Fire, Thrust, Rotate, Scan. Map those to controls in an asset, not code. Let the Input System handle the complexity of devices and rebinding.

---

**End of Complete Guide**


```
SO COVER ALL THE WAYS IN WHICH WE CONNECT THE InputActionMap to the code and use the Action Events and attach methods to it
Change the code form the Invoke Unity Events to Invoke C Sharp Events
```