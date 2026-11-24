# Scripted State — Exit Actions

**Exit Actions** define the conditions that end a Scripted State and route the character into its next state, blueprint, or state chain.

## How Exit Actions Execute

During each `FixedUpdate` tick, the character controller evaluates the active state in a strict top-to-bottom sequence:

1. **Exit Actions:** Evaluated one by one from top to bottom. The moment an Exit Action evaluates to `true` and finds a valid transition target, evaluation immediately stops—no remaining Exit Actions, Main Actions, or Sub Actions run on that tick.
2. **Main Action:** Executes if no Exit Action triggered.
3. **Sub Actions:** Execute sequentially from top to bottom if no Exit Action triggered.

---

## Core Exit Action Variables

Every Exit Action ScriptableObject—regardless of which specific exit condition it checks—shares the following core routing variables:

<img width="1508" height="830" alt="image" src="https://github.com/user-attachments/assets/aa16be23-90ab-47ac-869c-0958944eee76" />

> **Blueprint & State Override Priority (Top-to-Bottom Rule):**
> The blueprint and state override fields below follow a strict top-to-bottom priority hierarchy in the Inspector. **A lower option only executes if every option above it is disabled or left empty.** If multiple conflicting overrides are set on the same Exit Action, the system logs a warning in the Console, and removes lower priority changes.

### Categories to Swap To
An ordered list of State Categories that this exit will attempt to enter once its condition evaluates to `true`.

- **Priority Fallback Order:** The controller checks this array from top to bottom and transitions into the **first** category whose Gates allow entry.
- **Avoiding Premature Stat Refills:** Ordering this array alongside properly configured Gates prevents accidental state bounce. For example, if an `Ascend` (Jump) state exits into `Idle` without checking whether the character is actually grounded, `Idle` will immediately trigger any landing Sub Actions (such as `Replenish Stat` for jumps) mid-air before falling into `Descend`.
  - **Recommended Pattern:** Configure `Idle` and `Movement` with a **Grounded** Gate, and configure `Descend` with a **Not Grounded** Gate. Then order your jump's **Categories to Swap To** as `Descend` -> `Movement` -> `Idle`. Because a character cannot be grounded and airborne at the same time, airborne exits cleanly select `Descend`, while grounded exits skip `Descend` and select `Movement` or `Idle`.

### Fall Back To Master Blueprint
When enabled (`true`), a successful exit performs a **full reversion** back to the character's original **Master Blueprint**, re-initializing its attributes, passives, stats, and states.
- Instead of packing dozens of states into one massive blueprint (such as Idle, Patrol, Chase, and Attack), you can split AI or complex characters into smaller, focused blueprints (an Idle Blueprint, a Chase Blueprint, an Attack Blueprint) so a character can never get stuck between unrelated states—and simply fall back to the Master Blueprint when a phase finishes.

### Fall Back To Master Blueprint (No Reinit)
When enabled (`true`), reverts the character's active states back to the original **Master Blueprint**'s states **without** re-initializing the character or resetting stats, passives, or targets.
- **Pairs with Temporary Blueprint Swaps:** Ideal for returning from a temporary stance or transformation (such as a Berserker mode entered via **Blueprint To Swap To Without Overriding Master Config**) while preserving current health, active buffs, and target locks.

### Blueprint to Swap To
Performs a **full hard swap** to a brand-new Character Blueprint upon exiting.
- **Full Reset Warning:** This completely re-initializes the character from scratch—wiping all active passives, clearing the current target, resetting world-space orientation, resetting all stats to base values, and loading the new blueprint's attributes and states.
- **Best Used For:** Permanent or multi-phase transformations where reverting is not needed—for example, a multi-phase boss transitioning from Phase 1 into a stronger Phase 2 boss. Because the framework has no hardcoded Death state, you can even route a boss's zero-health exit into a temporary "Phase Transition / Death" blueprint (playing a stagger/death animation with an exit timer) that swaps into the Phase 2 blueprint when the timer finishes.

