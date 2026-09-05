# How Does Unity Work?

Unity is built on **four foundational pillars** that work together. Two of them are _hierarchies_ (structural, how your project is organized), and two of them are _support systems_ (functional, how your code actually gets things done and gets saved correctly).

| Pillar          | Unity Concept            | What it actually is                                                              |
| --------------- | ------------------------ | -------------------------------------------------------------------------------- |
| **Composition** | GameObjects / Components | The physical things that exist _in_ your scene.                                  |
| **Inheritance** | MonoBehaviour            | The DNA that lets your scripts exist in the scene at all.                        |
| **Global API**  | `Time`, `Input`, etc.    | The static tools and environment your scripts reach out to use.                  |
| **Attributes**  | `[SerializeField]`, etc. | The instructions that tell the Unity _Editor_ how to display and save your code. |

The first two (Composition + Inheritance) answer "**what exists, and how is it built**." The second two (Global API + Attributes) answer "**how does that thing talk to the engine, and how does the Editor understand it**." All four are running simultaneously the moment you hit Play.

---

## 1. Composition (GameObjects & Components) : "Has-a"

This is the **Composition Hierarchy**: how things are physically built in the Scene.

- **GameObject**: Everything in a game is a GameObject : lights, the player, sound sources, bullets, particles, all of it. Think of it as an empty cardboard box. By itself it does nothing — no shape, no physics, no behavior. It's just a container that holds a position, rotation, and scale in the world.
- **Component**: These are the pieces that actually _do_ something. A Component attaches to a GameObject and gives that empty box meaning — a `MeshRenderer` gives it a visible shape, a `Rigidbody` gives it physics, a `BoxCollider` gives it a hitbox.
- **Script**: A custom Component you write yourself, when the built-in ones don't do what you need.

Composition is "has-a" thinking:

- A GameObject **has-a** `PlayerController`.
- A GameObject **has-a** `Rigidbody`.
- A GameObject **has-a** `BoxCollider`.

None of these Components care about each other's internal code — they just sit side-by-side on the same container. That's composition: building complex behavior by _stacking_ simple, independent parts onto an object, rather than writing one giant class that does everything.

---

## 2. Inheritance (MonoBehaviour) — "Is-a"

This is the **Class Hierarchy**: how the C# code itself is written, and how a plain script becomes something Unity can actually attach to a GameObject.

Inheritance is "is-a" thinking:

- A `PlayerController` **is-a** `MonoBehaviour`.
- A `MonoBehaviour` **is-a** `Component`.
- A `Component` **is-a** `Object`.

### The Family Tree

In order of inheritance (each one inherits from the one before it):

1. **Object** — the base of literally everything in Unity.
2. **GameObject & Component** — both inherit from Object, but they're _siblings_, not parent/child of each other. One is the container, the other is the content.
3. **Behaviour** — adds the ability to be enabled/disabled.
4. **MonoBehaviour** — the important one. It adds the "magic methods" (`Start()`, `Update()`, etc.) that connect your script to the game loop, and it's what turns a plain C# class into something Unity's engine can actually run.

This hierarchy is what _gives your script the capability to exist_ inside Unity in the first place — it's the "DNA."

### Connecting to the Engine (the Game Loop)

Unity doesn't read your code once and stop — it runs a **Game Loop**. Think of a flipbook: to fake movement, Unity draws a picture, then another, then another, very fast (usually 60 times a second). Each picture is a **Frame**.

Your MonoBehaviour hooks into this loop via magic methods:

- **`Start()`** — runs once, when the object first comes into existence (or the scene starts). This is where you do setup, like grabbing a reference to that Rigidbody.
- **`Update()`** — runs every single frame. At 60 fps, that's 60 times a second. This is where you read input and move things.

When you enter Play mode, Unity scans every Component on every GameObject in the scene, and _that's_ the moment it starts wiring everything into the Game Engine.

### Native vs. Managed — the actual bridge

Unity's engine core is written in **C++** (native). You write in **C#**, which runs on .NET (Mono/IL2CPP) — the managed world. These two worlds live in separate memory spaces and can't talk to each other directly. **MonoBehaviour is the bridge.**

When you inherit from MonoBehaviour (and eventually `UnityEngine.Object`), your script inherits a hidden variable, often called `m_CachedPtr` (an `IntPtr`). Here's what happens on Play:

1. **Creation** : Unity (C++) creates the actual component in native memory.
2. **Instance** : Unity creates an instance of your C# class.
3. **The Link** : Unity stores the memory address of the native C++ object inside that `m_CachedPtr` field on your C# object.

Without inheriting MonoBehaviour, your C# class just floats in .NET memory with no link to the C++ game world at all.

### How `Update()` Actually Gets Called

You never write `override void Update()` — just `void Update()`. So how does C++ call a C# method it has no idea exists?

**Reflection**, at scan time:

