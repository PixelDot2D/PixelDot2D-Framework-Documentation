# Passives

In the Modular Character framework, **Passives** are far more than simple stat buffs or debuffs—they are self-contained, data-driven gameplay abilities and reactive behaviors.

Rather than hardcoding core mechanics like armor mitigation, knockback, or invulnerability frames directly into the character controller, Passives encapsulate that logic as modular assets. For example, instead of hardcoding physical armor, the framework provides a damage-scaling passive driven by a target stat—allowing you to reuse that exact same passive architecture for Physical Armor, Fire Resistance, or Frost Resistance (each with its own scaling curve) without writing a single line of C#.

## Exploring the Included Demo Passives

The framework ships with **35+ pre-built Passives** under `PixelDot2D/ModularCharacter/SO/Passives` to study and reuse, including:
- **Greedy Gluttony:** Doubles all health modifications (both positive healing and negative damage).
- **Health Stasis:** Blocks all modifications to the Health stat, applies it as a debt, then when removed, applies the debt to the character.
- **Frost Aura:** Periodically slows characters inside a Core collision shape using the given filters.
- **Healing Aura:** Periodically heals characters inside a collision shape using the given filters.
- **I-Frames & Knockback:** Core combat reaction behaviors implemented purely as starting passives rather than hardcoded controller logic.
- **Scaredy Cat:** Grants a temporary speed buff whenever the character takes Shadow damage.
- **Painful Dash:** Deals `1` damage as Fire, Water, and Lightning when triggered.

---

## Anatomy of a Passive (Gate -> Execution -> Exit)

Every Passive follows the same three-part modular structure used throughout the framework:
1. **Passive Gate:** Determines *when* or *under what condition* the passive activates (e.g., on taking damage, on stat change, or always active via **No Gate**).
2. **Passive Execution:** Defines *what* the passive actually does when triggered (e.g., modifying a stat, applying knockback, granting an immunity, or swapping a state).
3. **Passive Exit:** Determines *when* the passive expires and cleans itself up (e.g., duration timer, charge count, or permanent).

> **Important — Gate & Execution Compatibility:**
> While **Passive Exits** are completely self-contained and work with every Execution, **not every Passive Gate is compatible with every Passive Execution**. Combining an incompatible Gate and Execution will not throw console errors, but the Execution will simply never fire because it expects event data that only specific Gates provide.

---

## Creating a Passive (Recommended Workflow)

While you can create individual passive pieces manually via **ModularCharacter -> Passives** in the Creation Hub, using the **Bulk Creation Macro** is recommended so your folders stay organized:

1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. Navigate to **Macros -> ModularCharacter -> Bulk Creation -> Build Passive Matrix**.

<img width="1225" height="761" alt="image" src="https://github.com/user-attachments/assets/f2c2c973-3e0f-4af9-b2b9-a935214063b1" />

This macro generates a dedicated folder containing empty **Gate**, **Execution**, and **Exit** subfolders alongside a **Passive Blueprint** ready for you to populate in the Editor.

---

## Anatomy of a Passive Blueprint

Once you create a Passive Blueprint (either via **Build Passive Matrix** or manually under **ModularCharacter -> Passives**), inspect the asset to configure its UI metadata, immunity flags, and lifecycle modules:

<img width="888" height="636" alt="image" src="https://github.com/user-attachments/assets/e207eba0-5aa0-4ebf-b1ba-94abfe0f0dbe" />

- **Icon:** Sprite reference exposed for UI systems (such as buff/debuff bars or tooltips).
- **Passive Name:** Display name pulled by UI systems when rendering the passive.
- **Aura Flags:** Classifies the passive's aura behavior (e.g., `Buff`, `Debuff`, `Permanent Buff`, `Permanent Debuff`).
  - *Note:* Passives flagged as **Permanent** are protected by the internal Passive Manager and will not be stripped by standard removal/cleanse calls unless explicitly forced.
