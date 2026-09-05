![[Unity Scripting(The Starting Point)#3. Global API (Time, Input, etc.) — the tools you reach for]]

# The `Time` API

`Time` is one of the **Global API** classes (see Pillar 3 in your main notes). It's a **static class**, you never create an instance of it, you just call into it directly from anywhere: `Time.deltaTime`, `Time.time`, etc. No GameObject or Component reference required, because it doesn't belong to any single object in the scene — it's engine-wide state.

Under the hood, `Time` is a thin managed wrapper around the native C++ engine's internal clock. Every property you read on it is a live query into that native clock at the moment you read it, it's not a value sitting cached in your script.

## [[Time.deltaTime in Time API]]
