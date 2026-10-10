# ScribeNode API: Reference Manual v1.0

Welcome to the **ScribeNode API Reference Manual**. This technical document provides comprehensive specifications for initializing dialogue sessions and formatting dialogue tree scripts for the ScribeNode Java-based narrative engine.

---

## 1. Dialogue Session Initialization

A dialogue session is instantiated when the host game environment calls the core engine framework via:

```java
DialogueEngine.open(Path script, SessionContext context)
```

### Initialization Workflow
When this method is triggered, the engine executes the following lifecycle operations:

1. **Parsing:** The engine reads the target JSON file utilizing UTF-8 encoding and parses it into a structural document object.
2. **Schema Validation:** The document is strictly validated against the internal dialogue schema definitions (verifying required fields, unique node IDs, and resolvable routes).
3. **Graph Construction:** An in-memory graph architecture is generated, mapping one `DialogueNode` instance per `nodes[]` entry, indexed securely by its unique `id`.
4. **Context Binding:** A `DialogueSession` state is instantiated, bound directly to the runtime graph and the supplied `SessionContext` (which governs state flags, custom variables, and the active speaker table).
5. **Entry Point Resolution:** The engine points the active state to the node ID specified in `meta.start`. If `meta.start` is omitted, it defaults to the first index element inside the `nodes` array.
6. **Session Return:** The active session is returned to the runtime environment. The host application can then call `session.current()` to render the active dialogue line, and `session.choose(choiceId)` to process player input and branch the state.

> ⚠️ **Important Architecture Note:** Dialogue files are static and are not watched or hot-reloaded post-initialization. Any real-time modifications to the narrative files require an explicit invocation of a new `DialogueEngine.open()` routine. The runtime state managed inside `SessionContext` is transient and is never serialized back into the source JSON script.