### Blueprint To Swap To Without Overriding Master Config
Swaps the character's active state set to another blueprint **without** overwriting the stored Master Blueprint reference or resetting character attributes.
- **States Only:** This option reads **only the States** from the target blueprint and ignores its Character Attributes, stats, and passives.
- **Best Used For:** Temporary stances or transformations—such as a Berserker rage mode—where the character temporarily uses a different set of moves and later returns to its original moveset using **Fall Back To Master Blueprint (No Reinit)**.

### State to Inject on Exit
Hot-swaps a specific State ScriptableObject directly into the character's active state table upon exiting—without modifying the underlying Master Blueprint. This enables **self-chaining (cascading) states** entirely in the Editor with zero custom code:

- **Progressive Jump Example (`PixelDot2D/ModularCharacter/SO/Blueprints/Progressive Jump`):**
  1. Inspect **Jump 1**—its exit injects **Jump 2** into the active `Jump` slot.
  2. **Jump 2**'s exit injects **Jump 3**.
  3. **Jump 3**'s exit cycles back to **Jump 1**.
  4. Because the Master Blueprint itself remains untouched, an accompanying passive (such as the included **Jump Watchdog** passive) can automatically revert the character back to the Master Blueprint's default **Jump 1** if the player waits longer than `1 second` before jumping again.
- **Combo Attack Chains:** The same pattern lets you build multi-hit attack combos (`Attack 1` -> `Attack 2` -> `Attack 3`) purely through data. Because state injection only triggers when the exit condition actually succeeds (such as completing the attack timer or pressing the combo input window), getting interrupted early by an incoming-damage exit automatically prevents the combo from advancing.

> **Best Practice — Always Reserve State Slots Using Buffer States:**
> Because blueprints can hot-swap states or inject new states at runtime, **every Character Blueprint should include a slot for every State Category your project uses**—even if a specific character cannot perform those actions yet.
>
> Internally, the controller rejects any transition to a State Category that does not exist in the current blueprint (and falls back to `Idle`). For example, if a Patrol AI blueprint only contains `Idle` and `Movement`, and later hot-swaps or injects an `Ascend` state, transitions into `Ascend` will fail unless an `Ascend` slot already exists in the blueprint.
>
> To reserve unused state slots safely, use the included **Buffer State**:
> 1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
> 2. Navigate to **ModularCharacter -> States -> Buffer**.
>
> <img width="1025" height="771" alt="image" src="https://github.com/user-attachments/assets/1bf639b6-53d1-4eef-8f64-ee6cebcc631f" />
>
> A Buffer State is an empty placeholder asset with only two settings:
>
> <img width="1081" height="457" alt="image" src="https://github.com/user-attachments/assets/39e9c3c1-9089-455c-aef5-c2a1bcf9d445" />
>
> - **Category:** The State Category slot to reserve (e.g., `Ascend`, `Descend`, or `Glide`).
> - **Can Enter:** When unchecked (`false`), the controller cleanly blocks entry into that slot—making it ideal both for reserving AI hot-swap slots and for locking player abilities (such as `Glide`) until unlocked at runtime.

---

## Creating Exit Actions

To create an Exit Action asset:
1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. Navigate to **ModularCharacter -> States -> Scripted State -> Scripted Exit Actions**.

<img width="1090" height="765" alt="image" src="https://github.com/user-attachments/assets/d72564aa-b336-4834-a895-7dafab02e423" />

---

## Included Exit Actions

### Distance To Target

<img width="887" height="122" alt="image" src="https://github.com/user-attachments/assets/52fec046-4ba3-42b6-b7b9-18dbce57e6fc" />

Evaluates the distance between the character and its active target—ideal for AI entering or exiting attack, chase, or retreat ranges.
- **Comparison Type:**
  - `EqualTo`: Uses Unity's `Mathf.Approximately`. *Not recommended for continuous movement checks as floating-point precision can cause missed triggers.*
  - `LessThan` / `LessThanOrEqualTo`: Triggers when distance is below (or equal to) the threshold.
  - `GreaterThan` / `GreaterThanOrEqualTo`: Triggers when distance exceeds (or equals) the threshold.
