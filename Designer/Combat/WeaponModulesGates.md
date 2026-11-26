# Weapon Modules: Gates

**Execution Gates** act as the first line of defense before a weapon is allowed to fire. Whenever a weapon attempts to execute, it checks its attached Gate first—if the condition is not met (such as a cooldown timer still running), the weapon blocks the shot immediately.

> **Gate Checks vs. Busy States:** Passing a Gate does not guarantee an immediate shot if the weapon is already mid-sequence. For example, if a burst weapon is firing multiple delayed projectiles, the weapon enters a **busy** state and rejects new trigger requests until the burst finishes—even if the Gate condition itself is satisfied.

To create an Execution Gate module:
1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. Navigate to **Combat -> Weapons -> Execution Gate Types**.

<img width="1007" height="772" alt="image" src="https://github.com/user-attachments/assets/e1af414c-3c02-4b49-ac00-b9b8661f5682" />

## Execution Gate Types

Execution Gates act as the firing condition and resource controller, determining when a weapon is allowed to trigger.

### Ammo

<img width="1057" height="641" alt="image" src="https://github.com/user-attachments/assets/a79ef406-d1bd-4ee9-88dc-0d70cf309ce9" />

Manages ammunition counts, reload behaviors, and charge regeneration:

- **Reload Mode:**
  - **Exhaust:** Automatically reloads only after all ammo is depleted.
  - **Input:** Reloads only when the player presses the reload input (requires the Keybind Manager in the scene).
  - **Hybrid:** Reloads automatically when ammo hits `0`, or manually at any time via input (requires the Keybind Manager in the scene).
- **Reload Keybind:** Input binding used when **Reload Mode** is set to **Input** or **Hybrid**.
- **Ammo Mode:**
  - **Magazine:** Standard clip behavior—ammo stays depleted until a reload completes.
  - **Regenerating:** Skill-charge behavior—automatically regenerates `1` ammo every **Ammo Recovery Cooldown** seconds without needing a full reload.
- **Ammo Count:** Maximum ammunition or charge capacity.
- **Execution Cooldown:** Minimum delay between individual shots while ammo is available.
- **Ammo Recovery Cooldown:** In **Magazine** mode, acts as the full reload duration (in seconds). In **Regenerating** mode, acts as the time required to recover `1` charge.
- **Ammo Consumption Rate:** How much ammo is consumed per single execution.
- **Allow Cooldown While Executing:** When enabled, cooldown/recovery timers tick immediately during multi-shot bursts rather than waiting for the burst to finish.

### Gate Charge

<img width="1057" height="371" alt="image" src="https://github.com/user-attachments/assets/9a7002fe-73a8-4daa-80b5-cdc8e3583f86" />

Holds execution until the player charges the weapon to full capacity:

- **Charge Time:** Time in seconds required to reach full charge while the execute flag is held active.
- **Charge Decay:** Rate at which charge drains when the trigger is released before reaching full charge (`0` = no decay).

### Cooldown

<img width="1058" height="368" alt="image" src="https://github.com/user-attachments/assets/6bedb137-3db9-4359-96c9-4d905f2b5cbb" />

Enforces a standard timer delay between executions:

- **Cooldown:** Minimum delay in seconds between attacks.
- **Allow Cooldown During Execution:** When enabled, the cooldown timer starts ticking immediately when execution begins rather than waiting for multi-shot bursts or hitbox durations to finish.

---

---

## Shared

> [!IMPORTANT]
> Shared components require the **Shared Character** setup, which bridges decoupled systems—such as combat, inventory, damageables, weapons, and currency—into a unified character without coupling their underlying assemblies. In order to use any of the items below, you **must** use the Shared system.

### Item Requirement

<img width="1112" height="873" alt="image" src="https://github.com/user-attachments/assets/93868d24-5e37-450a-9399-839a7a6342ab" />

An Execution Gate that requires and consumes items from the character's inventory before permitting weapon execution (e.g., consuming arrows for a bow or reagents for a spell).

#### Variables
* **Requirement UI Description:** The text returned for UI tooltips and weapon descriptions (e.g., `"Requires 1 Arrow"` or `"Requires 1 Ammo"`). Because this gate supports an array of valid items ordered by priority, descriptions are authored manually rather than auto-generated—giving designers full control over how multi-item requirements are worded.
* **Item Requirement Type:** Controls how the **Item Requirements** array is evaluated and consumed:
  * **FirstConsumed:** Checks the array from top to bottom (index `0` first) and consumes the first entry whose stack requirement is satisfied, immediately allowing execution. Place your highest-priority items at the top of the list (e.g., End-Game Arrows -> Mid-Game Arrows -> Beginner Arrows).
  * **Must_Have_All:** Checks that every entry in the array is present in the inventory before consuming all of them at once; rejects execution if any single item requirement is missing (e.g., an arcane staff requiring both a Mana Crystal and an Arcane Rune).
* **Item Requirements:** An ordered list of required item entries evaluated from top to bottom. Each entry contains:
  * **Item:** The Item Blueprint ScriptableObject required for execution.
  * **Stack Count:** The number of items consumed per execution. Internally clamped between `1` and the item's **Max Stack Count**.
* **Delay Between Successful Executes:** The cooldown duration (in seconds) enforced after successfully validating and consuming the required item(s).


