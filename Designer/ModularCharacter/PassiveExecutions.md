
# Passive Executions

**Passive Executions** define *what* a Passive actually does when its Gate fires (or on its own autonomous lifecycle when paired with **No Gate**).

> **Compatibility Check:** Not every Execution works with every Gate—see the [Gate & Execution Compatibility Matrix](Passives.md#gate--execution-compatibility-matrix). Pairing an incompatible Gate and Execution will not throw console errors, but the Execution will simply never fire (or, for autonomous Executions like `Timed State Blueprint Revert`, will ignore the Gate and run on its own terms).

Just like Passives as a whole, Executions are not limited to buffs and debuffs—they act as isolated logic modules that inject core mechanics (such as damage mitigation, knockback, and invulnerability frames) directly into the character without hardcoding them inside the controller.

## Creating a Passive Execution

To create a Passive Execution asset:
1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. Navigate to **ModularCharacter -> Passives -> Executions**.

<img width="1175" height="767" alt="image" src="https://github.com/user-attachments/assets/065e8910-5cf2-49ff-b5d8-2b7fb792e55e" />

---

## Included Passive Executions

### Apply Passive

<img width="728" height="438" alt="image" src="https://github.com/user-attachments/assets/0da3f087-42c7-4417-99a1-7f2276f75617" />

Applies another Passive asset when triggered. When paired with spatial Gates like **On Collision Stay**, it applies the target Passive to each colliding target (including the owner if **Apply to Self** is enabled on the Gate).
- **Passive Description:** Custom tooltip/description text. While standard passives auto-generate descriptions from code, recursive passive-injecting executions require a manually written description here.
- **Passive to Apply:** The Passive asset granted when this Execution fires.

---

### Modify Stat Current

<img width="737" height="553" alt="image" src="https://github.com/user-attachments/assets/532736cf-92b1-4404-85a2-d653d65bf926" />

Modifies a character stat's current value (acting as a direct heal, resource gain, damage, or drain) and routes the change through the full stat pipeline.
- **Stat to Modify:** The target stat whose current value will change.
- **Stat to Scale Off Of:** Optional stat used to scale the modification amount.
- **Stat Scaling Type:** Reads either the scaling stat's **Max** value or **Current** value.
- **Scaling:** Scaling multiplier applied to the scaling stat (`0` disables stat scaling).
- **Flat Amount:** Base flat value added or subtracted. *Important:* **Flat Amount** acts as the sign driver—if **Flat Amount** is negative, the scaled portion is automatically treated as negative as well so the entire execution behaves as a drain/debuff.
- **School Flags:** Elemental/magical school flags attached to the modification as it passes through the stat pipeline.

---

### Grant Stat Immunity

<img width="728" height="446" alt="image" src="https://github.com/user-attachments/assets/d75bbc58-4d74-4740-9a0f-39a827efa12d" />

Temporarily or permanently grants immunity to buffs, debuffs, or both for a specific stat.
- **Duration:** Duration in seconds once triggered. Values `< 0` (default) keep the immunity active until the parent Passive itself is removed (still requires the Gate to trigger once first). Values `> 0` expire after the given duration and wait for the Gate to trigger again.
- **Stat to Grant Immunity To:** The stat protected by this immunity.
- **Immunity Grant Type:**
  - `Buff Immunity`: Blocks positive buff modifications to the stat.
  - `Debuff Immunity`: Blocks negative debuff modifications to the stat.
  - `Buff and Debuff Immunity`: Blocks both positive and negative modifications.

---

### IFrame

<img width="728" height="481" alt="image" src="https://github.com/user-attachments/assets/5a74478d-f985-44e1-a97c-b27cfb246b39" />

Grants invulnerability frames after taking damage and drives a sprite color-flicker visual effect.
- **Stat to Scale Duration Off Of:** Character stat read as the i-frame duration in seconds (snapshotted at the moment damage is received).
- **Color to Flicker To:** Alternate tint color used during the blink cycle (defaults to ~25% alpha transparency).
- **Blink Speed Multiplier:** Speed multiplier applied to elapsed time (`virtualClock = Repeat(currentTime * BlinkSpeedMultiplier, LoopDuration)`).
- **Loop Duration:** Total length (in seconds) of one complete blink cycle.
- **Half Split Threshold:** Duty-cycle cutoff within **Loop Duration**—while `virtualClock < HalfSplitThreshold`, the sprite renders with **Color to Flicker To**; once `virtualClock >= HalfSplitThreshold`, it renders as `Color.white`.

---

### Knock Back

<img width="920" height="770" alt="image" src="https://github.com/user-attachments/assets/07929638-7e55-4555-b628-7c4e77f799c2" />

Applies physical knockback impulse when the character takes damage, complete with a cumulative poise/damage threshold and state-override protection.
- **Knock Back Force:** Impulse force applied to push the character back.
- **Stat for Duration:** Character stat read as the knockback duration (snapshotted when knockback triggers).
- **Lock X / Lock Y:** Booleans that freeze knockback velocity along the X or Y axis. *(Note: Enabling both logs a warning because locking both axes freezes the character in place.)*
- **States to Overpower Knock Back:** State Categories that ignore or cancel incoming knockback (for example, listing `Dash` so dashing pushes straight through knockback).
- **Damage Threshold:** Cumulative damage required before knockback triggers.
- **Threshold Recovery Interval:** Tick rate (in seconds) at which accumulated threshold damage decays (`1` = default; higher decays faster, lower decays slower).
- **Threshold Recovery Per Tick:** Amount of accumulated threshold damage recovered on each tick.

---

### Change Master Blueprint State

<img width="911" height="337" alt="image" src="https://github.com/user-attachments/assets/4085161c-df92-4b58-b5ad-6cd3267fbd82" />

Replaces a state slot inside the character's **Master Blueprint** layer. Internally, a character tracks three state layers:
1. **Active States:** Live runtime states (which can be temporarily swapped or injected during gameplay).
2. **Master Blueprint States:** The active baseline configuration that states fall back to.
3. **Original Base Blueprint:** The immutable source asset (never altered unless **Change Master Blueprint** is explicitly executed).
- **State:** The new State ScriptableObject written into the Master Blueprint's matching State Category slot.

---

### Mirror Stat Change

<img width="917" height="397" alt="image" src="https://github.com/user-attachments/assets/e9f7e990-9eca-45d6-aa21-66189d9ba4e3" />

Mirrors a percentage of an incoming stat change onto a target stat—used by passives like **Greedy Gluttony** (which mirrors health changes back onto health to double both healing and damage).

> **Why There Is No "Stat to Watch" Field Here:**
> Because Executions are decoupled from Gates, **Mirror Stat Change** relies entirely on its paired **On Stat Change Reaction Gate** to filter *which* stat is being watched. Duplicating a stat filter inside the Execution could create a configuration mismatch (such as a Gate watching `Health` while the Execution expects `Mana`, preventing the effect from ever firing).

- **Percentage to Mirror:** Multiplier applied to the incoming stat change (`1` = `100%`, capped at `5` = `500%`).
- **Stat to Mirror To:** The destination stat that receives the mirrored change.
- **Mirror Type:** Filters whether to mirror `Positive Only`, `Negative Only`, or `All` stat changes.

---

### On Damage Reaction Gain Stat

<img width="1012" height="550" alt="image" src="https://github.com/user-attachments/assets/e432c48e-eb55-4245-b7f9-53961c266787" />

Restores or adds to a target stat's **current** value when dealing or taking damage—acting like Lifesteal, Mana Leech, or Rage-on-Hit depending on the chosen stat.
- **School Flags:** Elemental/magical school flags sent through the stat pipeline.
- **Stat to Gain:** The stat whose current value increases.
- **Percentage of Damage to Gain:** Percentage of the triggering damage converted into stat gain (`0.5` = `50%`, capped at `5` = `500%`).
- **Stat to Scale Off Of:** Optional character stat used to add bonus scaling gain.
- **Stat Scaling:** Multiplier applied to **Stat to Scale Off Of** (`0` disables stat scaling for flat/percentage-only gain).
- **Flat Amount to Gain:** Flat base amount gained per trigger. *(Tip: Combining a low flat amount with high stat scaling lets the gain grow dynamically as the character's stats increase.)*

---

### On Damage Reaction Reprisal

<img width="1011" height="626" alt="image" src="https://github.com/user-attachments/assets/13e2abc7-f0fd-42c5-a5cc-c252c1ab7a14" />

Deals retaliatory stat damage or healing back to the opposing character (to the attacker when taking damage, or to the defender when dealing damage)—ideal for Thorns or elemental counter-effects.

> **Sign Consistency Warning:** All numeric values below (**Flat Amount**, **Damage Percentage**, and **Self Stat Scaling Amount**) should share the same sign (all positive or all negative)—mixing positive and negative values can produce unintended results.

- **School Flags:** School flags sent through the target's stat pipeline so resistances and passives account for the reprisal.
- **Target Stat to Modify:** Which stat is modified on the opposing character (e.g., `Health`).
- **Flat Amount:** Flat base modification (positive or negative).
- **Damage Percentage:** Percentage of the triggering damage reflected (`0.5` = `50%`, `-0.5` = `-50%`).
- **Stat to Scale Off Of:** Which stat on **this character** scales the reprisal amount.
- **Self Stat Scaling Amount:** Multiplier applied to **Stat to Scale Off Of**.

---

### On Damage Taken Multiplier

<img width="1010" height="375" alt="image" src="https://github.com/user-attachments/assets/d277b545-fa47-43d2-b23c-0013985613f2" />

Adjusts incoming damage inside the additive damage-taken modifier pool. Because the damage pipeline operates as an additive tug-of-war, `-2.0` (`-200%` damage taken) combined with `+2.0` (`+200%` damage taken) nets out to `0%` change (dealing standard `100%` base damage).
- **Multiplier:** Additive modifier (`-0.5` = `-50%` damage taken, `0.5` = `+50%` damage taken).

---

### On Get Damage Multiplier

<img width="1007" height="371" alt="image" src="https://github.com/user-attachments/assets/4843536d-383c-498e-8a27-abf34a3a017a" />

Multiplies outgoing damage before it is sent to the target.
- **Multiplier:** Outgoing damage multiplier (positive values increase damage; negative values invert it).

### Grant Passive Immunities

<img width="891" height="458" alt="image" src="https://github.com/user-attachments/assets/694a3bba-fed3-41d3-a910-6e90972ace16" />

Grants immunity to incoming Passives matching specific School or Mechanic flags once triggered.

> **Flag Configuration Note:** Because immunities validate against each incoming Passive Blueprint's **School Flags** and **Mechanic Flags**, those flags must be properly configured on your Passive assets—otherwise an incoming debuff (such as a Slow missing its mechanic flag) will not be blocked.

- **Duration:** Duration in seconds once triggered. Values `< 0` remain active until the parent Passive itself is removed (still requires the Gate to trigger once first).
- **School Flags:** Elemental/magical school flags blocked by this immunity (e.g., `Fire` blocks Fire DoTs).
- **Mechanic Flags:** Gameplay mechanic flags blocked by this immunity (e.g., `Slow`, `Stun`, `DoT`).

---

### Stat Change Multiplier

<img width="888" height="368" alt="image" src="https://github.com/user-attachments/assets/b8e5de1c-c1dc-4671-a393-d4d499ef5d36" />

Multiplies the magnitude of an incoming stat change as it passes through the stat pipeline. Like **Mirror Stat Change**, it has no stat-watcher field because its paired **On Stat Change Reaction Gate** already filters which stat is monitored.
- **Multiplier:** Multiplier applied to the incoming stat change (positive or negative to amplify, reduce, or invert the change).

---

### Stat Damage Scaling

<img width="887" height="480" alt="image" src="https://github.com/user-attachments/assets/bae13613-1b70-48f5-bf99-15fef2396d0e" />

Provides smooth, diminishing-returns damage mitigation or vulnerability driven by a character stat (such as Armor, Fire Resistance, or Frost Resistance) without hardcoding resistance formulas in the controller.
- **Scaling Mode:**
  - `Mitigation`: Reduces incoming damage using the scaling formula.
  - `Vulnerability`: Inverts the calculation to increase incoming damage by the calculated percentage.
- **Scaling Stat:** The character stat read for the calculation (e.g., `Armor`).
- **Scaling Const:** Diminishing-returns curve constant calculated as `Reduction = Stat / (Stat + Scaling Const)`. When **Scaling Stat** equals **Scaling Const**, damage reduction reaches exactly `50%` (for example, with `Scaling Const = 100`, `100` Armor grants `50%` mitigation, and `200` Armor grants `66.7%` mitigation).
- **Max Scaling Cap:** Maximum percentage cap allowed (`0` = uncapped).

---

### Stat Step Damage Scaling

<img width="890" height="640" alt="image" src="https://github.com/user-attachments/assets/cdb197cd-5f8d-4506-922d-6fc76aa405c8" />

Applies linear or threshold-stepped damage mitigation/vulnerability based on how many increments of **Scaling Const** a character's stat reaches (`Steps = Stat / Scaling Const`, `Total Fraction = Steps * Percentage Per Step`).
- **Damage Scaling Mode:** `Mitigation` (reduces incoming damage) or `Vulnerability` (increases incoming damage).
- **Stat Scaling:** The character stat used for the step calculation.
- **Percentage Per Step:** The percentage of mitigation or vulnerability gained for every completed step (e.g., `0.05` = `5%` per step).
- **Scaling Const:** The stat amount required to equal `1` full step (for example, if `Scaling Const = 10` and `Percentage Per Step = 0.05`, every `10` points of the stat grants `5%` scaling).
- **Max Scaling:** Maximum total percentage cap allowed (`0` = uncapped).
- **Snap to Floor Breaking Point:** When `true`, rounds steps down (`Mathf.Floor`) so partial progress toward the next threshold grants zero bonus until a full breakpoint is reached (e.g., `19` stat with `Scaling Const = 10` counts as `1` step instead of `1.9` steps). When `false`, fractional progress scales smoothly between thresholds.

---

### Change State

<img width="891" height="376" alt="image" src="https://github.com/user-attachments/assets/e0a0837f-061a-4862-b116-416bad0f3e84" />

Forces the character into a specific State ScriptableObject, automatically injecting that state into its matching State Category slot in the active state table before transitioning.
- **State Name:** Display label used when auto-generating the Passive's description in code.
- **State to Change To:** The State ScriptableObject to inject and enter.

---

### Apply Stat Modifiers

<img width="883" height="522" alt="image" src="https://github.com/user-attachments/assets/4fcc7a04-2ea0-4a03-90a6-89d6eade0261" />

Applies a flat and/or percentage buff or debuff to a target stat.
- **Stat to Manipulate:** The target stat modified by this buff/debuff.
- **Percentage Manipulation:** Percentage modifier (`0` = no percentage change; can be positive or negative).
- **Flat Manipulation:** Flat modifier (`0` = no flat change; can be positive or negative, and can even oppose **Percentage Manipulation**—such as `-2` flat speed paired with `+50%` speed).
- **Duration:** Duration in seconds once triggered (`-1` default = lasts until the parent Passive is removed). Values `> 0` count down after triggering and remove the stat modifier upon expiring—*note that this only removes the stat modifier itself, leaving Passive removal to the Passive Exit*.

---

### Stat Over Time

<img width="882" height="806" alt="image" src="https://github.com/user-attachments/assets/ba9e1a1b-a160-4c47-8f25-6a9383548bd8" />

Applies periodic stat regeneration or drain (such as MP5 mana regen or a poison DoT) on its own autonomous internal timer, ignoring Gates.

> **Sign Consistency Warning:** **Flat Amount Per Tick** and **Stat Scaling Percentage** must share the same sign (both positive for gain/regen, or both negative for drain/damage)—do not mix positive and negative values.

- **Stat to Modify:** The target stat modified each tick.
- **Tick Intervals:** Time (in seconds) between each tick.
- **Flat Amount Per Tick:** Base flat stat change applied on every tick.
- **Stat Scaling Percentage:** Percentage of the target stat added per tick (`1` = `100%`, `0.5` = `50%`).
- **Scaling Type:**
  - `Max Value`: Scales off the stat's maximum value for consistent ticks.
  - `Current Value`: Scales off the stat's current value (ticks grow stronger as the current stat rises, or weaker as it drops).
- **States to Pause On:** State Categories during which the internal tick timer pauses, automatically resuming once the character leaves those states.

---

### Stat Stasis

<img width="891" height="317" alt="image" src="https://github.com/user-attachments/assets/256b9616-aaf6-4072-9c0e-ad9905724127" />

Freezes a stat by intercepting and halting all incoming changes while active, then releasing the accumulated balance through the full stat pipeline when the stasis ends. Because its paired **On Stat Change Reaction Gate** filters which stat is frozen, this Execution only configures the release school flags.
- **School Flags:** Elemental/magical school flags assigned to the accumulated stat change when stasis ends and the banked value passes through the stat pipeline (so active buffs/debuffs scale the released total—for example, `100` banked Mana debt released while under a `2x` Mana cost debuff becomes `200` Mana debt).

---

### Timed State Blueprint Revert

<img width="890" height="705" alt="image" src="https://github.com/user-attachments/assets/3e2e4c92-119b-4083-afb6-769305665f2e" />

An autonomous timer Execution (ignores Gates) that monitors a hot-swapped state and automatically reverts that slot back to the **Master Blueprint** once the character stays out of the watched state for a set duration (see the included **ProgressiveTripleJump** passive for an example).
- **State to Watch:** The State Category monitored by this timer. Whenever the character is *not* in this state, the revert countdown ticks down.
- **Time to Revert:** Duration (in seconds) outside **State to Watch** before the slot reverts back to the Master Blueprint's version of that state.
- **States That Pause Timer:** State Categories that pause the revert countdown for a single transition when entered directly from **State to Watch** (e.g., pausing a jump-combo revert window while mid-air in `Ascend` or `Descend` so the countdown only runs once the character lands).
- **States That Force Reset:** State Categories that immediately cancel the timer and revert the state back to the Master Blueprint on entry (for example, listing a `Stunned` state so getting hit immediately resets an active attack combo chain).










---

## Shared

> [!IMPORTANT]
> Shared components require the **Shared Character** setup, which bridges decoupled systems—such as combat, inventory, damageables, weapons, and currency—into a unified character without coupling their underlying assemblies. In order to use any of the items below, you **must** use the Shared system.

### Execute Weapon

<img width="772" height="390" alt="image" src="https://github.com/user-attachments/assets/6b901e53-2060-4a6d-87cf-a9fffc881cd8" />

Executes a Multi-Weapon when the passive triggers, allowing designers to stack as few or as many weapons as needed.

#### Variables
* **Weapon:** The Multi-Weapon executed when this passive triggers.
* **Weapon Name:** The display name injected when programmers retrieve the programmatically assembled description for this passive.


