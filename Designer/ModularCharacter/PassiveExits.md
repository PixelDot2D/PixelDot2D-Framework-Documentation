# Passive Exits

Passive Exits control when a passive removes itself from a character. Unlike Passive Gates and Passive Executions—where Gates communicate state to Executions—Passive Exits operate completely standalone and do not use a compatibility matrix.

Passive Exits are not limited to simple timers. They can persist indefinitely for equipment and racial traits, monitor combat and stat conditions (such as dealing 200 Fire damage to Beasts), or remove a passive after a fixed duration.

To create a Passive Exit asset, press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**, ModularCharacter -> Passives -> Exits then select the desired Passive Exit Asset.

<img width="1150" height="745" alt="image" src="https://github.com/user-attachments/assets/fb4b2a8a-91c4-4543-a88b-a2692e5aa22d" />

---

## Included Passive Exits

### No Exit

The passive persists indefinitely until removed manually via C# code (see the Programmer documentation for removal constraints and API details). This asset contains no configurable Inspector fields.

---

### On Damage Reaction Exit

A reactive exit that tracks damage dealt or received by the character. This is useful for conditional buffs or rogue-lite starting traits that break under specific combat conditions (e.g., a starting passive that is removed upon taking damage from an Undead enemy).

<img width="853" height="512" alt="image" src="https://github.com/user-attachments/assets/57de8ebd-7e4b-4fd0-a8d7-aa429158ca77" />

**Parameters**
- **School Elements:** Flag mask of damage schools to react to. Leave set to `None` to react to any school.
- **Entity Types:** Flag mask of target or attacker entity types (e.g., Beasts, Undead, Elves) to react to. Leave set to `None` to react to any entity type.
- **Reaction Type:** Determines which damage event accumulates toward the threshold (single selection):
  - **On Damage Taken:** Accumulates final damage received after mitigations.
  - **On Damage Done:** Accumulates damage dealt by this character.
  - **On Pre Damage Taken:** Accumulates raw incoming damage before mitigations are applied.
- **Damage Threshold:** The cumulative damage required to remove the passive. Set to `0` to trigger on any valid damage instance regardless of amount. There is no upper ceiling.
- **Internal Cooldown:** Delay in seconds between valid damage accumulations. Defaults to `-1` (no cooldown). Setting a small value (e.g., `0.1`) prevents multi-hit single-frame bursts from immediately exhausting the threshold.

---

### On State Enter

Monitors character state transitions and removes the passive once a target state has been entered a specified number of times.

<img width="1067" height="411" alt="image" src="https://github.com/user-attachments/assets/840c6af7-fb7b-4a8a-8df0-77986c466483" />

**Parameters**
- **Category:** The state category to monitor for entry.
- **Number of Times Entered to Trigger:** Total state entries required to remove the passive (e.g., entering a Hit Stun state `3` times). Set to `0` to remove the passive immediately upon entering the state.
- **Internal Cooldown:** Delay in seconds between counted state entries. Defaults to `-1` (no cooldown). Useful for preventing rapid or spammable states (such as Dash) from triggering multiple counts too quickly.

---

### On Stat Threshold

Removes the passive when a monitored stat satisfies a comparison against a flat numeric threshold. For example, granting +300 Strength until Mana reaches `200`.

<img width="1072" height="432" alt="image" src="https://github.com/user-attachments/assets/6495a935-59c8-4675-bb1b-01a60cd5e717" />

**Parameters**
- **Value Type:** Determines whether to evaluate the stat's Current or Max value:
  - **Current Value:** Evaluates the active value, which reflects buffs and debuffs and can exceed the base maximum (e.g., `150` current Strength on a `100` base stat).
  - **Max Value:** Evaluates the unbuffed maximum value of the stat, unaffected by active modifiers.
- **Stat:** The stat to monitor (e.g., Health, Speed, Strength, Mana).
- **Comparison Type:** The operator used to evaluate the stat against the threshold (`Greater Than`, `Less Than`, `Greater Than Or Equal To`, `Less Than Or Equal To`, `Equal To`).
  > [!TIP]
  > Avoid using **Equal To** for stats modified by floating-point math, as exact precision matches are unreliable. Use **Greater Than Or Equal To** or **Less Than Or Equal To** instead.
- **Threshold:** The target value evaluated using the selected **Comparison Type** (`Stat [Comparison Type] Threshold`).

---

### Exit On Stat Percentage

Removes the passive when a stat's current value reaches a specified percentage of its maximum value. For example, removing a low-health Rage buff once Health rises back above `25%`.

<img width="1070" height="397" alt="image" src="https://github.com/user-attachments/assets/a1c9c76d-c336-48fc-852d-84f237bc1b79" />

**Parameters**
- **Stat:** The stat to monitor.
- **Comparison Type:** The operator used to evaluate the current percentage (`Greater Than`, `Less Than`, `Greater Than Or Equal To`, `Less Than Or Equal To`, `Equal To`). Prefer inclusive range checks over **Equal To** for floating-point reliability.
- **Percentage:** Normalized target ratio from `0` to `1` (`0.25` = `25%`, `1` = `100%`).

---

### On Change Stat Other

Removes the passive when this character modifies a specific stat on another character. Cumulative changes are tracked using the raw delta applied, even if the target's stat is already clamped at `0` or its maximum (e.g., healing `10` Health on a target already at `100/100` Health still records `10` toward the threshold).

<img width="1065" height="392" alt="image" src="https://github.com/user-attachments/assets/5881ddbb-9fcc-43a3-91f5-457098100861" />

**Parameters**
- **School Flags:** Flag mask of schools allowed to count as valid stat changes.
- **Stat to Watch For:** The specific stat on the target character to monitor.
- **Threshold:** Cumulative stat change required to remove the passive. Use negative values (e.g., `-5`) to track stat reductions and positive values (e.g., `5`) to track stat increases.

---

### Exit Timer

Standard duration-based exit that removes the passive after a set amount of time.

<img width="1067" height="322" alt="image" src="https://github.com/user-attachments/assets/6c2cc2ce-8dd4-444b-915d-99caa2046f42" />

**Parameters**
- **Duration:** Time in seconds before the passive is automatically removed.
  > [!NOTE]
  > When pairing **Exit Timer** with periodic gates (such as a Timer Gate), add a small buffer to **Duration** so the passive does not expire right as the final tick fires. For example, if a passive deals `2` damage every `2` seconds for `5` ticks (`10` seconds total), set **Duration** to `11` seconds to guarantee all `5` cycles complete before removal.
