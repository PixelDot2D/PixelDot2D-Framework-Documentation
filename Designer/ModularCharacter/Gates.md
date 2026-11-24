# Scripted State — Gates

**Gates** act as entry guards that determine whether a Scripted State is allowed to be entered. Whenever an Exit Action or controller attempts to transition into a state, the controller queries that state's Gates first—allowing blocked transitions to fail gracefully and fall through to the next candidate state in priority order.

- **All-or-Nothing Rule:** Every Gate in a Scripted State's **Gates** array must evaluate to `true` for the state to be valid to enter.
- **Empty Array Default:** If the **Gates** array is left empty, the state has no entry restrictions and is always considered valid to enter.

## Locating Scripted State Gates

To create a Gate asset:
1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. Navigate to **ModularCharacter -> States -> Scripted State -> Scripted Gates**.

<img width="1065" height="747" alt="image" src="https://github.com/user-attachments/assets/f4daaa94-adf9-463f-a7bc-c5934dd0d468" />

---

## Included Scripted State Gates

### Always Block

An empty guard asset with no configurable variables that unconditionally returns `false` on entry checks.
- **Use Case:** Acts as a hard lock preventing the character from ever entering the state directly—useful for temporarily disabling a state or reserving a state slot designed strictly for background evaluation rather than active entry.

---

### Orientation Collision Check

<img width="1190" height="461" alt="image" src="https://github.com/user-attachments/assets/7c47b9a6-5875-4500-934a-7a416ffe4ef7" />

Queries a directional sensor based on the character's current facing or world axis and checks whether its collision state matches the requirement (for example, preventing a Dash when pressed directly against a wall).
- **Requires Collision:** When `true`, the gate passes only if the active sensor detects a collision; when `false`, it passes only if the active sensor detects *no* collision.
- **Orientation Source:** Determines which orientation axis selects the sensor (`Use_Sprite_X`, `Use_Sprite_Y`, `Use_World_Axis_X`, or `Use_World_Axis_Y`).
- **Positive / Negative:** Assign two Sensor assets created via **Core -> Sensors** (all shapes supported). For example, using `Use_Sprite_X`, an unflipped sprite queries the **Positive** sensor while a flipped sprite queries the **Negative** sensor.

<img width="1070" height="747" alt="image" src="https://github.com/user-attachments/assets/34777973-39bb-43bb-8089-7b3da8968515" />

---

### Single Sensor

<img width="1182" height="375" alt="image" src="https://github.com/user-attachments/assets/5730ea3b-19f8-4f61-8495-9cb555fc7cba" />

Evaluates a single Core Sensor regardless of character facing direction.
- **Requires Collision:** When `true`, passes only when a collision is detected; when `false`, passes only when no collision is detected.
- **Sensor:** Any Sensor asset created via **Core -> Sensors**.

---

### Stat Requirement

<img width="1180" height="406" alt="image" src="https://github.com/user-attachments/assets/78076f9d-6b26-4e51-96cb-a6d3a6d34525" />

Blocks entry unless a target character stat satisfies a threshold comparison (for example, requiring Mana to enter a Dash or Stamina to enter a Sprint).

> **Evaluation Order & Floor:** Internally, the check evaluates as **`Amount [Comparison Type] Current Stat`**. Additionally, character stats in the framework can never drop below `0`—so configuring a negative **Amount** with a comparison that expects the stat to be below `0` will always evaluate to `false`.

- **Stat:** The character stat to check.
- **Comparison Type:** How **Amount** is compared against the current stat (`LessThan`, `LessThanOrEqualTo`, `GreaterThan`, `GreaterThanOrEqualTo`, or `EqualTo`).
- **Amount:** The target threshold value.

---

### State Gate

<img width="1190" height="327" alt="image" src="https://github.com/user-attachments/assets/b828abc2-9786-4f8d-98c7-e4138ee6fc58" />

Wraps one of the framework's dedicated **State Gate** assets (shared across both Coded States and Scripted States) to enforce stateful resource costs or dynamic cooldown rules.

To create a State Gate asset:
1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. Navigate to **ModularCharacter -> State Gates**.

<img width="1062" height="763" alt="image" src="https://github.com/user-attachments/assets/07a88a23-be03-421b-b329-0f0154230ce8" />

---

## State Gates Reference

### Cooldown State Gate

<img width="1178" height="896" alt="image" src="https://github.com/user-attachments/assets/22bfd4d0-69c8-4148-9b2d-f9acb91b01a9" />

