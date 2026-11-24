# Passive Gates

**Passive Gates** control *when* and *under what conditions* a Passive is allowed to fire. Whenever a Gate's conditions are satisfied, it triggers its assigned **Passive Execution**—provided the pair is compatible (see the [Gate & Execution Compatibility Matrix](Passives.md#gate--execution-compatibility-matrix)).

Because Gates expose rich filtering rules (such as school flags, entity types, cooldowns, and positional checks), pairing a single Gate with different Executions can produce entirely distinct gameplay mechanics without custom code.

## Creating a Passive Gate

To create a Passive Gate asset:
1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. Navigate to **ModularCharacter -> Passives -> Gates**.

<img width="1052" height="771" alt="image" src="https://github.com/user-attachments/assets/8d5a2a98-b1d8-4895-8df1-e7d6bda58382" />

---

## Core Gate Slot

Every Passive Gate asset exposes one shared slot at the top:

<img width="853" height="295" alt="image" src="https://github.com/user-attachments/assets/8a793464-1ffe-4754-8436-4bf6a8f314c0" />

- **Execution:** The **Passive Execution** ScriptableObject triggered whenever this Gate fires. Every Gate requires an Execution assigned here.

---

## Included Passive Gates

### No Gate

A special pass-through gate that never triggers an execution event on its own—instead, it allows the assigned Execution to run its own autonomous lifecycle logic (such as continuous ticking or timed state monitoring).

---

### IFrame Gate

While most Gates listen to a single scoped lifecycle hook (such as taking damage or entering a state), the **IFrame Gate** bridges **two separate internal hooks** simultaneously so invulnerability frames can track triggers and intercept incoming damage cleanly.

- **Pairs Exclusively With:** The **IFrame** Execution. (This same multi-hook pattern is also available to programmers building hyper-specific custom passives.)

---

### On Active

Triggers its assigned Execution **once** immediately when the Passive is first applied to the character.

---

### On Collision Stay

<img width="851" height="538" alt="image" src="https://github.com/user-attachments/assets/f1a0bea1-d011-4914-8cfd-7d2f2f7c830b" />

Acts as a periodic spatial heartbeat rather than a one-time `OnTriggerEnter` check, repeatedly triggering its Execution at intervals while valid targets remain inside the collision area (ideal for auras such as Frost Aura or Healing Aura).
- **Internal Cooldown:** Interval (in seconds) between heartbeat pulses.
- **Apply to Self:** When `true`, the gate also runs an independent heartbeat check targeting the passive's owner—allowing an Execution like `Apply Passive` to affect the caster as well.
- **Collision:** Any Core Collision shape created via **Core -> Collision** in the Creation Hub (all shapes supported).
- **Target Alignment:** Faction filter (`Enemies`, `Friendlies`, etc.) specifying which entities inside the shape are valid targets.
- **Line of Sight:** Optional Line of Sight asset created via **Core -> Line of Sight** in the Creation Hub. If left `null`, every faction-valid target inside the collision shape is accepted; if assigned, targets must also pass a line-of-sight check.
  - **Tip:** Set the Line of Sight asset's distance to a large value—the collision shape already bounds the interaction range, so a large ray distance avoids guessing exact lengths per shape.

---

### On Damage Reaction

<img width="847" height="562" alt="image" src="https://github.com/user-attachments/assets/569fc120-8061-461b-b47a-3ae96a5fc87a" />

Triggers when damage occurs—either before mitigation, after taking damage, or after dealing damage.
- **School Elements:** Required elemental/magical school flags on the damage (`None` allows any school).
- **Entity Types:** Required Entity Type classification on the opposing character (`None` allows any entity type).
- **Reaction Type:**
  - `On_Pre_Damage_Taken`: Fires before incoming damage mitigation is finalized.
  - `On_Damage_Taken`: Fires after the character takes damage.
  - `On_Damage_Done`: Fires when the character deals damage to another target.
- **Chance:** Probability (`0.0` to `1.0`) that the gate triggers when conditions are met (`0.5` = `50%`, `1.0` = `100%`).
- **Luck Protection:** Bad-luck protection threshold (in failed attempts). For example, if **Chance** is `0.01` (`1%`) and **Luck Protection** is `100`, the gate guarantees a trigger on the `100th` attempt. Whenever the gate triggers—whether by luck or by reaching the cap—the counter resets to `0`.
- **Internal Cooldown:** Cooldown (in seconds) before the gate can evaluate and trigger again.

