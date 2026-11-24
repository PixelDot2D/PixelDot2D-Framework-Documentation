# Scripted State — Sub Actions

**Sub Actions** are modular behavior blocks that execute **after** the state's Main Action every `FixedUpdate`. Any number of Sub Actions can co-exist inside a single Scripted State—by mixing, matching, and tuning their data, you can assemble virtually any gameplay behavior without writing custom C# states.

> **Reminder:** Sub Actions execute strictly in array order from top to bottom. Always place actions that generate data (such as updating sprite direction) above actions that read that data.

## Locating Sub Actions

To create a Sub Action asset:
1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. Navigate to **ModularCharacter -> States -> Scripted State -> Scripted Sub Actions**.

<img width="1050" height="767" alt="image" src="https://github.com/user-attachments/assets/01c7f631-1ef6-4bbc-977b-bf6f5dcd159e" />

---

## Included Sub Actions

### Afk Animation Trigger

<img width="898" height="712" alt="image" src="https://github.com/user-attachments/assets/ce8dcc36-a3fb-4b47-9b4d-cd6dac7c98ed" />

Plays a randomized idle/AFK animation after a random delay—similar to classic games where a character sits down or looks around when left idle. Each trigger automatically adds the chosen `SO_Animation`'s playback length to the next wait timer so animations never cut each other off.
- **Time to Trigger:** An array of wait durations (in seconds). When triggered, the system picks a random element from this array and adds the selected animation's duration before scheduling the next trigger (e.g., a `20s` interval + a `5s` animation waits `25s` before the next roll).
- **Animations to Pick From:** An array of `SO_Animation` assets selected from at random when the timer fires.

---

### Animation Chain

<img width="898" height="676" alt="image" src="https://github.com/user-attachments/assets/bc166672-3eb0-42e6-8a5b-768a5e8a390f" />

Plays a multi-step sequence of animations when entering the state, automatically looping the final animation in the chain (for example, transitioning from `Descend` -> `Run` plays a landing `Roll` animation once and then seamlessly loops `Run`).
- **Chain Dictionary:** Maps a **Previous State Category** key to an ordered array of animations. Every animation in the array plays sequentially, and the final entry loops automatically.
- **Fallback Animations:** An ordered array of animations played if the previous state is not found in the dictionary—or whenever you want a fixed chain (such as `Roll` -> `Run`) to play regardless of which state the character came from (leave the dictionary empty in that case).

---

### Apply Gravity

<img width="906" height="612" alt="image" src="https://github.com/user-attachments/assets/01f36612-df73-4e2a-9888-f1ddf7e2fcfc" />

Applies continuous downward acceleration along the Y velocity axis.
- **Gravity Stat:** The character stat read as the base gravity strength.
- **Acceleration Rate:** Controls how quickly downward velocity builds (`1` = standard speed, `> 1` = faster acceleration, `< 1` = floatier acceleration).
- **Max Gravity:** The terminal velocity cap for downward Y speed.
- **Gravity Multiplier:** Direct multiplier applied against the **Gravity Stat**.
- **States From/To for Delay:** Optional list of previous state categories that trigger an apex hang-time delay before gravity kicks in (e.g., transitioning from `Ascend` -> `Descend`). Leave empty (size `0`) to apply gravity immediately.
- **Delay Before Applying Gravity:** Duration (in seconds) to suspend gravity upon entering from one of the configured states. Ignored if the state list above is empty.

---

### Apply Vertical Impulse

<img width="896" height="361" alt="image" src="https://github.com/user-attachments/assets/8c0a8c2b-58a5-45f6-a7b9-484248ded170" />

Applies an instantaneous upward or vertical velocity impulse along the Y axis (such as jumping).
- **Target Stat:** The character stat used as the base impulse force.
- **Multiplier:** Direct multiplier applied to the **Target Stat**.

---

### Axis Look At Sprite

<img width="903" height="406" alt="image" src="https://github.com/user-attachments/assets/42d08125-f99a-4635-8839-d1e1e120fc86" />

Aligns the character's internal world movement axis to match the visual flip direction of its sprite. For example, if a sprite is facing left and X-axis alignment is enabled, the internal X axis flips from `1` to `-1` so forward movement travels left—allowing AI characters to automatically move in the direction they are facing.
- **Look At Axis X:** Matches the internal horizontal world axis (`1` or `-1`) to the sprite's horizontal flip.
- **Look At Axis Y:** Matches the internal vertical world axis (`1` or `-1`) to the sprite's vertical flip.

---

### Flip Axis On Timer

<img width="901" height="447" alt="image" src="https://github.com/user-attachments/assets/fd129f41-86c6-4a37-b181-1cf5e54eaf1a" />

Periodically flips the character's internal world axis on a randomized timer—useful for patrolling AI that turns left/right or moves up/down at intervals.
- **Axis to Flip:** Chooses which axis to invert (`Right` flips horizontal left/right; `Up` flips vertical up/down).
- **Timer:** An array of durations (in seconds) picked at random between each axis flip.

---

### Flip World Axis On Collision

