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