- **School Flags:** Elemental or magical school tags (e.g., `Fire`, `Frost`, `Shadow`) checked by Character Blueprint **School Immunities** and passive immunity filters.
- **Mechanic Flags:** Gameplay mechanic tags (e.g., `Slow`, `Stun`, `DoT`, `HoT`) checked by Character Blueprint **Mechanic Immunities** and passive immunity filters.
- **Gate:** The **Passive Gate** ScriptableObject that controls when this passive triggers.
- **Exit:** The **Passive Exit** ScriptableObject that determines when this passive expires and removes itself.

> **Why Is There No Execution Slot on the Passive Blueprint?**
> The **Passive Execution** asset is slotted directly inside the **Passive Gate** rather than on the Passive Blueprint itself—because the Gate owns and triggers the Execution whenever its conditions are met.


### Applying Passives
- **Starting Passives:** Drag any Passive asset directly into a Character Blueprint's **Starting Passives** array.
- **Runtime Application:** Apply passives dynamically through other passives (e.g., `Apply Passive`), abilities, or C# code. There is no limit to how many passives a character can hold simultaneously.

---

## Gate & Execution Compatibility Matrix

Use this quick-reference matrix when pairing **Passive Executions** with **Passive Gates**. (For detailed parameter breakdowns of each module, see the dedicated Passive Gates, Executions, and Exits pages.)

> **Note on `No Gate`:** `No Gate` does **not** mean leaving the Gate slot `null`—there is an explicit **No Gate** asset in the Creation Hub that keeps the passive continuously active without a conditional trigger.

| Passive Execution | Compatible Passive Gate(s) | Notes & Constraints |
| :--- | :--- | :--- |
| **Apply Passive** | All Gates | Compatible with every Gate. |
| **Apply Stat Modifiers** | All Gates | Compatible with every Gate. |
| **Change Master Blueprint State** | **No Gate** only | Must use the **No Gate** asset. |
| **Change State** | All Gates | Compatible with every Gate. |
| **`[Shared]` Execute Weapon** | All Gates | Requires the **Shared** character template (fails on standard Modular Characters). See **Shared** docs. |
| **Grant Passive Immunities** | All Gates | Compatible with every Gate. |
| **Grant Stat Immunity** | All Gates | Compatible with every Gate. |
| **IFrame** | **IFrame Gate** only | Requires the dedicated **IFrame Gate**. |
| **Knock Back** | **On Damage Reaction Gate** only | Must set Reaction Type to **`On Damage Taken`**. |
| **Mirror Stat Change** | **On Stat Change Reaction Gate** only | Must set On Stat Change Type to **`On Post Change`**. Mirrors the final post-pipeline value without re-triggering the stat pipeline (preventing infinite recursion). |
| **Modify Stat Current** | All Gates except **On Stat Change Reaction Gate** | Compatible with every Gate except **On Stat Change Reaction Gate**. |
| **On Damage Reaction Gain Stat** | **On Damage Reaction Gate** only | Works with all Reaction Types. |
| **On Damage Reaction Reprisal** | **On Damage Reaction Gate** only | Works with all Reaction Types. |
| **On Damage Taken Multiplier** | **On Damage Reaction Gate** only | Must set Reaction Type to **`On_Pre_Damage_Taken`**. |
| **On Get Damage Multiplier** | **On Get Damage Gate** only | Requires the **On Get Damage Gate**. |
| **Stat Change Multiplier** | **On Stat Change Reaction Gate** only | Must set On Stat Change Type to **`On Pre Change`**. |
| **Stat Damage Scaling** | **On Damage Reaction Gate** only | Must set Reaction Type to **`On Pre Damage Taken`**. |
| **Stat Over Time** | **No Gate** only | Must use the **No Gate** asset. |
| **Stat Stasis** | **On Stat Change Reaction Gate** only | Must set On Stat Change Type to **`On Pre Change`**. |
| **Stat Step Damage Scaling** | **On Damage Reaction Gate** only | Must set Reaction Type to **`On Pre Damage Taken`**. |
| **Timed State Blueprint Revert** | **No Gate** only | Must use the **No Gate** asset. |