1. **Scanning** : when the scene loads, Unity inspects your class via Reflection, literally searching for method names like `"Start"`, `"Update"`, `"OnCollisionEnter"`.
2. **Caching** : if it finds a method named `"Update"`, it saves a pointer to that function in an internal C++ list.
3. **Execution** : every frame, the C++ engine walks that list and fires the saved function pointers. It's not re-reading your file each time.

This is also a performance detail worth knowing: if you inherit MonoBehaviour but never write an `Update()`, Unity sees that during the scan and simply never adds you to the list, so you pay zero per-frame cost for a method you don't have.

---

## 3. Global API (Time, Input, etc.) — the tools you reach for

This is the piece that sits _outside_ both hierarchies above. Composition and Inheritance are about **your** objects and **your** scripts. The Global API is Unity's way of exposing **engine-wide state** that doesn't belong to any single GameObject : things like `Time`, `Input`, `Physics`, `Screen`, `Application`, `Camera.main`, and `Debug`.

- **`Time.deltaTime`** — the time in seconds since the last frame. Used to make movement frame-rate independent.
- **`Time.time`** — total elapsed time since the game started.
- **`Input.GetKeyDown(KeyCode.Space)`** / **`Input.GetAxis("Horizontal")`** — reads the player's keyboard/controller state this frame.
- **`Physics.Raycast(...)`** — asks the physics engine a one-off question ("is there anything between point A and B?") without needing a Component reference.
- **`Screen.width` / `Screen.height`** — current window/display resolution.

**Why this is a separate pillar and not part of Composition:** GameObjects and Components are _instance-based_ — to use one, you need an actual reference to that specific object in the scene (`myRigidbody.AddForce(...)`). The Global API classes are **static** — you can call `Time.deltaTime` or `Input.GetKeyDown(...)` from _any_ script, at _any_ time, with zero reference needed. They're not "things in the scene," they're the shared environment every script reaches into.

Under the hood, the same native/managed split from Section 2 still applies: many of these static calls are thin C# wrappers that make a native call into the C++ engine every time you access them (e.g., `Time.deltaTime` isn't a stored number sitting in your script — it's a property that queries the engine's internal clock fresh each time you read it). It's a simpler, non-per-instance version of the same bridging idea as MonoBehaviour's `m_CachedPtr`.

---

## 4. Attributes (`[SerializeField]`, etc.) — instructions for the Editor

Attributes are metadata tags written in square brackets directly above a field, method, or class — `[SerializeField]`, `[Header("Movement")]`, `[Range(0, 10)]`, `[Tooltip("...")]`, `[HideInInspector]`.

**Critically: attributes don't run any logic on their own.** They don't execute like a method call. The C# compiler just attaches them as metadata onto the compiled assembly. Unity's Editor then reads that metadata using the _same Reflection mechanism_ it uses to find `Update()` and `Start()` — except this time it's asking "how should I draw this field in the Inspector, and should I save it into the scene/prefab file?"

- **`[SerializeField]`** specifically: by default, Unity's serializer only picks up **public** fields to show in the Inspector and save to disk. 
	- Making a field public just to see it in the Inspector breaks encapsulation — anyone can edit it from any other script. `[SerializeField]` fixes that: it tells the serializer "treat this **private** field as if it were public, for saving/displaying purposes only." You get to keep the field private in code while Unity still shows and stores its value.
- **`[Header("...")]` / `[Tooltip("...")]`**:
	- purely cosmetic, Inspector-only labeling/UI. No effect on runtime behavior.
- **`[Range(min, max)]`**
	- turns a number field into a slider in the Inspector, and clamps the value.
- **`[HideInInspector]`** 
	- the opposite of `[SerializeField]`: keeps a field serialized (saved) but hides it from the Inspector UI.
- **Worth flagging as the obvious exception**: not every attribute is purely cosmetic/Editor-only. `[RequireComponent(typeof(Rigidbody))]`, for example, actually changes runtime behavior — it forces Unity to auto-add a Rigidbody the moment your script is attached to a GameObject, and blocks you from removing that Rigidbody afterward. Most attributes are "Editor instructions," but this one enforces a real dependency.

**How this connects to the other three pillars:** Attributes are the layer that governs how data living on your Components (Composition) gets displayed and saved by the Editor, completely separate from what MonoBehaviour (Inheritance) does. MonoBehaviour gives your script the _capability_ to run in the engine; attributes just control how the Editor _presents and stores_ the data sitting inside it.

---

## Putting It All Together

The moment you press Play:

1. Unity scans the **Class Hierarchy** via Reflection to find every script inheriting MonoBehaviour, and caches pointers to their magic methods (`Start`, `Update`, etc.) : **Inheritance**.
2. It looks at which Components are attached to which GameObjects in the scene : **Composition**.
3. Before that, back in the Editor, it already used **Attributes** to decide which private fields to serialize/display and what values got saved into the scene file.
4. While the game loop runs, your scripts reach out to the **Global API** (`Time`, `Input`, `Physics`, etc.) for engine-wide state that doesn't belong to any single object.