The **Cooldown** State Gate is more than a simple countdown timer—it combines **state-aware natural recovery** with **transition-based cooldown reduction rewards**. For example, in a classic setup, a player with a double-jump who performs **Jump -> Dash -> Jump** cannot dash again because natural cooldown recovery is paused while airborne, forcing them to land and wait out the full timer. With this system, you can keep natural cooldown recovery paused while airborne, grant a cooldown reduction when chaining **Dash -> Jump** mid-air so they can dash a second time, and trigger a full reset upon landing—creating fluid, engagement-rewarding movement instead of rigid cooldown locks.

#### How Cooldown Recovery & Rewards Work
1. **Natural Recovery:** While the cooldown is active (`>= 0`), it ticks down each frame by `deltaTime * timeScale`—**unless** the character is currently in one of the **Categories Not Allowed To Cooldown On** (for example, pausing Dash cooldown recovery while airborne in `Ascend` or `Descend` until the character touches the ground).
2. **Sequence Cooldown Reduction Rewards:** Every time the character transitions from a `Previous` state to a `Current` state, the gate evaluates two rules:
   - **Exclusion Filter First:** If the transition matches **both** an entry in **Ignore Cooldown Reduction Previous** and **Ignore Cooldown Reduction Current**, the reward check is skipped. (Leave either exclusion list empty to disable this filter.)
   - **Eligibility Reward Second:** Otherwise, if the transition matches **both** an entry in **Reduction Condition Previous** and **Reduction Condition Current**, the remaining cooldown is immediately reduced by `Cooldown * Successful Sequence Cooldown Reduction Percentage`—even if natural time-based recovery is currently paused.

#### Cooldown Inspector Variables
- **Cooldown:** Base cooldown duration in seconds.
- **Successful Sequence Cooldown Reduction Percentage:** Normalized percentage (`0.0` to `1.0`) of the **base** cooldown removed when an eligible state transition occurs (`0` = no reduction, `0.5` = `50%` reduction, `1.0` = `100%` instant reset).
- **Ignore Cooldown Reduction Current / Previous:** Exclusion lists. If both the previous state and the new current state appear in their respective ignore lists, no cooldown reduction reward is granted for that transition.
- **Reduction Condition Current / Previous:** Eligibility lists. When not excluded above, transitioning from a state in **Reduction Condition Previous** into a state in **Reduction Condition Current** triggers the percentage cooldown refund.
- **Categories Not Allowed To Cooldown On:** State Categories during which natural time-based cooldown recovery pauses. Transition-based cooldown reduction rewards can still trigger during these states.

---

### Stat Consumption State Gate

Unlike a standard **Stat Requirement** gate (which only checks whether a stat meets a threshold), the **Stat Consumption** State Gate both verifies that a minimum stat amount is available **and** automatically deducts a configured cost when the state is entered.

<img width="931" height="401" alt="image" src="https://github.com/user-attachments/assets/c1ceacbb-3860-450d-bc11-2cbe5cf915ea" />

- **Stat To Consume:** The character stat checked against the entry requirement and deducted upon entering the state.
- **Consume Amount On Enter:** The amount subtracted from the stat when the state is entered (can differ from the required threshold).
- **Required Amount To Enter:** The minimum stat value required to pass the gate.
  - **Example:** If **Stat To Consume** is `Stamina`, **Required Amount To Enter** is `10`, and **Consume Amount On Enter** is `5`, the character must have at least `10` Stamina to enter the state, and entering will spend `5` Stamina.

---

## Hybrid Gates (Entry Guard + Active Duration)

### Timed Gate

While most Gates only evaluate entry conditions and leave exiting to Exit Actions, the framework's decoupled architecture allows a Gate to enforce both an entry lock and an active operational timer at the same time.

<img width="922" height="690" alt="image" src="https://github.com/user-attachments/assets/e59bd146-60a5-417c-8a38-537baf5dca27" />

The **Timed Gate** functions simultaneously as an **entry blocker** and an **active durational ceiling**:
1. **Duration:** Maximum active operational time (in seconds) allowed before the gate locks the state. While the parent state is active, this internal timer counts down.
2. **State Change On Duration End:** Priority-ordered array of State Categories the controller attempts to transition into once **Duration** reaches `0`, immediately locking the state against further re-entry.
3. **Categories To Reset Duration:** List of recovery State Categories that replenish the internal timer back to **Duration**. The gated state remains completely inaccessible until the character enters one of these states.

- **Best Used For — Resource-Limited Mechanics (e.g., Hovering):** Allows Sub Actions (such as `State Stat Shift`) to handle continuous per-second resource drains (Mana or Stamina) while the **Timed Gate** enforces a strict maximum flight/hover ceiling—preventing players from re-entering Hover mid-air until they touch down in a recovery state like `Idle` or `Movement`.



