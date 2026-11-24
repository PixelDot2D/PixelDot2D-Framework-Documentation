# Character Blueprint

A **Character Blueprint** is the central configuration asset that defines what a character is, what it is immune to, its base stats, its faction filters, and the states it can enter.

---

## 1. Character Attributes

Character Attributes define the core identity, starting passives, and defensive rules of the character.

### Remove Duplicates and Nulls
An editor cleanup checkbox at the top of Character Attributes. Clicking it scans the **Starting Passives**, **Buff Immunities**, and **Debuff Immunities** arrays and automatically strips out any duplicate or `null` entries. Because this cleanup utility is stripped from runtime builds, clicking it before building your project is recommended.

### Entity Type
Defines the classification or race of the character (e.g., Elf, Beast, Undead, Humanoid). Other systems and passives can query this value for conditional effects or bonus damage.

### School Immunities
A flag mask specifying which elemental or magical schools this character is immune to (e.g., Fire, Frost, Shadow).
- **Buff Exception:** Internally, the Passive Manager never rejects a passive flagged with the **Buff** or **Permanent Buff** aura flag. For example, if a character is immune to the **Fire** school, an incoming Fire Damage-over-Time (DoT) debuff is rejected, while a friendly **Fire Shield** buff is still allowed—provided its aura flags are configured properly.

### Mechanic Immunities
A flag mask specifying which gameplay mechanics this character is immune to (e.g., Slows, Stuns, HoTs, DoTs). Any incoming passive carrying a matching mechanic flag is blocked.

### Starting Passives
An array of Passive assets applied to the character automatically on initialization.
- **Behavior-Driven Passives:** Passives are not limited to stat buffs or debuffs—they can also encapsulate reactive behaviors. For example, the framework includes Knockback and Invulnerability Frame (i-frame) passives that you can slot in here, keeping the character controller free of hardcoded reaction logic.

### Buff & Debuff Immunities
Stat-specific immunity arrays that neutralize incoming stat modifiers:
- **Buff Immunities:** Any stat added to this array cannot be positively buffed. For example, adding `Move_Speed` causes any incoming speed-buff passive to have its speed modification zeroed out internally.
- **Debuff Immunities:** Works identically to Buff Immunities, but zeroes out negative stat modifications (debuffs) targeting the listed stats.

---

## 2. Stats & Validation

The **Stats** array defines the base values for every stat defined in `Enum_Stat.cs`.

### Initial Setup
If you create a Character Blueprint manually outside of the bulk creation tool, you must expand the **Stats** section and click the **Validate** checkbox in the Inspector to populate the stat entries.
- If the array is incorrectly sized at runtime, the system logs an error message and falls back to default values for every stat.

### Updating Stats After Modifying `Enum_Stat.cs`
Whenever you add, remove, or reorder entries in `Enum_Stat.cs`, existing blueprints and character prefabs must be resized:
- **Save Old Data (Optional):** Check **Save Old Data** before validating. When adding new stats, this preserves all existing stat values while appending the new entries. When removing stats, it retains valid entries by index/script and trims stale data.
- **Bulk-Updating All Blueprints:**
  1. In the Unity Project window search bar, type: `t:ModularCharacterBluePrint`
  2. Select all matching assets.
  3. Click the **Validate** checkbox in the Inspector to resize all blueprints at once.
- **Updating Character Prefabs:** Any prefab containing a character component (such as the included `ModularCharacterTemplate` and `Behaviour_ModularCharacter` prefabs) also stores a local stat array and must have its **Validate** checkbox clicked after modifying `Enum_Stat.cs`.

---

## 3. Filters

Filters define how the character perceives the world around it:
- **Obstacles:** Geometry and colliders that block movement or line of sight.
- **Enemies:** Targets considered hostile to this character.
- **Friendlies:** Targets considered allied to this character.

All internal systems, passives, and state modules rely on these filters automatically—for example, a Frost Aura queries the **Enemies** filter to find valid targets, while line-of-sight checks inside passives or scripted state modules raycast against the **Obstacles** filter.

---
## 4. States

The **States** list holds all State ScriptableObjects (both Coded States and Scripted States) available to the character.

- **Remove Duplicates and Nulls:** Located directly above the States list, this editor cleanup checkbox removes duplicate state entries and `null` slots before you build.

