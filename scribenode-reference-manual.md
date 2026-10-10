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

## 2. Script Syntax & Schema Specifications

ScribeNode engines ingest custom data structures formatted in standard JSON. Below is a production-ready template demonstrating a standard dialogue node featuring branching choices, routing wrappers, and speaker definition variables.

### Sample Narrative File (`inn_greeting.json`)

```json
{
  "meta": {
    "id": "inn_greeting",
    "start": "inn.entrance"
  },
  "nodes": [
    {
      "id": "inn.entrance",
      "speaker": "marta",
      "text": "Evening. You look like you have been on the road.",
      "choices": [
        {
          "id": "ask_room",
          "text": "I need a room.",
          "routing": { "next": "inn.room" }
        },
        {
          "id": "ask_rumor",
          "text": "Heard anything strange lately?",
          "routing": { "next": "inn.rumor" }
        }
      ]
    },
    {
      "id": "inn.room",
      "speaker": "marta",
      "text": "Two silver for the night. Stable is extra.",
      "choices": []
    },
    {
      "id": "inn.rumor",
      "speaker": "marta",
      "text": "Travelers say the north road is watched after dark.",
      "choices": []
    }
  ]
}
```

### Schema Schema Reference Matrix

| Object Property | Required | Data Type | Description & System Behavior |
| :--- | :---: | :---: | :--- |
| `meta.id` | **Yes** | String | Global unique identifier for this specific narrative script. |
| `meta.start` | No | String | Target node ID to load upon initialization. Defaults to `nodes[0].id` if absent. |
| `nodes[].id` | **Yes** | String | Contextual unique identifier for a dialogue node inside this file scope. |
| `nodes[].speaker`| **Yes** | String | Speaker entity key, dynamically resolved via the system's `SessionContext`. |
| `nodes[].text` | **Yes** | String | The actual text localized and displayed to the end-user environment. |
| `nodes[].choices`| **Yes** | Array | List of choice objects. An empty array signifies a terminal node branch. |
| `choices[].id` | **Yes** | String | Stable parameter token passed back to execution via `session.choose()`. |
| `choices[].text` | **Yes** | String | The UI visual selection label presented directly to the user interface. |
| `choices[].routing.next` | **Yes** | String | The explicit node target ID to evaluate and transition to upon choice selection. |

## 3. System Validation & Error Handling

Script analysis executes eagerly during the initialization phase (`DialogueEngine.open()`). Should any validation constraint fail, the architecture aborts session generation entirely and throws a native `DialogueScriptException` bound to a specific runtime error code.

### Exception Reference Catalog

| Error Code Token | Category | Condition & Validation Context |
| :--- | :---: | :--- |
| `E_JSON_PARSE` | Fatal / Input | The source file stream does not comply with standard valid JSON criteria. |
| `E_SCHEMA` | Structural | A required field (`speaker`, `text`, `routing`, `choices`, `id`) is missing or uses an invalid data type. |
| `E_NODE_MISSING` | Logical | The `meta.start` parameter or a `routing.next` pointer references a node ID that does not exist in this scope. |
| `E_NODE_DUPLICATE`| Conflict | Multiple node elements inside the `nodes[]` collection declare identical `id` values. |
| `E_CHOICE_DUPLICATE`| Conflict | An individual dialogue node contains multiple branching elements sharing the same `id` value. |
| `E_EMPTY_SCRIPT` | Structure | The `nodes` node container is completely missing, uninstantiated, or empty. |

> ℹ️ **Runtime Runtime Note:** While initial graph loading handles structural parsing, `session.choose()` can still yield an `E_NODE_MISSING` exception during active runtime operations if the hosting application mutates the loaded memory graph programmatically post-validation. Source files that fail structural schema checks will never enter active execution paths.