- **Distance:** The target distance threshold.
- **Offset:** Positional offset applied when measuring the distance check.

---

### On Collision

<img width="877" height="125" alt="image" src="https://github.com/user-attachments/assets/140723f6-f0e4-4800-9898-305c37756768" />

Queries a Core Sensor and exits when its collision state matches the configured boolean.
- **Exit on Collision:** When `true`, exits when the sensor detects a collision; when `false`, exits when the sensor detects *no* collision.
- **Collision Check:** Any Sensor asset created via the Creation Hub under **Core -> Sensors** (all shapes supported).

<img width="982" height="755" alt="image" src="https://github.com/user-attachments/assets/b8f1458c-d322-419e-a526-187f4946dc40" />

---

### Orientation Collision Check

<img width="887" height="180" alt="image" src="https://github.com/user-attachments/assets/3d277dfc-3e70-4ab1-99d6-b8cc75daf4b8" />

Performs a directional collision check that automatically selects between a positive or negative sensor based on the character's current facing or world axis.
- **Orientation Source:** Determines which orientation value selects the active sensor (`Use_Sprite_X`, `Use_Sprite_Y`, `Use_World_Axis_X`, or `Use_World_Axis_Y`).
- **Positive Sensor / Negative Sensor:** Assign two Sensor assets (created via **Core -> Sensors**). For example, when using `Use_Sprite_X`, an unflipped sprite (`flipX == false`) queries the **Positive Sensor**, while a flipped sprite (`flipX == true`) queries the **Negative Sensor**.

<img width="982" height="755" alt="image" src="https://github.com/user-attachments/assets/a3995cbf-0e76-4928-ad26-7c5e04863354" />

---

### Persistent Cooldown

<img width="1043" height="101" alt="image" src="https://github.com/user-attachments/assets/1b85485f-5860-45d8-aa98-51e1bfbbb522" />

Maintains a persistent cooldown timer across state transitions so a state can automatically exit if re-entered before its cooldown finishes.
- **Allow Cooldown During Same State:** When `true`, the timer continues ticking while inside this state; when `false`, the cooldown pauses while this state is active.
- **Cooldown:** Cooldown duration in seconds. Starts ready by default and only begins cooling down once the state is entered and this exit triggers.

---

### Stat Duration

<img width="1055" height="181" alt="image" src="https://github.com/user-attachments/assets/cf53705d-b8d4-4f46-8ee6-1fbe0b49a317" />

Drives a state's duration using a character stat rather than a hardcoded timer (for example, creating `Enum_Stat.DashDuration` so buffs or equipment can extend dash time).
- **Stat:** The character stat read as the duration in seconds.
- **Min Allowed:** Minimum fallback duration if the stat is `0` or lower (e.g., setting `0.1` guarantees the state still runs for at least `0.1s`).
- **Countdown Rate:** Speed multiplier for the timer per `FixedUpdate` (`1` = real-time seconds, `< 1` = slower countdown, `> 1` = faster countdown).
- **Drain Current:** When `true`, actively drains the character's current stat value as time ticks down. When `false`, snapshots the stat value upon entering the state and counts down internally without modifying the stat.

---

### Stat Requirement

<img width="1072" height="122" alt="image" src="https://github.com/user-attachments/assets/82d02a41-cb5b-4d7c-acd9-ee6e188530fe" />

Exits once a target character stat satisfies a comparison threshold (such as exiting a charging state when Mana reaches full, or exiting a sprint state when Stamina drops to `0`).
- **Stat:** The character stat to monitor.
- **Comparison Type:** How the current stat value is compared against **Amount** (`LessThan`, `LessThanOrEqualTo`, `GreaterThan`, `GreaterThanOrEqualTo`, or `EqualTo`—avoid `EqualTo` for continuously changing stats).
- **Amount:** The threshold value required to trigger the exit.

---

### Target Condition

<img width="1043" height="118" alt="image" src="https://github.com/user-attachments/assets/630e256c-3703-4427-8cbf-881a4b27e0d7" />