You can add any number of states (from 1 upward), subject to two strict rules:
1. **Idle State Required:** An **Idle** state must always be present in the list.
2. **Unique State Categories:** No two states in the same blueprint may share the same State Category (`Enum_ModularCharacterStateCategory`).

> **Best Practice — Always Reserve State Slots Using Buffer States:**
> Because blueprints can hot-swap states or inject new states at runtime, **every Character Blueprint should include a slot for every State Category your project uses**—even if a specific character cannot perform those actions yet.
>
> Internally, the controller rejects any transition to a State Category that does not exist in the current blueprint (and falls back to `Idle`). For example, if a Patrol AI blueprint only contains `Idle` and `Movement`, and later hot-swaps or injects an `Ascend` state, transitions into `Ascend` will fail unless an `Ascend` slot already exists in the blueprint.
>
> To reserve unused state slots safely, use the included **Buffer State**:
> 1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
> 2. Navigate to **ModularCharacter -> States -> Buffer**.
>
> <img width="1025" height="771" alt="image" src="https://github.com/user-attachments/assets/4ba0a3a8-85bd-405a-8b27-a9614a11edba" />
>
> A Buffer State is an empty placeholder asset with only two settings:
>
> <img width="1090" height="472" alt="image" src="https://github.com/user-attachments/assets/2b984fb1-a771-43ea-9887-2d2a5c1bc9ef" />
>
> - **Category:** The State Category slot to reserve (e.g., `Ascend`, `Descend`, or `Glide`).
> - **Can Enter:** When unchecked (`false`), the controller cleanly blocks entry into that slot—making it ideal both for reserving AI hot-swap slots and for locking player abilities (such as `Glide`) until unlocked at runtime.


---

<img width="932" height="1021" alt="image" src="https://github.com/user-attachments/assets/7d20209a-d269-4ba3-90a3-5ce72ce2fc28" />

---

---

## 5. Creating a Character Blueprint (Recommended Workflow)

While you can manually create a blank Character Blueprint by opening the Creation Hub (**Ctrl + E** or right-click in the Project window -> **PixelDot2D -> Open Advanced Creation Hub**) and navigating to **ModularCharacter -> Character Blueprint**:

<img width="1131" height="752" alt="image" src="https://github.com/user-attachments/assets/4bff4f1e-8d4a-4da2-8f03-618460bc14c4" />

**Manual creation is not recommended.** Setting up a full character from scratch involves wiring together dozens of assets and references. Instead, use the **Bulk Creation Macros** built into the Creation Hub.

### Using Bulk Creation Macros
1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. Navigate to **Macros -> ModularCharacter -> Bulk Creation**.

<img width="1396" height="771" alt="image" src="https://github.com/user-attachments/assets/ad006b4d-7192-442b-b5ee-dec5b878100d" />

From here, choose the matrix generator that matches your character setup:

- **Build Player Side Scroller Blueprint Matrix:** Generates a complete Coded State blueprint pre-wired with standard side-scroller states. While all structural links are connected automatically, individual state parameters and sub-assets still need to be populated (reference the included **ClassicStyle** demo for a completed example).
- **Build Scripted Input Blueprint Matrix:** Generates a full Scripted State character tailored for input-driven control (such as a Player). Because movement in Scripted States is composable rather than hardcoded, the macro automatically shares a single movement asset across every state that naturally needs locomotion (such as Move, Ascend, and Descend) while skipping states like Dash.
- **Build Scripted Blueprint Matrix:** Generates a full Scripted State character without the input-polling wrapper. Instead of reading directional player inputs and computing input-scaled velocities, states drive the underlying Movement Manager directly using raw movement sequences—ideal for AI, NPCs, or automated behavior kits. Like the input matrix, it automatically shares movement assets across locomotion states where appropriate.

> **Note on `[Shared]` Macros:** You will also see `[Shared]` options in the Bulk Creation menu. Those macros generate blueprints built for the **Shared** character template (which acts as the decoupling bridge between `Combat` and `ModularCharacter`). Slotting a `[Shared]` blueprint into a standard Modular Character will log an error indicating an incompatible character type. See the **Shared** section of the documentation for setup details.

