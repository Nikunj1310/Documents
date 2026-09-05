![[Unity Scripting(The Starting Point)#2. Inheritance (MonoBehaviour) — "Is-a"]]


It is a script in `UnityEngine` Namespace. Has properties and methods, and is used by all the object that are / can be used in the game scene/ actual game in unity.

## The Magic Methods in MonoBehaviour:
### What are Magic Methods?
MonoBehaviour methods are actually "magic methods" or messages that Unity calls via reflection if they exist in your script, rather than virtual methods you override.
Unity's message system checks at runtime whether a script contains specific method names like `OnCollisionEnter2D`, `Start()`, or `Update()`. 
If the method exists, Unity adds it to internal lists and calls it at the appropriate time.

### `start()`

The `Start` method is called **only once** in the lifetime of a script instance. It is triggered when:

1. The GameObject is **Active**.
2. The Script (Component) is **Enabled**.
3. The frame starts, but **before** the first `Update` call.    

**It is not necessarily at the start of the game.** If you instantiate a prefab or activate a GameObject halfway through your game, its `Start` method will run at that moment (specifically, right before its first frame update).

Once a script has called its `Start` or `Awake` methods, they are "checked off" the list for that specific instance. Flipping the GameObject from inactive to active (or toggling the checkbox on the script) will **not** make them run a second time.

If you need code to run every single time an object is toggled back on, you’ll want to use `OnEnable()`.

### `FixedUpdate()`
Gets executed on each frame of the physics engine of unity. On a fixed timer (independent of frame rate).
For the first frame, get executed after running the start() method.

### `Update()`
Gets executed at the start of each frame. 
For the first frame, get executed after running the start() method.

### `LateUpdate()`
Gets executed at the end of each frame after all the update calls are finished. 
For the first frame, get executed after running the start() method.

### `OnEnable()`
**What:** Called automatically every time the script component or its GameObject becomes enabled.​

**When called:**
- GameObject activated
- Script component enabled
- Scene loads (if object already enabled)​
- **Can be called multiple times** unlike Awake/Start​

**Execution order:**
```text
Awake() → OnEnable() → Start()
```

### Invoke()
**Invoke** is a method in `MonoBehaviour` that lets you call a function after a delay.
Invoke is a **magic method** like `Start()` and `Update()` - it's built into `MonoBehaviour`. You don't see its implementation, but Unity calls it automatically based on time.[^6][^4][^1][^2]

```csharp
// These are ALL methods in MonoBehaviour
void Start() { }        // Magic method - Unity calls automatically
void Update() { }       // Magic method - Unity calls automatically  
Invoke("Method", 1f);   // Magic method - Unity handles timing
```

#### Syntax

```csharp
Invoke(string methodName, float delay);
```

- **methodName**: Name of method to call (as string)
- **delay**: Time in seconds before calling

**Important**: Only methods with **no parameters** and **void return type** can be invoked.

##### Example - Spawn After Delay

```csharp
using UnityEngine;

public class SpawnExample : MonoBehaviour 
{
    public GameObject enemy;
    
    void Start() 
    {
        // Call SpawnEnemy after 2 seconds
        Invoke("SpawnEnemy", 2f);
    }
    
    void SpawnEnemy() 
    {
        Instantiate(enemy, Vector3.zero, Quaternion.identity);
    }
}
```


### InvokeRepeating - Repeat Function

```csharp
InvokeRepeating(string methodName, float startDelay, float repeatRate);
```

- **startDelay**: Initial delay before first call
- **repeatRate**: Time between each repeat

```csharp
using UnityEngine;

public class SpawnRepeating : MonoBehaviour 
{
    public GameObject coin;
    
    void Start() 
    {
        // Call SpawnCoin after 2 seconds, then every 1 second
        InvokeRepeating("SpawnCoin", 2f, 1f);
    }
    
    void SpawnCoin() 
    {
        float x = Random.Range(-5f, 5f);
        Instantiate(coin, new Vector3(x, 2f, 0), Quaternion.identity);
    }
}
```


### CancelInvoke - Stop Invokes

```csharp
CancelInvoke();              // Cancel all invokes
CancelInvoke("MethodName");  // Cancel specific method
```

```csharp
using UnityEngine;

public class StopSpawning : MonoBehaviour 
{
    void Start() 
    {
        InvokeRepeating("SpawnEnemy", 0f, 2f);
    }
    
    void Update() 
    {
        if (Input.GetKeyDown(KeyCode.Space)) 
        {
            // Stop spawning
            CancelInvoke("SpawnEnemy");
        }
    }
    
    void SpawnEnemy() 
    {
        Debug.Log("Enemy spawned!");
    }
}
```


### IsInvoking - Check if Invoking

```csharp
bool isActive = IsInvoking("MethodName");
```

```csharp
void Update() 
{
    if (IsInvoking("SpawnEnemy")) 
    {
        Debug.Log("Still spawning enemies");
    }
}
```


### Avoiding String Errors - Use nameof

Using strings is error-prone if you rename methods. Use `nameof()`:

```csharp
using UnityEngine;

public class SafeInvoke : MonoBehaviour 
{
    void Start() 
    {
        // Safe - updates automatically if you rename PrintMessage
        Invoke(nameof(PrintMessage), 2f);
        
        // NOT SAFE - breaks if you rename PrintMessage
        // Invoke("PrintMessage", 2f);
    }
    
    void PrintMessage() 
    {
        Debug.Log("Hello!");
    }
}
```


#### Complete Example - Game Over Delay

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class PlayerDeath : MonoBehaviour 
{
    void OnCollisionEnter(Collision collision) 
    {
        if (collision.gameObject.tag == "Enemy") 
        {
            Debug.Log("Player died!");
            
            // Restart scene after 2 seconds
            Invoke(nameof(RestartLevel), 2f);
        }
    }
    
    void RestartLevel() 
    {
        SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex);
    }
}
```


## The Collision and Trigger Methods
All the Collision and Trigger callbacks are magic methods:
- [[#Collider Event Methods]]
- [[#Trigger Event Methods]]
#### How Unity Connects Them
Here's the flow:
1. **Collider components** detect overlaps/collisions through Unity's physics engine
2. **Unity's physics engine** determines which GameObjects are involved in the collision/trigger
3. **Unity scans all MonoBehaviour components** attached to those GameObjects
4. **If any MonoBehaviour has OnCollision/OnTrigger methods**, Unity automatically calls them

The Collider component provides the **data** (collision geometry, trigger status), but the **MonoBehaviour provides the callbacks** where you write your response logic.

### So do we have regular methods? Other methods we write, what about them?
Other than that, we also have regular methods, and those are executed as normal C# code.
So if I use a method of transform, then it has to be relayed to render engine right?
So is that transform method executed as a normal C# and then we have the toll calls in the actual defination of that method to the render engine.
Also, consider this:
```csharp
public class MyScript : MonoBehaviour 
{
    void Start() // Magic method - Unity calls automatically
    {
        CalculateDamage(10); // Your custom method - you call it
    }

    void CalculateDamage(int amount) // Regular C# method
    {
        // Only runs when YOU call it
    }
}
```
So here, only start is a magic method, hence what happens is that When you call `CalculateDamage()` from inside `Start()`, it doesn't become a "magic method". Here's what actually occurs at runtime:
- **Unity's C++ engine calls `Start()`** because it's a recognized lifecycle method
- **Your C# code inside `Start()` executes normally**, including the call to `CalculateDamage(10)`
- **`CalculateDamage()` executes as a regular method call** on the call stack—it's just normal C# execution at this point
The "magic" only applies to Unity's engine knowing when to invoke `Start()` in the first place. Once you're inside `Start()`, the rest is standard C# code execution—method calls, loops, conditionals all work exactly like normal C#.
