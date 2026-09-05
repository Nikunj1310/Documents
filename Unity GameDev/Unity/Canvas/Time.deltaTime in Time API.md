
**What it is:** the amount of time, in seconds, that passed between the _previous_ frame finishing and the _current_ frame starting.

- **Type:** `float`, read-only (you can't assign to it — you can only read it).
- **Scope:** it's not a per-script or per-GameObject value. It's one single global number, the same for every script that reads it during a given frame.

### Why it exists — frame-rate independence

`Update()` runs once per frame, but frame rate isn't constant. A powerful PC might run your game at 144 fps; a phone might run it at 30 fps. If you move an object by a fixed amount every `Update()` call, it will physically move **faster** on the high-fps machine and **slower** on the low-fps machine, purely because `Update()` is being called more or less often — not because you told it to move at different speeds.

`Time.deltaTime` fixes this. By multiplying your movement/change by `Time.deltaTime`, you convert "amount per frame" into "amount per second," which stays consistent regardless of frame rate.

csharp

```csharp
void Update()
{
    // Moves at "speed" units per SECOND, not per FRAME
    transform.Translate(Vector3.forward * speed * Time.deltaTime);
}
```

Without `* Time.deltaTime`, `speed` would mean "units per frame" — and the object's real-world speed would silently depend on whatever fps the machine happens to be hitting.

### The obvious-but-worth-stating part

`Time.deltaTime` is usually a _small_ number — for a game running at 60 fps, each frame takes about `1/60 ≈ 0.0167` seconds, so `Time.deltaTime` will read roughly `0.0167` on most frames (it fluctuates slightly frame to frame, since real frame timing isn't perfectly even).

### One nuance tied directly to `deltaTime` itself

`Time.deltaTime` behaves differently depending on _where_ you read it:

- Inside **`Update()`** — it returns the real, variable time since the last rendered frame (what's described above).
- Inside **`FixedUpdate()`** — it instead returns Unity's _fixed_ timestep value (a constant, configured in Project Settings), because `FixedUpdate` runs on its own fixed schedule independent of the rendering frame rate.

This isn't a separate variable — it's the same `Time.deltaTime` call, just returning a different number depending on which of the two loop methods you're currently inside.