<img width="905" height="492" alt="image" src="https://github.com/user-attachments/assets/bc9a3a80-15cc-4c61-a177-484e4d9702ab" />

Queries a Sensor and flips the character's internal world axis when a collision condition is met—ideal for AI patrol turnarounds at walls or platform ledges.
- **Flip on Collision:** When `true`, flips the axis when the sensor detects a collision (e.g., hitting a wall). When `false`, flips when the sensor detects *no* collision (e.g., a downward-angled forward sensor stepping off a ledge and seeing no ground).
- **Axis to Flip:** Chooses whether to flip the horizontal (`Right`) or vertical (`Up`) world axis.
- **Positive Check:** Checks the sensor along the positive direction when the character's internal world axis is `1`.
- **Negative Check:** Checks the sensor along the negative direction when the character's internal world axis is `-1`.


---

### Play Random Animation

<img width="900" height="422" alt="image" src="https://github.com/user-attachments/assets/b7e16b40-6ea6-44ed-9e1e-192ed0ae5cb1" />

Picks and plays a single random animation from a list whenever the state is entered (for example, alternating between left-handed and right-handed attack animations).
- **Animations:** An unlimited array of animations chosen from randomly on state entry.

---

### Reduce Damage

<img width="902" height="373" alt="image" src="https://github.com/user-attachments/assets/c4e941e9-4f47-4404-9a6b-9820129cff1c" />

Applies an absolute damage reduction multiplier inside the damage pipeline. Standard damage-taken modifiers operate as an additive tug-of-war (where `-100%` damage taken can be canceled out by `+200%` damage taken, still dealing `100%` damage). This Sub Action bypasses that additive pool and directly scales final incoming damage efficiency.
- **Reduction Amount:** A normalized scalar from `0.0` to `1.0` (`0.5` = `50%` damage reduction, `1.0` = `100%` damage reduction / full negation).

---

### Replenish Stat

<img width="907" height="457" alt="image" src="https://github.com/user-attachments/assets/9886dd26-23b4-4056-9d7d-9ac5cc7128e9" />

Fully restores a target stat to its maximum value upon entering the state—commonly used to reset air-jump charges when landing in `Idle` or `Movement`.
- **Stat:** The character stat to replenish.
- **Categories to Replenish:** An array of State Categories that trigger the replenishment. This check is non-exclusive: if `Idle` is listed, transitioning from `Movement` -> `Idle` still refreshes the stat even if `Movement` never consumed it.

---

### Set Animation

<img width="901" height="337" alt="image" src="https://github.com/user-attachments/assets/6cd71f8b-a2a0-43c0-9f0e-1ecc05b566da" />

Plays a single designated animation when entering the state.
- **Animation:** The `SO_Animation` asset played on state entry.

---

### Set Rotation

<img width="902" height="383" alt="image" src="https://github.com/user-attachments/assets/323fd27b-f3da-44c4-b4d2-6f8709dc16cd" />

Sets the character's transform rotation to a specific angle—useful for resetting tilt or spin introduced by dashes, slopes, or aerial movements.
- **Target Rotation:** The target angle to apply.
- **Set on Enter:** Applies the target rotation when entering the state.
- **Set on Exit:** Applies the target rotation when exiting the state (can be enabled alongside **Set on Enter**).

---

### Set Target On Collision

<img width="900" height="435" alt="image" src="https://github.com/user-attachments/assets/470c4f8f-ebd7-4d3a-a39b-675fdfb36a19" />

Detects valid entities inside a collision shape and assigns them as the character controller's active target.
- **Collision Shape:** Any shape created via the Creation Hub under **Core -> Collision**.

<img width="1051" height="492" alt="image" src="https://github.com/user-attachments/assets/8e340a71-886e-45d2-a06d-bdef18f778b1" />

- **Line of Sight:** Optional line-of-sight check created via the Creation Hub under **Core -> Line of Sight**. If left `null`, any valid target matching the Character Blueprint's faction filters inside the collision shape is accepted automatically; if assigned, the target must also pass a line-of-sight check against the Blueprint's **Obstacles** filter.

<img width="1051" height="493" alt="image" src="https://github.com/user-attachments/assets/a162e688-3b8d-4602-b4e1-1d9d44411987" />

---

### Set Target On Damage Taken

<img width="902" height="346" alt="image" src="https://github.com/user-attachments/assets/7ab52edf-5ff2-4f1e-a8ac-bec6e7fe6a7f" />

Automatically assigns the attacker as the character controller's active target whenever the character takes damage while in this state.
- **Override If Already Has Target:** When `false`, keeps the character's existing target if one is already assigned. When `true`, always overwrites the active target with the new attacker.

---

### Set World Axis

<img width="895" height="408" alt="image" src="https://github.com/user-attachments/assets/1fa905de-94da-45d8-8cab-cc034162d488" />

Explicitly resets or forces the character's internal world orientation axis (`1` = default positive right/up, `-1` = inverted left/down).
- **Set on Enter:** Applies the configured world axis when entering the state.
- **Set on Exit:** Applies the configured world axis when exiting the state.
- **World Space:** The X and Y axis signs (`1` or `-1`) to apply.

