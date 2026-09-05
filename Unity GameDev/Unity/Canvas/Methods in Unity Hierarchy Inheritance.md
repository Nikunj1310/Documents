Same chain as the properties doc: **Object → Component / GameObject** (siblings) → **Behaviour → MonoBehaviour**

---

## Object (base of everything)

|Method|Meaning|
|---|---|
|`Destroy(obj, delay = 0)`|**Static.** Removes a GameObject, Component, or asset. Runs at end of the current frame, not instantly.|
|`DestroyImmediate(obj)`|**Static.** Destroys right now, no frame delay. Mainly for Editor scripting — avoid in normal gameplay code (can break the physics/render loop mid-frame).|
|`Instantiate(original)`|**Static.** Clones an object — this is how you spawn prefabs at runtime (bullets, enemies, pickups).|
|`DontDestroyOnLoad(target)`|**Static.** Marks an object to survive scene loads instead of being destroyed with the old scene.|
|`FindFirstObjectByType<T>()`|**Static.** Searches the whole scene for the first object of type T. Slow — avoid calling every frame.|
|`ToString()`|Returns the object's name.|

**Self-explanatory bit:** `Destroy` / `Instantiate` are static — you call them as `Destroy(gameObject)`, not `gameObject.Destroy()`. That's because they don't belong to one specific object; they're general-purpose factory/cleanup calls.

---

## Component (base of everything attachable to a GameObject)

|Method|Meaning|
|---|---|
|`GetComponent<T>()`|Gets another component of type T on the **same** GameObject.|
|`GetComponents<T>()`|Same, but returns **all** matches (e.g. multiple `AudioSource`s).|
|`GetComponentInChildren<T>()`|Searches this object **and its children** for type T.|
|`GetComponentInParent<T>()`|Searches this object **and its parents (upward)** for type T.|
|`TryGetComponent<T>(out T comp)`|Same as `GetComponent<T>`, but returns bool success instead of `null` — avoids the overhead of a null check on the returned reference. Preferred when the component might not exist.|
|`CompareTag(string tag)`|Checks this object's tag without the string-allocation cost of `gameObject.tag == "X"`.|

---

## Behaviour (enable/disable capability)

No new methods of its own — it exists purely to add the `enabled` / `isActiveAndEnabled` **properties** (covered in the properties doc). Nothing new to learn method-wise here.

---

## MonoBehaviour (the scripting bridge) — the important one

### Lifecycle methods (called automatically by Unity, in this rough order)

|Method|When it runs|
|---|---|
|`Awake()`|Once, when the object is first loaded — before `Start()`, even if the script is disabled.|
|`OnEnable()`|Every time the object/component becomes active.|
|`Start()`|Once, before the first frame's `Update()` — only if the script is enabled.|
|`Update()`|Every frame.|
|`FixedUpdate()`|On a fixed timestep, independent of frame rate — used for physics.|
|`LateUpdate()`|Every frame, **after** all `Update()` calls have finished (e.g. camera-follow logic).|
|`OnDisable()`|When the object/component becomes inactive.|
|`OnDestroy()`|Once, when the object is destroyed.|

**Self-explanatory bit:** none of these are overrides (`void Update()`, not `override void Update()`) — Unity finds them via Reflection at scan time, as covered in your earlier notes.

### Helper methods

|Method|Meaning|
|---|---|
|`StartCoroutine(...)`|Begins a coroutine — code that can pause/spread across multiple frames (`yield return`).|
|`StopCoroutine(...)` / `StopAllCoroutines()`|Cancels a running coroutine (or all of them on this script).|
|`Invoke("MethodName", delay)`|Calls a method once after a delay, by string name.|
|`InvokeRepeating("MethodName", delay, repeatRate)`|Calls a method repeatedly, starting after `delay`.|
|`CancelInvoke()`|Cancels any pending `Invoke`/`InvokeRepeating` calls on this script.|

---

## GameObject (the container itself)

|Method|Meaning|
|---|---|
|`SetActive(bool value)`|Turns the GameObject on/off in the scene — this is what actually changes `activeSelf`.|
|`AddComponent<T>()`|Attaches a new component of type T to this GameObject at runtime.|
|`GetComponent<T>()`|Same idea as Component's version — GameObject has its own copy of this method (it's a sibling of Component, not a subclass of it).|
|`CompareTag(string tag)`|Same as Component's version.|
|`Find(string name)`|**Static.** Finds a GameObject in the scene by name. Slow, and fails silently (`null`) if the object is inactive — avoid using every frame.|

**Self-explanatory bit:** the fact that both `GameObject` and `Component` separately define `GetComponent<T>()` / `CompareTag()` is a direct callback to them being **siblings** under `Object`, not one inheriting from the other — Unity just duplicated the convenience methods on both so you don't have to write `gameObject.GetComponent<T>()` redundantly from inside a script that's already a Component.

---

_(To be continued — add the next inheritance-chain detail here as you learn it.)_