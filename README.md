# PixelDot2D Framework Documentation

Official documentation for the PixelDot2D modular Unity framework.

> [!IMPORTANT]
> **Documentation Overhaul (Work in Progress)**
> 
> These docs are currently undergoing a major rewrite. While programming is still required to extend the framework with custom modules, PixelDot2D has evolved to empower designers to build complete 2D games directly in the Unity Editor—assembling characters, unique behaviors, weapons, and items out of modular, "Lego-like" pieces without writing code.


## Required Setup & Critical Rules

Before configuring any modules, ensure your project follows these two foundational contracts (also located in `REQUIRED_SETUP.txt` inside the package root):

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
  * [SaveAndLoading](Designer/Core/SaveAndLoading.md)
* **Combat**
  * [Summary](Designer/Combat/Summary.md)
  * [Weapon Modules](Designer/Combat/WeaponModules.md)
  * [Hitscans](Designer/Combat/Hitscans.md)
  * [Projectiles](Designer/Combat/Projectiles.md)
  * [Hitbox](Designer/Combat/Hitbox.md)
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