### Creating Individual Blank States
If you do not need an entire character matrix and only want to add a single new state to an existing blueprint, use the single-state macros in the same menu:
- **Build Scripted Input State:** Creates a standalone input-driven Scripted State ready to configure.
- **Build Scripted State:** Creates a standalone direct-movement Scripted State ready to configure.



---

## 6. Wiring Movement Sequences in Generated States

Whenever you generate a Scripted State (single or matrix), inspect its **Main Action** (or **Shared Main Action** for matrix builds) and locate the **Sequencer** slot—it will contain a pre-linked `{State/Shared}_Movement_Sequence` asset.

The sequence is already hooked up to the state, but you must assign a **Movement** asset inside it:
1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. Navigate to **Core -> RB2D Movements** and create a movement asset (for most standard 2D locomotion, select **Regular Movement** for standard X/Y vector movement).
3. Drag the new Movement asset into the `{State/Shared}_Movement_Sequence`.

### Balancing Movement Speed vs. Character Stats
The Movement asset and the character's `Move_Speed` attribute act as multipliers against each other. You can configure speed scaling in one of two ways:
- **Stat-Driven Speed (Recommended):** Set the Regular Movement asset's axis values to `1` (e.g., `X = 1, Y = 0` for side-scrollers) and let the character's `Move_Speed` stat drive the actual speed (e.g., `Move_Speed = 10` moves the character at 10 units/sec).
- **Asset-Driven Speed:** Set the character's `Move_Speed` stat to `1` and define the raw speed directly inside the Movement asset.
  - *Caveat:* Because `Move_Speed` acts as a multiplier, flat speed debuffs on a character with a base stat of `1` can reduce the multiplier to `0` and freeze movement completely unless your debuffs use percentage modifiers.

---

## 7. Scripted State vs. Scripted Input State

When configuring movement inside a Scripted State, the parameters exposed in the Inspector depend on whether the state is **Non-Input** or **Input-Driven**.

### Direct Scripted State (Non-Input)
Exposes only the **Movement Sequence** reference, executing the configured movement directly without reading directional input.

<img width="573" height="303" alt="image" src="https://github.com/user-attachments/assets/fbfaa187-858e-40f0-9e57-14d8fe24b041" />

<img width="647" height="323" alt="image" src="https://github.com/user-attachments/assets/a3cc2c94-9525-4189-ad3d-ac7bfe8f68af" />

### Scripted Input State
Wraps the Movement Sequence with directional input polling and velocity scaling controls:

<img width="582" height="302" alt="image" src="https://github.com/user-attachments/assets/2f2e437b-8833-4565-bbe2-3fc7c7598a5c" />

<img width="832" height="611" alt="image" src="https://github.com/user-attachments/assets/bd6615cf-3140-4a58-b1e9-086bc6b49af8" />


- **Max Vector:** Clamps or masks input per axis. Setting an axis to `0` ignores input along that axis (e.g., `X = 1, Y = 0` locks movement to horizontal side-scroller input).
- **Move Speed Multiplier:** Multiplies the character's `Move_Speed` stat while in this state (`1` = normal speed, `< 1` = slower movement, `> 1` = faster movement—for example, setting this to `3.5` on a Dash lets dash distance scale dynamically with the character's speed stat).
- **Forced Input:** Forces directional input on an axis (`1` or `-1`) even when the player provides no input (default `X = 0, Y = 0`). Setting `X = 1` on a Dash guarantees the character always dashes forward regardless of whether a movement key is held.
- **Input Polling Type:** Controls when and how directional input is sampled:
  - `Continuous`: Polls both X and Y input continuously every frame.
  - `Continuous X` / `Continuous Y`: Polls only the specified axis continuously every frame.
  - `Snapshot`: Captures both X and Y input once when the state is entered and locks that direction until the state exits (useful for committed aerials or dashes).
  - `Snapshot X` / `Snapshot Y`: Captures and locks only the specified axis upon entering the state.

---

## 8. Reusing States Across Blueprints

Scripted States are modular ScriptableObjects—you do not need to duplicate an entire state library for every new character variant.

Inspect the demo blueprints under `PixelDot2D/ModularCharacter/SO/Blueprints/ScriptedInputBased` to see this pattern in action: each variant blueprint points back to the shared states in `MasterScriptedInputStates`, reusing core states (such as Idle, Move, and Jump) while swapping out or adding only the specific states that make that character unique.

