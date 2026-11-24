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


---

<img width="932" height="1021" alt="image" src="https://github.com/user-attachments/assets/7d20209a-d269-4ba3-90a3-5ce72ce2fc28" />

---

