# PixelDot2D Framework Documentation

Official documentation for the PixelDot2D modular Unity framework.

> [!IMPORTANT]
> **Documentation Overhaul (Work in Progress)**
> 
> These docs are currently undergoing a major rewrite and **do not reflect the current live version on the Unity Asset Store**. 
> 
> The upcoming update restructures the framework internally and introduces breaking changes with two main goals:
> - **Designer-First Workflows:** Build complete 2D characters, AI, weapons, items, and reactive behaviors directly in the Unity Editor by assembling modular, "Lego-like" ScriptableObjects—without writing code.
> - **Decoupled Factory & Flyweight Extensions:** Core extension points have been refactored from assembly-bound registries into Factory and Flyweight patterns, allowing programmers to create custom modules inside their own `.asmdef` boundaries while keeping full visibility of their game code without modifying framework files.

## Required Setup & Critical Rules

Before configuring any modules, ensure your project meets these foundational requirements (also located in `REQUIRED_SETUP.txt` inside the package root):

- **Unity Version (`Unity 6.6+` Required):** This framework requires **Unity 6.6 or newer** and is not compatible with earlier Unity releases.
- **Layer & Enum Sync (`Enum_EditorLayers.cs`):** Enum integer values must match your Unity Project Settings layer indices exactly (e.g., if `Player` is Layer `8`, `Enum_EditorLayers.Player = 8`) and remain sequential (`0, 1, 2, 3...`) with no gaps so bit-shifting and array indexing stay deterministic.
- **Right-Facing Vector Orientation (`+X` Origin):** All sprites, sprite sheets, and animations must be authored facing **Right** (`0°` along the positive X-axis) with `SpriteRenderer.flipX = false` by default. Directional flipping is handled purely as a visual inversion so underlying physics and angle math remain undisturbed.

---

## Designer Documentation

*Setup, Inspector configuration, and minimal copy-paste wiring.*


* **Core**
  * [Summary](Designer/Core/Summary.md)
  * [Animation](Designer/Core/Animation.md)
  * [Input](Designer/Core/Input.md)
  * [Collision](Designer/Core/Collision.md)
  * [Line Of Sight](Designer/Core/LineOfSight.md)
  * [Sensor](Designer/Core/Sensor.md)
  * [RB2DMover](Designer/Core/RB2DMover.md)
  * [RB2D Movements](Designer/Core/RB2DMovements.md)
  * [SaveAndLoading](Designer/Core/SaveAndLoading.md)
* **Combat**
  * [Summary](Designer/Combat/Summary.md)
  * [Weapon Modules Virtual Transform](Designer/Combat/WeaponModules.md)
  * [Weapon Modules Aiming](Designer/Combat/WeaponModulesAiming.md)
  * [Weapon Modules Gates](Designer/Combat/WeaponModulesGates.md)
  * [Weapon Modules Executions - Projectiles](Designer/Combat/Projectiles.md)
  * [Weapon Modules Executions - Hitbox](Designer/Combat/Hitbox.md)
  * [Weapon Modules Executions - Hitscans](Designer/Combat/Hitscans.md)
  * [Template Combat Entity and Prefabs](Designer/Combat/TemplateCombatEntityAndPrefabs.md)
* **Modular Character**
  * [Summary](Designer/ModularCharacter/Summary.md)
  * [Character Blueprint](Designer/ModularCharacter/CharacterBlueprint.md)
  * [States](Designer/ModularCharacter/States.md)
  * [Scripted Main Actions](Designer/ModularCharacter/ScriptedMainActions.md)
  * [Scripted Sub Actions](Designer/ModularCharacter/ScriptedSubActions.md)
  * [Scripted Exit Actions](Designer/ModularCharacter/ScriptedExitActions.md)
  * [Scripted Gates](Designer/ModularCharacter/Gates.md)
  * [Passives](Designer/ModularCharacter/Passives.md)
  * [Passive Gates](Designer/ModularCharacter/PassiveGates.md)
  * [Passive Executions](Designer/ModularCharacter/PassiveExecutions.md)
  * [Passive Exits](Designer/ModularCharacter/PassiveExits.md)
  * [Prefabs and Templates](Designer/ModularCharacter/PrefabsAndTemplates.md)
