Chain: **Object → Component / GameObject** (siblings) → **Behaviour → MonoBehaviour**

---

## Object (base of everything)

|Property|Meaning|
|---|---|
|`name`|The object's name (string) — same one shown in the Hierarchy/Inspector.|
|`hideFlags`|Controls how this object is saved/shown in the Editor (e.g. hidden from Hierarchy, not saved to scene). Rarely touched at basic level.|

---

## Component (base of everything attachable to a GameObject)

|Property|Meaning|
|---|---|
|`gameObject`|The GameObject this Component is attached to.|
|`transform`|Shortcut for `gameObject.transform` — the Transform on the same object.|
|`tag`|Shortcut for `gameObject.tag`.|

**Self-explanatory bit:** every Component — including scripts — automatically gets `gameObject` and `transform` for free just by inheriting Component. That's why `transform.Translate(...)` works directly inside any MonoBehaviour without you ever declaring a `transform` variable.

---

## Behaviour (adds enable/disable capability)

|Property|Meaning|
|---|---|
|`enabled`|Bool. The checkbox next to a component in the Inspector. `false` = component is attached but inactive (e.g. a disabled script stops receiving `Update()`).|
|`isActiveAndEnabled`|Read-only bool. `true` only if **both** `enabled` is true **and** the GameObject itself is active in the hierarchy.|

**Self-explanatory bit:** `enabled` is per-component. A GameObject can be active while one of its scripts is individually disabled — that's what this property controls.

---

## MonoBehaviour (the scripting bridge)

Adds **no major new properties** of its own — it inherits everything above (`name`, `gameObject`, `transform`, `enabled`, etc.) as-is.

Its actual job isn't properties — it's giving your script the magic lifecycle **methods** (`Start()`, `Update()`, etc.) and helper methods like `StartCoroutine()`, `Invoke()`, `GetComponent<T>()`. Property-wise, treat it as "Behaviour, plus scripting capability."

---

## GameObject (the container itself)

|Property|Meaning|
|---|---|
|`activeSelf`|Read-only bool. Whether _this_ object's own active checkbox is on — ignores parents.|
|`activeInHierarchy`|Read-only bool. Whether the object is _actually_ active in the scene, factoring in every parent above it. A GameObject can have `activeSelf = true` and still be `activeInHierarchy = false` if a parent is disabled.|
|`tag`|The Tag assigned in the Inspector (for identifying/filtering objects, e.g. `"Player"`).|
|`layer`|The Layer index — used for physics collision filtering, camera rendering, raycasts.|
|`transform`|The Transform attached to this GameObject (every GameObject has exactly one, guaranteed).|
|`scene`|Which Scene this GameObject belongs to.|
|`isStatic`|Bool — marks the object as non-moving for engine optimizations (baked lighting, occlusion culling, etc.).|
|`name`|Inherited from Object.|

**Self-explanatory bit:** `activeSelf` vs `activeInHierarchy` is the GameObject-level version of the same "local vs. effective" pattern you already know from Transform's local-vs-world split.

---

_(To be continued — add the next inheritance-chain detail here as you learn it.)_