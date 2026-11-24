# Modular Character Overview

The Modular Character system is a character-agnostic architecture designed to drive any entity—whether a Player, AI, or NPC—without locking your project into a specific genre or movement style.

Every character is driven by a **Character Blueprint** populated with **State ScriptableObjects** and **Character Attribute Data**. These assets store configuration data and act as runtime factories—when the character initializes, each asset constructs its own dedicated runtime state instance so no logic executes on the ScriptableObject itself.

## Two Workflows: Coded States vs. Scripted States

Both Coded States and Scripted States are **State ScriptableObjects** that you drag and drop directly into a Character Blueprint. The difference lies in how each asset builds its runtime state.

| Feature | Coded States | Scripted States |
| :--- | :--- | :--- |
| **Blueprint Asset** | State ScriptableObject (dragged into a Character Blueprint). | State ScriptableObject (dragged into a Character Blueprint). |
| **Runtime Construction** | Instantiates a single, dedicated C# state class that owns the entire state's logic (e.g., `State_ModularCharacter_Dash_SideScroller`). | Instantiates a state assembled from an array of sub-behavior assets, each constructing its own modular runtime logic block. |
| **Authoring Method** | Programmers write a bespoke C# state class; designers configure its exposed parameters on the asset. | Designers compose states in the Editor by adding modular behavior assets like building blocks. |
| **Best Used For** | Mechanics requiring custom math, low-level overrides, or delegating control to external systems. | Rapid prototyping, designer-owned character kits, and emergent mechanics without writing C#. |

### 1. Coded States
A Coded State is a State ScriptableObject paired with a dedicated C# runtime state class that owns its execution logic, physics handling, and exit conditions. Programmers define the exact behavior in C#, and designers drag the resulting asset into a Character Blueprint and tune its Inspector data.

### 2. Scripted States
A Scripted State is a State ScriptableObject with no hardcoded genre logic of its own. Instead, its Inspector exposes arrays where designers drag and drop modular sub-behavior ScriptableObjects—each responsible for constructing a specific slice of runtime logic (such as applying velocity, modifying gravity, or evaluating exit conditions).

Because these runtime blocks are composable, designers can build complex, emergent mechanics that were never explicitly hardcoded into the framework:
- **Steerable Dashes:** Combine dash velocity with directional steering behaviors to control movement mid-dash.
- **Physics-Free Aerials:** Assemble glide or hover states by tweaking gravity modifiers or omitting gravity entirely.
- **Chained Jump Escalation:** Configure sequential jump states that replace one another on consecutive inputs (e.g., Normal Jump -> Medium Jump -> High Jump).

## Mixing Coded and Scripted States

Because both workflows share the exact same State ScriptableObject contract, Coded and Scripted states can seamlessly co-exist inside the same Character Blueprint. A character can be built entirely with Coded States, entirely with Scripted States, or with any hybrid mix of the two.

For example, an **Interact** state is best implemented as a Coded State because it relies on **Inversion of Control (IoC)**—handing execution authority off to the target world object (such as a Pushable Object, or Wall Climbing) rather than driving movement directly. Meanwhile, that exact same Character Blueprint can use Scripted States for its entire locomotion kit (running, jumping, dashing, and gliding).
## Included Demo Scenes

The framework includes demo setups showcasing both workflows side by side (located under `PixelDot2D/ModularCharacter/SO/Blueprints/ClassicStyle` and `PixelDot2D/ModularCharacter/SO/Blueprints/ScriptedInputBased`):
- **Classic Demo (Coded States):** Demonstrates a traditional setup where each State ScriptableObject instantiates a dedicated C# class owning its own logic.
- **Scripted Demo (Scripted States):** Recreates the entire Classic Demo movement kit purely through composed behavior blocks, alongside advanced mechanics like steerable dashes, gliding, hovering, and multi-stage escalating jumps.

> **Note:** Both `ClassicStyle` and `ScriptedInputBased` are configured around standard input-driven control—typically used for Players, though nothing stops you from feeding simulated input into them from an AI controller or custom driver if your game calls for it.