* **Items**
  * [Summary](Designer/Items/Summary.md)
  * [Items Creation](Designer/Items/ItemsCreation.md)
  * [Crafting](Designer/Items/Crafting.md)
  * [Drop Tables](Designer/Items/DropTables.md)
  * [Merchant](Designer/Items/Merchant.md)
* **Shared**
  * [Summary](Designer/Shared/Summary.md)
  * [Templates](Designer/Shared/Templates.md)

---

## Programmer Documentation
*Architecture internals, lifecycle contracts, extension points.*

* **Core**
  * [Summary](Programmer/Core/Summary.md)
  * [Animation](Programmer/Core/Animation.md)
  * [Extensions](Programmer/Core/Extensions.md)
  * [Input](Programmer/Core/Input.md)
  * [Object_Pool](Programmer/Core/Object_Pool.md)
  * [Collision](Programmer/Core/Collision.md)
  * [LineOfSight](Programmer/Core/LineOfSight.md)
  * [Sensors](Programmer/Core/Sensors.md)
  * [Profiling](Programmer/Core/Profiling.md)
  * [RB2DMover](Programmer/Core/RB2DMover.md)
  * [SavingAndLoading](Programmer/Core/SavingAndLoading.md)
  * [ComponentCache](Programmer/Core/ComponentCache.md)
* **Combat**
  * [Summary](Programmer/Combat/Summary.md)
  * [Weapon Manager vs WeaponizedModule](Programmer/Combat/WeaponManagerVsWeaponizedModule.md)
  * [Weapons - Virtual Transform](Programmer/Combat/WeaponsVirtualTransform.md)
  * [Weapons - Aiming](Programmer/Combat/WeaponsAiming.md)
  * [Weapons - Gates](Programmer/Combat/WeaponsGates.md)
  * [Weapons - Execution](Programmer/Combat/WeaponsExecution.md)
  * [Combat Sequencer](Programmer/Combat/CombatSequencer.md)
  * [Example Template](Programmer/Combat/ExampleTemplate.md)
* **Modular Character**
  * [Summary](Programmer/ModularCharacter/Summary.md)
  * [ModularCharacterController](Programmer/ModularCharacter/ModularCharacterController.md)
  * [Modular Character Blueprint](Programmer/ModularCharacter/ModularCharacterBlueprint.md)
  * [Dedicated States](Programmer/ModularCharacter/DedicatedStates.md)
  * [Scripted State](Programmer/ModularCharacter/ScriptedState.md)
  * [Scripted State - Main Action](Programmer/ModularCharacter/ScriptedStateMainAction.md)
  * [Scripted State - Sub Action](Programmer/ModularCharacter/ScriptedStateSubAction.md)
  * [Scripted State - Gate](Programmer/ModularCharacter/ScriptedStateGate.md)
  * [Scripted State - Exit](Programmer/ModularCharacter/ScriptedStateExit.md)
  * [Passive Manager](Programmer/ModularCharacter/PassiveManager.md)
  * [Passives - Gates](Programmer/ModularCharacter/PassivesGates.md)
  * [Passives - Executions](Programmer/ModularCharacter/PassivesExecutions.md)
  * [Passives - Exits](Programmer/ModularCharacter/PassivesExits.md)
  * [Passives - Compatibility List](Programmer/ModularCharacter/PassivesCompatibilityList.md)
  * [StateGates](Programmer/ModularCharacter/StateGates.md)
  * [Interactables](Programmer/ModularCharacter/Interactables.md)
* **Items**
  * [Summary](Programmer/Items/Summary.md)
  * [Crafting](Programmer/Items/Crafting.md)
  * [Drop Tables](Programmer/Items/DropTables.md)
  * [Inventory](Programmer/Items/Inventory.md)
  * [Item Components](Programmer/Items/ItemComponents.md)
  * [Merchant](Programmer/Items/Merchant.md)
  * [Interfaces](Programmer/Items/Interfaces.md)
* **Shared**
  * [Summary](Programmer/Shared/Summary.md)