Exits based on whether the character controller currently has an active target—ideal for AI transitions between patrolling, chasing, and returning to idle.
- **Exit on Target Null:**
  - When `true`: Exits when the active target is lost (`null`).
  - When `false`: Exits as soon as a valid target is acquired (`!= null`)—for example, pairing a `Set Target On Collision` Sub Action in an `Idle` or `Movement` patrol state with **Exit on Target Null = `false`** and **Blueprint To Swap To Without Overriding Master Config** to seamlessly transition an AI into its Chase blueprint upon spotting the player.

---

### Timer

<img width="1077" height="137" alt="image" src="https://github.com/user-attachments/assets/59ced689-e7ca-4fef-9c63-cf62007ed779" />

Exits after a randomized duration in seconds.
- **Timer:** An array of durations (in seconds); one value is picked at random each time the state is entered.
- **Tip — Randomized AI Patrols:** You can stack multiple **Timer** Exit Actions on the same state, each pointing to different target categories (such as one exiting to `Idle` and another exiting to `Movement`) with identical duration arrays (e.g., `5`, `10`, `15`). Because each exit rolls its own random duration on state entry, whichever rolls shorter triggers first—producing natural, unpredictable patrol pacing.

---

## Input-Based Exit Actions

### Input Action

<img width="1057" height="787" alt="image" src="https://github.com/user-attachments/assets/d0d754aa-290c-498f-a579-cfe5a1a10769" />

A priority-ordered input router that maps keybinds to target states without hardcoded `if/else` chains. Multiple entries can share the **exact same keybind** to create emergent, context-sensitive abilities:
- **Example (Jump -> Glide on One Button):** Add two entries bound to the **Jump** key—the first targeting `Ascend` (Jump) and the second targeting `Glide`. While the character has air-jump charges, `Ascend`'s Gates pass and the character jumps. Once jump charges hit `0`, `Ascend`'s Gates fail, so **Input Action** automatically falls through to the second entry and transitions into `Glide` using the same button.
- **Frames Allowed To Read Input:** Optional array of animation frame indices during which input polling is enabled (leave empty to allow input on all frames). For example, adding `4` restricts dash-canceling so it can only trigger on frame `4` of an attack animation.
- **Delay Before Reading Input:** Time (in seconds) after entering the state before input is polled. Values below `0` (default `-1`) disable the delay so input is read immediately.
- **States to Read Input For:** Priority-ordered array of input bindings (evaluated top to bottom):
  - **Name:** Automatically mirrors **State To Swap To** so collapsed array elements stay readable in the Inspector without manual labeling.
  - **Keybind:** The input binding to listen for.
  - **Enforce Release Guard:** When `true`, the player must release and re-press the key after entering the state before it can trigger. When `false`, holding the key down continuously triggers immediately.
  - **State To Swap To:** The target State Category to request. If that target state's Gates evaluate to `false`, this entry is skipped and evaluation continues to the next element in the array.

---

### Input Direction

<img width="1063" height="97" alt="image" src="https://github.com/user-attachments/assets/cc213817-cd97-49a5-82c0-9082b7643514" />

Monitors directional movement axes and exits when input presence matches the requirement (for example, exiting `Idle` into `Movement` as soon as directional input is pressed, or exiting `Movement` back to `Idle` when input stops).
- **Axis Filter:** Which axes to inspect (`X_Axis`, `Y_Axis`, `Both` requiring both X and Y active, or `Any` requiring either X or Y active).
- **Input Requirement:** Whether the exit triggers when directional input is active (`Any Active Input`) or completely released (`No Input`).

---

### Input Release

<img width="1077" height="68" alt="image" src="https://github.com/user-attachments/assets/ce5142fd-d394-4f4b-821b-7831b66cf35c" />

Exits when a held keybind is released—commonly used for variable-height jumps where releasing the Jump button early cuts the jump short.
- **Keybind:** The input binding to monitor for release.
- **Release Grace Period:** Delay timer (in seconds) that begins counting down once the keybind is released; when it reaches `0` or less, the exit triggers `true`.