---

### Sprite Look At Axis

<img width="901" height="367" alt="image" src="https://github.com/user-attachments/assets/fa9f965d-70df-4d55-b84b-a75d52bad444" />

Forces the character's sprite flip direction to follow its internal world axis—ideal for AI characters whose movement logic drives the world axis directly.
- **Look At Axis X:** Aligns the sprite's horizontal flip with the X world axis.
- **Look At Axis Y:** Aligns the sprite's vertical flip with the Y world axis (can be enabled alongside X).

---

### Sprite Look At Target

<img width="897" height="358" alt="image" src="https://github.com/user-attachments/assets/962eeab9-301f-4860-bd89-b3cc01d62225" />

Orients the character's sprite toward or away from the controller's active target. Does nothing if the current target is `null`.
- **Look At Target:** When `true`, faces the sprite toward the active target; when `false`, faces the sprite away from the target.

---

### State Stat Shift

<img width="901" height="591" alt="image" src="https://github.com/user-attachments/assets/984545fb-d073-4fec-a722-0133a020af66" />

Modifies a character stat (positively or negatively) upon entering and/or remaining inside a state. Because this routes through the standard stat modification pipeline rather than bypassing it, any active passives, school rules, or stat multipliers apply automatically—for example, a mana-charging state granting `+1` Mana per tick becomes `+1.5` Mana per tick if the character has a `+50%` Mana modifier passive, while draining `Jump` after `0.25s` when transitioning from `Idle` -> `Descend` creates data-driven **Coyote Time** with zero hardcoded timer logic.
- **Previous States To Trigger Stat Shift:** Optional array of State Categories that activate this shift when transitioned from. Leave empty to trigger regardless of the previous state.
- **Target Stat:** The stat to manipulate.
- **School Flags:** The elemental or magical school flags associated with this stat modification as it passes through the stat pipeline.
- **Execute Cycles:**
  - `One Shot`: Executes the stat shift only once per state entry. If **Shift on Enter** is `true`, the entry shift counts as the single execution; if `false`, it waits for the cooldown timer to trigger once.
  - `Repeating`: Repeatedly applies the stat shift on every cooldown interval while remaining in the state (also triggers immediately on entry if **Shift on Enter** is `true`).
- **Stat Shift Amount:** The base value added to or subtracted from the target stat before pipeline modifiers (positive to gain, negative to drain).
- **Internal Cooldown:** Interval (in seconds) between shifts while in the state. Resets whenever the character exits and re-enters the state.
- **Shift on Enter:** When `true`, applies the stat shift immediately upon entering the state—useful for enforcing an upfront resource cost so players cannot cancel out of a state early to dodge the cost.

---

### Temporary Passive Scope

<img width="901" height="457" alt="image" src="https://github.com/user-attachments/assets/c5cb32dc-d6ca-4193-9b13-09b7cf27a668" />

Temporarily grants or suspends specific Passives while the character is in this state (for example, granting a Knockback or Parry passive only during a defensive stance).

> **Initialization Note:** If used on a character's default starting `Idle` state, the initial state entry occurs right before starting passives are initialized, so the very first entry will be skipped. Every subsequent re-entry into the state will apply the scope normally.

- **Passive Scope Mode:**
  - `Temporary Add`: Adds the listed passives when entering the state and removes them upon exiting.
  - `Temp Remove`: Temporarily removes/suspends the listed passives while in the state and restores them upon exiting. Before removing a passive, the system checks whether the character actually had that passive active—if they did not, it will not blindly add it back on exit.
- **Passives:** The array of Passive assets to temporarily add or remove.

---

## Input-Based Sub Actions

### Input Animation Mapping

<img width="901" height="565" alt="image" src="https://github.com/user-attachments/assets/8b755db8-e16b-425b-a198-c26223266039" />

Drives state animations directly from live directional input—for example, playing a hovering idle animation when no input is held (`0, 0`) and switching to a directional movement animation when input is pressed.
- **Ignore Y Input:** Masks out vertical input when evaluating direction keys.
- **Ignore X Input:** Masks out horizontal input when evaluating direction keys.
- **Input Direction Mapping (Dictionary):** Maps an X/Y direction key (e.g., `(1, 0)` for right, `(-1, 0)` for left, `(0, 0)` for neutral) to the `SO_Animation` played when that direction is active.

---

### Sprite Face Input Direction

<img width="901" height="388" alt="image" src="https://github.com/user-attachments/assets/e673f20d-9789-41f1-9be0-7a80dfd6505d" />

Updates the sprite's facing direction based on directional input—ideal for mechanics like a steerable dash where the player can flip direction mid-dash.
- **Update Type (Flag Enum):** Controls which lifecycle phases update the sprite's facing direction (`Enter`, `Update`, `Exit`, `Everything`, or any combined flags).
- **Update X:** Flips the sprite horizontally based on X input.
- **Update Y:** Flips the sprite vertically based on Y input.