All four pillars run in parallel, all the time — they're not phases, they're different _axes_ of the same running engine.



# [[MonoBehaviour in Unity]]
(Some references which we can use: https://omitram.com/the-unity-script-lifecycle-a-beginners-guide-to-execution-order/)

In Unity, **`MonoBehaviour`** is the base class from which every script derives by default. If you want a C# script to be attached to a GameObject and interact with the game world, it must inherit from `MonoBehaviour`. Think of it as the bridge between your custom C# logic and Unity’s internal C++ engine. Instead of writing your own massive `while(true)` game loop, `MonoBehaviour` gives your script a lifecycle—Unity automatically calls specific methods inside your script at specific times.

## The Lifecycle (Order of Execution)

![[Pasted image 20260808000452.png|1284]]

A script's lifecycle dictates exactly when your code runs. Misunderstanding this order is the most common source of bugs for beginners.

| **Method**          | **When it runs**                                                                       | **Best used for...**                                                                 |
| ------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| **`Awake()`**       | Once, the moment the GameObject is created (even if the script component is disabled). | Initializing variables and grabbing references (e.g., `GetComponent`).               |
| **`OnEnable()`**    | Every time the script or object is turned on.                                          | Subscribing to C# events or resetting player stats upon respawn.                     |
| **`Start()`**       | Once, right before the first frame, but **only** if the script is enabled.             | Passing data to other objects (since you know their `Awake` has already finished).   |
| **`FixedUpdate()`** | On a fixed timer (independent of frame rate).                                          | **Physics only.** Applying forces to Rigidbodies or moving physics-based characters. |
| **`Update()`**      | Once per frame (speed varies based on FPS).                                            | Reading player input, simple timers, and general game logic.                         |
| **`LateUpdate()`**  | Once per frame, _after_ all `Update()` calls finish.                                   | Camera follow logic. (Ensures the player has fully moved before the camera adjusts). |
| **`OnDisable()`**   | When the object is disabled or destroyed.                                              | Unsubscribing from events to prevent memory leaks.                                   |
| **`OnDestroy()`**   | Right before the GameObject is deleted.                                                | Final cleanup of generated materials or background processes.                        |

## Unity 6 Modern Nuances

While the core lifecycle has remained stable for years, Unity 6 introduces and refines several properties to support modern C# programming standards:

- **Async/Await Support (`destroyCancellationToken`):** Historically, Unity developers used Coroutines to pause code over time. Unity 6 heavily supports modern `async/await` tasks (via `Awaitable`). `MonoBehaviour` now natively includes a `destroyCancellationToken` property. You can pass this token into async tasks so they automatically cancel if the GameObject is destroyed—preventing nasty background errors and memory leaks    

- **State Tracking (`didAwake`, `didStart`):** Unity 6 exposes `didAwake` and `didStart` boolean properties directly on the component. This saves you from having to write custom private booleans just to check if an object has fully initialized before another script tries to interact with it.

- **Avoid the Null Check Trap:** When a `MonoBehaviour` is destroyed, the C# object actually remains in memory until garbage collection occurs, but Unity overrides the `==` operator to act as if it is `null`. Because of this custom engine quirk, you cannot use modern C# null-conditional operators (like `?.` or `??`) when checking if a `MonoBehaviour` exists. You must use standard `if (myScript != null)`.

## When NOT to use MonoBehaviour
A common beginner mistake is making _every_ script a `MonoBehaviour`. Inheriting from it comes with a slight performance overhead because Unity has to hook it into the engine's callback loop.

**Only use it if:**
1. The script needs to be attached to a GameObject in the scene.
2. You need access to lifecycle events (`Update`, `OnCollisionEnter`).
3. You need to use Coroutines or access the `Transform`.

**Drop it and use a plain C# class if:**
1. You are just storing data (e.g., an inventory item's stats). For Unity-specific data storage, use `ScriptableObject` instead.
2. You are writing pure math, utility, or network logic that operates independently of objects in the scene.

# [[Composition (Components) in Unity]]


# [[Global APIs in Unity]]



# [[Attributes in Unity]]
## `[SerializableField]`
`[SerializeField]` is an **attribute** that forces Unity to serialize a private field, making it visible and editable in the Inspector.


# [[Properties in Unity Hierarchy Inheritance]]
# [[Methods in Unity Hierarchy Inheritance]]


---
---



## Events in Unity

Events are a **communication system** in Unity that allows different game objects and scripts to notify each other when something happens, without directly depending on one another.

---

## **What Are Events?**

Events follow the **Observer Pattern**:

- **Publisher**: Fires/raises the event when something happens
- **Subscribers**: Listen for the event and execute their own code in response

This creates **decoupled architecture** — components don't need to know about each other directly.

---

## **Unity Hierarchy Context**

Before diving into events, understand where they fit in Unity's structure:

Code

```
GameObject
├── Transform
├── Components (Scripts, Collider, Renderer, etc.)
│   └── Events are defined and raised WITHIN components/scripts
├── Child Objects
└── ...
```

**Key Points:**

- **Objects/GameObjects**: Containers that hold components
- **Components**: Scripts attached to GameObjects that contain logic (including events)
- **Events**: Communication mechanism between components

When a component raises an event, **other components (on the same or different GameObjects) can respond**.

## Built-In Delegates in Unity

### 1) Action


# Classes in `UnityEngine` namespace
## Rigidbody
Add physics in unity to the gameObject. 

### Attributes

| Syntax               | Type  | Working                                                                                          |
| -------------------- | ----- | ------------------------------------------------------------------------------------------------ |
| `rb.angularVelocity` | float | **property** (not a method) that controls rotational speed of a Rigidbody in radians per second. |


### Method:

#### AddRelativeForce()
`AddRelativeForce` applies force to a Rigidbody in its **local coordinate system** instead of world space.[^1][^2]

##### Syntax
```csharp
// Vector3 version
rb.AddRelativeForce(Vector3 force);
rb.AddRelativeForce(Vector3 force, ForceMode mode);

// Individual components
rb.AddRelativeForce(float x, float y, float z);
rb.AddRelativeForce(float x, float y, float z, ForceMode mode);
```

##### Example - Rocket/Vehicle Movement
```csharp
using UnityEngine;

public class RocketMovement : MonoBehaviour 
{
    public float thrust = 10f;
    private Rigidbody rb;
    
    void Start() 
    {
        rb = GetComponent<Rigidbody>();
    }
    
    void FixedUpdate() 
    {
        // Push forward in local space (always moves in object's forward direction)
        rb.AddRelativeForce(Vector3.forward * thrust);
        
        // Push upward in local space
        rb.AddRelativeForce(Vector3.up * thrust);
        
        // Or use individual components
        rb.AddRelativeForce(0, 0, thrust); // x, y, z in local space
    }
}
```

##### Key Difference from AddForce
- `AddForce`: Uses **world space** - `Vector3.forward` always means global Z-axis[^2]
- `AddRelativeForce`: Uses **local space** - `Vector3.forward` means object's forward direction regardless of rotation

### Fields:
#### useGravity
A boolean value to turn gravity on or off in rigidbody3D

# Structs in `UnityEngine` Namespace
## Vector3
`Vector3` is a **struct** in the `UnityEngine` namespace that represents 3D vectors and points.
`Vector3` is a **value type** (struct, not a class) containing three float components: `x`, `y`, and `z`. This means it's stored on the stack and copied by value rather than by reference.

```csharp
Vector3 position = new Vector3(5f, 2f, 10f);  // x=5, y=2, z=10
Vector3 flat = new Vector3(3f, 0f);  // x=3, y=0, z=0 (z defaults to 0)
```
### Attributes
```csharp
float xValue = position.x;
float yValue = position.y;
float zValue = position.z;

// Can also use indexer
float xValue = position[0];  // x
float yValue = position[1];  // y
float zValue = position[2];  // z

Vector3.zero       // (0, 0, 0)
Vector3.one        // (1, 1, 1)
Vector3.up         // (0, 1, 0)
Vector3.down       // (0, -1, 0)
Vector3.left       // (-1, 0, 0)
Vector3.right      // (1, 0, 0)
Vector3.forward    // (0, 0, 1)
Vector3.back       // (0, 0, -1)

//magnitude of a vector
Vector3 vec = new Vector3(3, 4, 0);
float length = vec.magnitude;  // Returns 5

//Normalized Vector: Unity vector in the same direction
Vector3 direction = new Vector3(10, 0, 0);
Vector3 unit = direction.normalized;  // (1, 0, 0)


```
### Methods
#### Distance between points:
```csharp
fload distance = Vector3.Distance(pointA, pointB);
```

#### Lerp(linear interpolation)
```csharp
Vector3 current = Vector3.Lerp(start, end, 0.5f);  // 50% between start and end
```

#### MoveTowards
Returns a vector that is closer to the given target (`target.position`) from the starting point (`transform.position`) by given units (`speed * Time.deltaTime`).
```csharp
void Update() 
{
    transform.position = Vector3.MoveTowards(
        transform.position, 
        target.position, 
        speed * Time.deltaTime
    );
}
```

# Ways to do some stuff:
## Managing movements and collision

## Taking user input (Both 2D and 3D)
There are two ways of taking input in unity:
- [[The Legacy Input System]]
- [[The New Input System]]

### The Difference?
**Both the old and new Input Systems rely on device polling at the C++ engine level**. The hardware still needs to be checked every frame—there's no way around this physical requirement. 
The difference is in **how your C# code interacts with that polled data**.​

NOTE: Device polling is a method used in computing and system design where a central controller or computer periodically checks the status of connected devices to determine if they have data to report or require attention

#### Old Input System (Poll-Based in C#)
With the old system, **you** had to poll in your `Update()` method:
```csharp
void Update() 
{     
	if (Input.GetKeyDown(KeyCode.Space)) 
	// You poll every frame    
	{        
		Jump();    
	} 
}
```
This means your code is **actively checking** the input state every frame, even when nothing is happening.​

#### New Input System (Event-Driven Option)
The new Input System **still polls devices at the hardware level**, but it gives you an **event-driven approach** as an option. The system checks for state changes and only calls your methods when input actually occurs:​
```csharp
void Awake() 
{     
	jumpAction.performed += OnJump; // Subscribe once 
} 

void OnJump(InputAction.CallbackContext context) 
{     
	Jump(); // Only called when jump actually happens 
}
```

**However**, the new Input System **also supports polling** if you prefer it:[unity3d+1](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.4/manual/Actions.html)​
```csharp
void Update() 
{     
	if (jumpAction.WasPressedThisFrame()) 
	// New system polling    
	{        
		Jump();    
	}         	
	// Or continuous value reading    
	Vector2 moveValue = moveAction.ReadValue<Vector2>(); 
}
```

#### The Key Difference
**Old System:** You must poll in `Update()`[stackoverflow+1](https://stackoverflow.com/questions/79330015/unity-new-input-system-generating-multiple-events)​
**New System:** You can choose:[unity3d+1](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.8/manual/RespondingToActions.html)​
- **Event-driven**: Subscribe to callbacks, no polling in your code (better performance)[zerotomastery+1](https://zerotomastery.io/blog/unity-new-input-system/)​
- **Polling**: Use `WasPressedThisFrame()`, `ReadValue<T>()` in `Update()` (simpler for continuous input like movement)[unity3d+1](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.4/manual/Actions.html)​

The Checking Still Happens - But Not In Your Code
**Both approaches check every frame**, but here's the critical difference:
- **Event-Driven (New Input System):**
```text
Unity's Input Update (before Update()):
├─ Input System checks ALL enabled InputActions once
├─ Compares device states to previous frame
├─ If changed → Fire callbacks for affected actions only
└─ Your callback runs only if something changed

Your Update():
└─ (nothing related to input here)
```

- **Polling (Old System or New System Polling):**
```text
Unity's Input Update:
└─ Device states updated (same as above)

Your Update():
├─ Check if jump pressed
├─ Check if fire pressed  
├─ Check if reload pressed
├─ Check if crouch pressed
└─ (repeat for every possible input)
```

The term "event-based instead of poll-based" in the documentation means the new system gives you **event-driven C# code as an option**, not that it eliminated hardware polling entirely. 
The C++ engine still polls devices every frame in both systems—that's unavoidable—but the new system lets you write **event-driven game logic** that consumes fewer resources because your code only runs when input changes.​



## Making player jump

## Making player crouch 

## Making player slide 

## Making player stick to walls

---
# Utilities in Unity
## Cinemachine
**Controls camera** in unity. Gives ways to add stuff like camera shakes, camera order control, aim assist, etc,. 
Cinemachine doesn't create new physical cameras—instead, it uses **virtual cameras** (CinemachineCameras) that control your scene's single Unity Camera. 
Think of it like a TV director managing multiple camera angles.

### Classes:
#### Cinemachine Camera (`CinemachineCamera`)
Inherits `CinemachineVirtualCameraBase`, and `CinemachineVirtualCameraBase` inherits `MonoBehaviour, ICinemachineCamera`.

##### Fields 
# **Procedural Components:**

	Position Control and Rotation Control are **separate systems** that each have their own **algorithm** (method) for controlling their part of the camera transform.[^1][^5]

## How It Works

Each algorithm has a **specific way** of calculating what value to set:

### Position Control Algorithms (Body)

| Algorithm               | Method                                                               |
| :---------------------- | :------------------------------------------------------------------- |
| **Position Composer**   | Keeps target in frame using **screen position rules + damping** [^4] |
| **Orbital Follow**      | Maintains **fixed distance + orbit angle** around target [^7]        |
| **Follow**              | Maintains **fixed offset** from target in world/local space          |
| **Hard Lock to Target** | Sets position = target position (no offset) [^10]                    |
| **Spline Dolly**        | Position based on **spline path percentage**                         |

### Rotation Control Algorithms (Aim)

| Algorithm                 | Method                                                                         |
| :------------------------ | :----------------------------------------------------------------------------- |
| **Rotation Composer**     | Calculates rotation to keep target in **screen space zone + damping** [^4][^6] |
| **Hard Look At**          | Calculates rotation to **point directly at target** [^1]                       |
| **Pan Tilt**              | Sets rotation based on **user input values** [^1]                              |
| **Same As Follow Target** | Copies **target's rotation quaternion** [^1]                                   |
| **None**                  | Does nothing - rotation set by **external source** [^1]                        |

## The Pattern

Each algorithm answers the question: **"How do I calculate the transform value I control?"**

- **Hard Look At**: "Calculate the rotation quaternion that points from camera to target"[^1]
- **Rotation Composer**: "Calculate rotation to keep target in screen zone with smoothing"[^4][^6]
- **Orbital Follow**: "Calculate position at radius R, angle θ from target"[^7]
- **Hard Lock**: "Just use target's position"[^10]

Position algorithms compute **Vector3 position**, rotation algorithms compute **Quaternion rotation**.

and then each mode has a component that it requires to get added for these modes to work as intended:
#### Cinemachine Follow
It makes the main camera follow the tracking target given in [[#Cinemachine Camera (`CinemachineCamera`)]].

#### Cinemachine Hard Lock To Target
When we want the camera to be inside the tracking target given in [[#Cinemachine Camera (`CinemachineCamera`)]]



##### Methods 

---
# Stuff in Unity
# Audio System in Unity

Unity's audio system has **two main components**: AudioListener (the ear) and AudioSource (the speaker).

## Core Concept

- **AudioListener** = Your ears (microphone) - receives sound
- **[AudioSource](AudioSource%20in%20Unity)** = Speaker - plays sound
- **AudioClip** = The actual sound file (mp3, wav, ogg)

Only **1 AudioListener** per scene, usually on Main Camera. Multiple AudioSources can exist.

## Class Hierarchy

```csharp
// Both inherit from Behaviour → Component → Object
UnityEngine.Object
    └── Component
        └── Behaviour
            ├── AudioListener    // Component class
            └── AudioSource      // Component class

// AudioClip is separate - it's an asset, not a component
UnityEngine.Object
    └── AudioClip               // Asset class (not a component)
```

**AudioListener** and **AudioSource** are **Components** (inherit from `Behaviour`) that attach to GameObjects. **AudioClip** is an **asset** you assign to AudioSource.[^1][^3]


## AudioSource Component Properties

```csharp
// Common properties
audioSource.clip = myAudioClip;       // The sound to play
audioSource.volume = 0.5f;            // 0 to 1
audioSource.pitch = 1.0f;             // Speed/pitch
audioSource.loop = true;              // Repeat sound
audioSource.playOnAwake = false;      // Auto-play on start
audioSource.spatialBlend = 1.0f;      // 0 = 2D, 1 = 3D spatial
audioSource.isPlaying;                // Tells if the audio is playing

// Play methods
audioSource.Play();                   // Play clip
audioSource.PlayOneShot(clip);        // Play without interrupting current
audioSource.Stop();                   // Stop playing
audioSource.Pause();                  // Pause
```


### Basic Setup

```csharp
using UnityEngine;

public class SoundExample : MonoBehaviour 
{
    public AudioClip jumpSound;      // Assign in Inspector
    private AudioSource audioSource;
    
    void Start() 
    {
        // Get AudioSource component on this GameObject
        audioSource = GetComponent<AudioSource>();
        
        // Or add one if missing
        if (audioSource == null) 
        {
            audioSource = gameObject.AddComponent<AudioSource>();
        }
    }
    
    void Update() 
    {
        if (Input.GetKeyDown(KeyCode.Space)) 
        {
            // Play the sound
            audioSource.PlayOneShot(jumpSound);
        }
    }
}
```



### Simple Example - Background Music

```csharp
using UnityEngine;

public class MusicPlayer : MonoBehaviour 
{
    void Start() 
    {
        AudioSource music = GetComponent<AudioSource>();
        music.loop = true;           // Keep playing
        music.playOnAwake = true;    // Auto-start
        music.volume = 0.3f;         // Quiet background
        music.spatialBlend = 0f;     // 2D sound (no 3D positioning)
    }
}
```


### Example - 3D Sound Effect

```csharp
using UnityEngine;

public class Footsteps : MonoBehaviour 
{
    public AudioClip walkSound;
    private AudioSource audioSource;
    
    void Start() 
    {
        audioSource = gameObject.AddComponent<AudioSource>();
        audioSource.clip = walkSound;
        audioSource.spatialBlend = 1f;        // Full 3D
        audioSource.minDistance = 1f;         // Full volume within 1 unit
        audioSource.maxDistance = 20f;        // Silent beyond 20 units
        audioSource.loop = true;
    }
}
```


## AudioListener

```csharp
// Automatically on Main Camera - usually don't touch it
// Only 1 per scene or you get errors

// If needed, move it:
AudioListener listener = Camera.main.GetComponent<AudioListener>();
```

AudioListener has **no properties** - just add it and it works. It "listens" for all AudioSources and plays them through speakers based on distance.

# SceneManager

**SceneManager** is a **static class** in Unity that controls scene loading, unloading, and management at runtime.

## Namespace \& Class Structure

```csharp
// Namespace
UnityEngine.SceneManagement

// Class hierarchy
System.Object
    └── SceneManager    // Static class (NOT a Component)
```

**SceneManager is NOT a Component** - it's a **static utility class**. You don't attach it to GameObjects or inherit from it. All methods are **static** so you call them directly.

## Required Namespace

```csharp
using UnityEngine;               // GameObject, MonoBehaviour, etc.
using UnityEngine.SceneManagement;  // SceneManager

public class MyScript : MonoBehaviour 
{
    // Now you can use SceneManager
}
```

Must add `using UnityEngine.SceneManagement;` at the top.

## Methods

```csharp
// Load scene (replaces current scene)
SceneManager.LoadScene("SceneName");
SceneManager.LoadScene(0);  // By build index

// Load scene additively (keeps current scene)
SceneManager.LoadScene("SceneName", LoadSceneMode.Additive);

// Unload scene
SceneManager.UnloadSceneAsync("SceneName");

// Get active scene
Scene currentScene = SceneManager.GetActiveScene();

// Get scene count
int sceneCount = SceneManager.sceneCount;
```

| Type       | Name                              | Description                             |
| ---------- | --------------------------------- | --------------------------------------- |
| **Method** | `LoadScene(string name)`          | Load scene by name ​                    |
| **Method** | `LoadScene(int index)`            | Load scene by build index ​             |
| **Method** | `LoadSceneAsync(string name)`     | Load scene asynchronously ​             |
| **Method** | `UnloadSceneAsync(string name)`   | Unload scene asynchronously ​           |
| **Method** | `GetActiveScene()`                | Returns currently active Scene ​        |
| **Method** | `SetActiveScene(Scene scene)`     | Set which Scene is active ​             |
| **Method** | `GetSceneAt(int index)`           | Get Scene at index from loaded scenes ​ |
| **Method** | `GetSceneByName(string name)`     | Find Scene by name ​                    |
| **Method** | `GetSceneByPath(string path)`     | Find Scene by file path ​               |
| **Method** | `GetSceneByBuildIndex(int index)` | Find Scene by build index ​             |
| **Method** | `CreateScene(string name)`        | Create new empty Scene at runtime       |

## Attributes

| Type         | Name                        | Description                         |
| ------------ | --------------------------- | ----------------------------------- |
| **Property** | `sceneCount`                | Number of currently loaded scenes ​ |
| **Property** | `sceneCountInBuildSettings` | Total scenes in Build Settings      |

## Example

Example - Main Menu to Game

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class MainMenu : MonoBehaviour 
{
    // Call from button
    public void PlayGame() 
    {
        SceneManager.LoadScene("GameScene");
    }
    
    public void QuitGame() 
    {
        Application.Quit();
    }
}
```

Example - Restart Level

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour 
{
    void Update() 
    {
        if (Input.GetKeyDown(KeyCode.R)) 
        {
            // Reload current scene
            SceneManager.LoadScene(SceneManager.GetActiveScene().name);
            
            // Or by index
            SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex);
        }
    }
}
```

Example - Load Next Scene

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class LevelComplete : MonoBehaviour 
{
    public void LoadNextLevel() 
    {
        // Get current scene index
        int currentIndex = SceneManager.GetActiveScene().buildIndex;
        
        // Load next scene
        SceneManager.LoadScene(currentIndex + 1);
    }
}
```


## LoadSceneMode Options

```csharp
// Single mode (default) - replaces current scene
SceneManager.LoadScene("Level2", LoadSceneMode.Single);

// Additive mode - keeps current scene, adds new one
SceneManager.LoadScene("UIScene", LoadSceneMode.Additive);
```


## Important Setup

**Must add scenes to Build Settings**:

1. File → Build Settings
2. Drag scenes from Project window to "Scenes In Build" list
3. Each scene gets an index number (0, 1, 2...)

Without this, `LoadScene` throws errors.[^5]

## Scene Class

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class SceneInfo : MonoBehaviour 
{
    void Start() 
    {
        Scene currentScene = SceneManager.GetActiveScene();
        
        Debug.Log("Scene name: " + currentScene.name);
        Debug.Log("Scene index: " + currentScene.buildIndex);
        Debug.Log("Scene path: " + currentScene.path);
        Debug.Log("Root objects: " + currentScene.rootCount);
    }
}
```

### Attributes
|Type|Name|Description|
|---|---|---|
|**Property**|`name`|Name of the Scene ​|
|**Property**|`path`|File path of the Scene asset ​|
|**Property**|`buildIndex`|Index in Build Settings (-1 if not in build) ​|
|**Property**|`rootCount`|Number of root GameObjects in Scene ​|
|**Property**|`isLoaded`|Whether Scene is loaded ​|
|**Property**|`isDirty`|Whether Scene has unsaved changes

### Methods 
| Type       | Name                   | Description                         |
| ---------- | ---------------------- | ----------------------------------- |
| **Method** | `GetRootGameObjects()` | Returns array of root GameObjects ​ |
| **Method** | `IsValid()`            | Whether Scene reference is valid    |

## Async Loading (Advanced)

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;
using System.Collections;

public class LoadSceneAsync : MonoBehaviour 
{
    public void LoadLevel() 
    {
        StartCoroutine(LoadAsyncScene());
    }
    
    IEnumerator LoadAsyncScene() 
    {
        AsyncOperation asyncLoad = SceneManager.LoadSceneAsync("HeavyScene");
        
        // Wait until scene fully loaded
        while (!asyncLoad.isDone) 
        {
            Debug.Log("Loading: " + asyncLoad.progress);
            yield return null;
        }
    }
}
```


# Particle

**Particle System** is a Component in Unity that creates visual effects like fire, smoke, explosions, magic spells, and more.

## Namespace & Class Hierarchy

```csharp
// Namespace
UnityEngine

// Class hierarchy
System.Object
    └── UnityEngine.Object
        └── Component
            └── ParticleSystem  // Component (NOT MonoBehaviour)
```

**ParticleSystem is a Component** that inherits directly from `Component`, not `MonoBehaviour`. You attach it to GameObjects like any other component.

## Attributes
|Property|Type|Description|
|---|---|---|
|`main.startLifetime`|float|How long each particle lives in seconds unity3d+1​|
|`main.startSpeed`|float|Initial speed of particles unity3d+1​|
|`main.startSize`|float|Initial size of particles unity3d+1​|
|`main.startRotation`|float|Initial rotation angle of particles unity3d+1​|
|`main.startColor`|Color|Initial color of particles unity3d+1​|
|`main.gravityModifier`|float|Scales gravity effect on particles unity3d+1​|
|`main.maxParticles`|int|Maximum number of particles allowed unity3d+1​|
|`main.loop`|bool|Whether system loops continuously unity3d+1​|
|`main.playOnAwake`|bool|Auto-play when scene starts unity3d+1​|
|`main.duration`|float|Length of one cycle in seconds [unity3d](https://docs.unity3d.com/ScriptReference/ParticleSystem.html)​|
|`main.simulationSpace`|ParticleSystemSimulationSpace|Local or world space simulation unity3d+1​|
|`emission.enabled`|bool|Enable/disable particle emission unity3d+1​|
|`emission.rateOverTime`|float|Particles emitted per second unity3d+1​|
|`shape.shapeType`|ParticleSystemShapeType|Shape of emission volume (Sphere, Cone, Box, etc.) learn.unity+2​|
|`shape.radius`|float|Radius for sphere/cone shapes learn.unity+1​|
|`shape.angle`|float|Cone angle at its point learn.unity+1​|
|`isPlaying`|bool|Whether system is currently playing [unity3d](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/ParticleSystem.html)​|
|`isPaused`|bool|Whether system is paused [unity3d](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/ParticleSystem.html)​|
|`particleCount`|int|Current number of particles [unity3d](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/ParticleSystem.html)​|
|`time`|float|Playback position in seconds [unity3d](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/ParticleSystem.html)​|

## Methods 
| Method                                                             | Return Type | Description                                                                                                                               |
| ------------------------------------------------------------------ | ----------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `Play()`                                                           | void        | Start emitting particles                                                                                                                  |
| `Play(bool withChildren)`                                          | void        | Start with option to include child systems                                                                                                |
| `Stop()`                                                           | void        | Stop emitting particles ​                                                                                                                 |
| `Stop(bool withChildren, ParticleSystemStopBehavior stopBehavior)` | void        | Stop with control over children and behavior ​                                                                                            |
| `Pause()`                                                          | void        | Pause the particle system ​                                                                                                               |
| `Pause(bool withChildren)`                                         | void        | Pause with option for children [unity3d](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/ParticleSystem.html)​              |
| `Clear()`                                                          | void        | Remove all particles immediately reddit+1​                                                                                                |
| `Clear(bool withChildren)`                                         | void        | Clear with option for children [unity3d](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/ParticleSystem.html)​              |
| `Emit(int count)`                                                  | void        | Emit specific number of particles instantly [unity3d](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/ParticleSystem.html)​ |
| `Simulate(float t)`                                                | void        | Fast-forward particle system by t seconds [unity3d](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/ParticleSystem.html)​   |
| `IsAlive()`                                                        | bool        | Check if system has any active particles [unity3d](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/ParticleSystem.html)​    |
| `IsAlive(bool withChildren)`                                       | bool        | Check if system or children have particles [unity3d](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/ParticleSystem.html)​  |
| `GetParticles(ParticleSystem.Particle[] particles)`                | int         | Get current particle array for manipulation [unity3d](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/ParticleSystem.html)​ |
| `SetParticles(ParticleSystem.Particle[] particles, int size)`      | void        | Set particle array after manipulation [unity3d](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/ParticleSystem.html)​       |
| `TriggerSubEmitter(int index)`                                     | void        | Manually trigger a sub-emitter [unity3d](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/ParticleSystem.html)​              |