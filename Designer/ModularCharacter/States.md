
# Modular Character States

Every capability a character can perform—standing idle, running, jumping, dashing, or playing a scripted sequence—is encapsulated inside a **State ScriptableObject** (`SO_Modular_X`).

At runtime, the character reads the configuration data stored on these assets as **read-only**, ensuring shared State assets remain safe to reuse across multiple Character Blueprints without mutating data during play.

---

## State Categories & Routing

Every State ScriptableObject belongs to a **State Category** (`Enum_ModularCharacterStateCategory`), such as `Idle`, `Movement`, `Ascend`, `Descend`, or `Dash`.

### Coded States (Locked Category)
In a Coded State (such as the included Player Side-Scroller states), the Category Enum is locked internally to match the class's purpose. You cannot change it in the Inspector. This prevents accidental setup errors—such as forgetting to assign a category, duplicating a category, or mislabeling a Dash state as `Idle`.

### Scripted States (Flexible Routing ID)
In a Scripted State, the Category Enum is unlocked and must be assigned manually.

Think of the Category Enum in a Scripted State not as a strict rule for what the state *must* do, but as a **routing address** (or slot ID) inside that Character Blueprint:
- **Labels are suggestions, not restrictions:** While names like `Idle` or `Ascend` describe standard locomotion roles, a Scripted State can execute any behavior you assemble—for example, an AI or cutscene controller could transition from `Idle` -> `Ascend` to drive a scripted cinematic sequence.
- **How transitions use it:** When a state finishes, its exit logic tells the state machine, *"I'm done—transition to whichever state holds Category X in this blueprint."*
- **Uniqueness rule:** As long as no two states in the same Character Blueprint share the same Category Enum, you have full freedom to wire any emergent state flow you design.

---

## Anatomy of a Scripted State

While Coded States expose only the specific parameters required by their C# class, a **Scripted State** exposes modular arrays that let you compose behavior step by step in the Inspector:

<img width="823" height="931" alt="image" src="https://github.com/user-attachments/assets/bf5f61ff-549a-4cf1-abb9-549e5d93a76c" />

### 1. State Name
A custom string identifier for the state. Whenever internal systems or passives query the character's current state name (for example, displaying or checking `"Dash"` while dashing), this string is returned.

### 2. Remove Duplicates and Nulls
An editor cleanup checkbox that scans all arrays below (**Sub Actions**, **Exit Actions**, and **Gates**) and strips out duplicate entries or `null` slots. Because this utility is stripped from runtime builds, clicking it before building is recommended.

### 3. Category
The `Enum_ModularCharacterStateCategory` routing ID assigned to this state within its Character Blueprint.

### 4. Main Action
The primary locomotion driver for the state (typically a `Movement_Sequence` or input-wrapped movement configuration). If you generated the state using the Bulk Creation tool, this slot is already wired up.

### 5. Sub Actions (Order Matters)
An array of modular behavior blocks that execute **after** the Main Action every `FixedUpdate`.

**Execution order is strictly sequential from top to bottom:**
- For example, if you place **Axis Look At Sprite** above **Sprite Look At Input**, the axis orientation updates using the sprite's direction *before* the sprite reads the new input—leaving the axis one `FixedUpdate` tick behind. Always order dependent actions from cause to effect.

### 6. Exit Actions (Order Matters)
An array of exit conditions evaluated to determine when the state should end and which State Category to transition into next.

**Evaluation order determines priority—and short-circuits remaining work:**
- **First match wins:** Exit Actions are checked from top to bottom. For example, if you list a **Max Duration Timer** above an **On Input Exit (to `Ascend`)**, and the timer expires on the exact same frame the player presses Jump, the timer triggers first and routes the character to the timer's target state instead of `Ascend`.
- **Early exit stops Sub Actions:** The moment an Exit Action evaluates to true and successfully triggers a state transition, the state immediately stops and skips Sub Actions for that tick.

### 7. Gates
An array of entry requirements that act as a security checklist before the character is allowed to enter this state.
- **All-or-nothing evaluation:** Every Gate in the array must evaluate to `true` for the state to be considered valid to enter.
- For example, adding a **Ground Check Gate** (configured via a sensor checking `Collision == true`) ensures the state can only be entered while grounded; if the sensor returns `false`, the transition request is rejected.