---

### On Get Damage

<img width="852" height="417" alt="image" src="https://github.com/user-attachments/assets/b564d4d6-3257-4552-b95c-fb14e71885af" />

Triggers whenever the character calculates outgoing damage—ideal for conditional damage bonuses (such as bonus damage against Beasts or backstab multipliers).
- **Entity Type:** Required Entity Type of the target receiving damage (`None` allows any entity type).
- **School Flags:** Required school flags on the outgoing damage (`None` allows any school).
- **Positional Requirement:** Spatial positioning required relative to the target (such as being behind the target).

---

### On Other Stat Change

<img width="848" height="488" alt="image" src="https://github.com/user-attachments/assets/50f3750b-a8d7-4bc6-8771-3f6954d25b85" />

Triggers whenever this character modifies a stat on **another** character (for example, triggering a bonus effect whenever this character heals an ally).

> **Important:** This gate evaluates the *attempted* stat modification amount, regardless of whether the target's stat actually changed. For example, healing a full-health ally for `10` Health still registers as a valid `10`-point modification even though their health stayed capped.

- **School Flags:** Required school flags on the stat change (`None` allows any school).
- **Stat To Watch:** The specific stat being modified on the other character (e.g., `Health`).
- **Stat Change Type:** Filters the direction of the modification (`Positive Only`, `Negative Only`, or `All`).
- **Internal Cooldown:** Cooldown (in seconds) between valid triggers.
- **Threshold:** Minimum modification magnitude required to trigger; changes below this value do not fire the Execution.

---

### On Stat Change Reaction

<img width="851" height="411" alt="image" src="https://github.com/user-attachments/assets/43fce114-57ea-4ad6-b8d3-53d6e724b075" />

Triggers whenever one of **this character's own stats** is about to change or finishes changing.
- **Stat:** The character stat to monitor.
- **School Flags:** Required school flags on the stat modification (`None` allows any school). For example, if a *Potion of Flaming Health* restores Health with the `Fire` school flag and this gate filters for `Fire`, the gate triggers.
- **On Stat Change Type:**
  - `On Pre Change`: Fires before the stat value is modified in the pipeline (used by pre-pipeline modifiers like `Stat Change Multiplier` and `Stat Stasis`).
  - `On Post Change`: Fires after the final post-pipeline value is calculated (used by post-pipeline reactions like `Mirror Stat Change`).

---

### Sequence Gate

<img width="845" height="636" alt="image" src="https://github.com/user-attachments/assets/b5a0fb85-9f45-4c9c-8573-b2f2d27ae2bb" />

Triggers when the character transitions along a specific state-to-state sequence (for example, transitioning from `Idle` -> `Dash`).
- **From:** Array of valid previous State Categories. Leave empty to allow transitions from *any* previous state.
- **To:** Array of valid destination State Categories. Leave empty to match *any* destination state. *(Note: Leaving both **From** and **To** empty prevents the gate from ever firing.)*
- **Internal Cooldown:** Delay (in seconds) before the gate can fire again.

---

### On State Entered

<img width="845" height="403" alt="image" src="https://github.com/user-attachments/assets/b54ca690-d728-4078-af6b-86962e693421" />

Triggers after the character enters a designated State Category a specified number of times.
- **Category:** The State Category to count entries for.
- **Number of Times Entered:** How many state entries are required before the gate fires.
- **Internal Cooldown:** Delay (in seconds) after triggering before the gate can begin counting again. *Note: Entries occurring while the cooldown is active do not increment the counter.*

---

### Timer Gate

<img width="846" height="362" alt="image" src="https://github.com/user-attachments/assets/ff04e4b2-03ad-4c1e-92c7-90e8a48ea348" />

Triggers periodically on a fixed timer interval while the passive is active.
- **Timer to Trigger Passive:** Interval duration (in seconds) between each trigger.

---

### While

<img width="852" height="390" alt="image" src="https://github.com/user-attachments/assets/a250a878-4524-4b07-a761-edbc1d2e561a" />

Repeatedly triggers at fixed intervals for as long as the character remains inside a specific State Category.
- **Category:** The State Category that must remain active.
- **Intervals:** Time interval (in seconds) between triggers while inside that state.

