# RB2D Movements

RB2D Movements are hyper-focused, modular movement behaviors that power everything across the framework. Rather than locking an entity into a single movement script, you can mix and match multiple RB2D Movement assets inside a **Movement Sequence Blueprint** to chain complex trajectories, patrols, and combat behaviors together.

> [!IMPORTANT]
> Before configuring movement speeds, read the [RB2D Mover](https://github.com/PixelDot2D/PixelDot2D-Framework-Documentation/blob/main/Designer/Core/RB2DMover.md) documentation. **Move Speed** is a `Vector2` (`X` and `Y`) whose final runtime value is scaled dynamically by the controlling system (for example, Modular Character states pass runtime speed multipliers into `FixedUpdate`—the ScriptableObject asset itself is **never mutated**), and individual movement types interpret `X` and `Y` differently.

To create an RB2D Movement asset, press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**, then navigate to **Core -> RB2D Movement**.

---

## Shared Parameters

Every RB2D Movement asset includes two base properties:

- **Duration:** Time in seconds that the Movement Manager runs this module before advancing to the next inner or outer sequence index (see [RB2D Mover](https://github.com/PixelDot2D/PixelDot2D-Framework-Documentation/blob/main/Designer/Core/RB2DMover.md) for sequence indexing details).
- **Move Speed:** `Vector2` (`X` and `Y`) speed values passed into the movement calculation. How `X` and `Y` are applied varies per movement type (detailed below).

<img width="778" height="110" alt="image" src="https://github.com/user-attachments/assets/627e27e3-3204-48f4-81b5-3a7323cc1d09" />

---

## Included Movements

### Follow Array of Vector2s Path

Moves the Rigidbody sequentially through a dynamic `Vector2[]` array supplied at runtime via a C# delegate (such as a patrol route or dynamically generated waypoints). See the [C# Wiring & Combat Interfaces](../Combat/TemplateCombatEntityAndPrefabs.md#c-wiring--combat-interfaces) section in [Template Combat Entity and Prefabs](../Combat/TemplateCombatEntityAndPrefabs.md) for an example of wiring this delegate.


<img width="786" height="502" alt="image" src="https://github.com/user-attachments/assets/8d99c67d-deb6-4107-b197-d2c48fedfa81" />

**Parameters**
- **Stopping Distance:** Distance from a waypoint at which the mover considers the point reached. Setting this too low can cause the Rigidbody to overshoot and orbit the point indefinitely (`0.25` to `0.5` is a reliable starting range; slower movement speeds allow smaller values).
- **Ignore Y:** When enabled, ignores the Y-axis and evaluates waypoint arrival using only the X position.
- **Ignore X:** When enabled, ignores the X-axis and evaluates waypoint arrival using only the Y position.
- **Look at Target:** When enabled, rotates the Rigidbody to face the current target waypoint.

---

### Follow Points

Follows a pre-authored waypoint path baked directly into the ScriptableObject at edit time, eliminating runtime scene object overhead.

<img width="785" height="587" alt="image" src="https://github.com/user-attachments/assets/40ee4e84-2ba0-46ae-9681-4e215f114a96" />

**Parameters**
- **Points Parent:** Create an empty GameObject in your scene, add child GameObjects to lay out your desired path (with no limit on point count), save it as a prefab, and drag the prefab into this slot. The asset automatically snapshots the child positions into the **Points** list and clears the prefab reference—you can safely delete the temporary scene object and prefab afterward, or keep them as a visual reference.
- **Loop Style:** Determines how the mover behaves after reaching the final waypoint (single selection):
  - **Continuous:** Continues updating the path and rotation seamlessly past the end of the sequence.
  - **Ping Pong:** Traverses forward (`0 -> 5`), then reverses back through the path (`4 -> 0`) and repeats.
  - **Loop:** Traverses the path (`0 -> 5`), then returns to index `0` and repeats.
- **Stopping Distance:** Distance from a waypoint at which the mover considers the point reached. Setting this too low can cause the Rigidbody to overshoot and orbit the point indefinitely (`0.25` to `0.5` is a reliable starting range; slower movement speeds allow smaller values).
- **Points:** Read-only snapshot list of `Vector2` positions populated by **Points Parent** (can also be typed in manually if needed).

---

### Follow Target

Tracks and moves toward a live target supplied at runtime via a C# delegate subscription (see the Programmer documentation for delegate subscription details).

<img width="790" height="508" alt="image" src="https://github.com/user-attachments/assets/a00fc53f-e792-46ee-b21c-e42554c52309" />

**Parameters**
- **Stopping Distance:** Distance from the target position at which the mover halts. Setting this too low can cause the Rigidbody to overshoot (`0.25` to `0.5` is a reliable starting range; slower movement speeds allow smaller values).
- **Offset:** Distance offset relative to the target. Tuning the relationship between **Offset** and **Stopping Distance** lets you build distinct AI archetypes from this single movement module without custom scripts:
  - **Kiting Archer / Mage:** Setting **Offset** to `5` and **Stopping Distance** to `0.1` causes the entity to pursue a trailing position `5` units away from the target, maintaining distance and repositioning automatically.
  - **Guarding Sentry:** Setting **Offset** to `0` and **Stopping Distance** to `5` makes the entity pursue the target until entering a `5`-unit perimeter, then hold its ground even if the target moves closer.
- **Ignore Y:** When enabled, tracks the target along the X-axis only.
- **Ignore X:** When enabled, tracks the target along the Y-axis only.

---

### Radial Spiral

Moves the Rigidbody in a continuous spiral around the Movement Manager's origin point by blending orbital tangent motion with radial inward/outward steering—avoiding heavy polar-to-Cartesian math at runtime.

<img width="792" height="398" alt="image" src="https://github.com/user-attachments/assets/b1eab559-89d7-4f00-9bbc-f338a3f93967" />

**How Move Speed Works**
- **Move Speed X:** Controls the orbital speed around the origin (positive values orbit counter-clockwise; negative values orbit clockwise).
- **Move Speed Y:** Acts as the radial steering rate (similar to steering a car inward or outward):
  - **Positive (`> 0`):** Spirals outward away from the origin.
  - **Zero (`0`):** Holds a steady circular orbit at the current radius.
  - **Negative (`< 0`):** Spirals inward toward the origin (stopping once within the internal radius tolerance).
- Overall travel speed scales with the combined magnitude of `Move Speed` (`X` and `Y`), and the GameObject automatically rotates to face its travel direction.

---

### Regular Movement

Moves the Rigidbody in a straight line along the configured `Move Speed` (`X` and `Y`) vector (e.g., horizontal, vertical, or diagonal movement).

<img width="793" height="372" alt="image" src="https://github.com/user-attachments/assets/8de76252-bbe0-49d5-80a4-161dbf1f5e3a" />

---

### Rotate Around Origin

Orbits the Rigidbody around the Movement Manager's origin point (configurable via C# code; see the Programmer documentation). Pairing this movement with projectiles in a line formation anchored to a fire point creates a classic rotating firewall hazard.

<img width="782" height="373" alt="image" src="https://github.com/user-attachments/assets/9dbd193a-690c-426a-bf1e-29751d38f8d4" />

This movement uses only the shared **Duration** and **Move Speed** parameters.

---

### Rotate to Target

Rotates the Rigidbody toward a target position supplied at runtime via the Movement Manager's target delegate, with support for either instant snapping or smooth turning while moving forward.

<img width="791" height="482" alt="image" src="https://github.com/user-attachments/assets/4bf54aee-5ac0-43f6-acf6-6bae96eae2a5" />

**Emergent Behavior (Homing, Lock-On & Charges)**
Because **Rotate to Target** automatically advances to the next movement in the sequence once aligned with the target, chaining it before a **Regular Movement** creates emergent combat behaviors without custom scripts:
- **Snap + Charge:** Set **Rotate Type** to **Snap** followed by a fast **Regular Movement** to make an enemy or projectile instantly aim at the target and lunge straight forward.
- **Loose Homing / Lock-On:** Set **Rotate Type** to **Smooth** with a positive `Move Speed X` and a low **Rotate Smooth Speed**, followed by a high-velocity **Regular Movement**. The entity or projectile will travel forward in a wide, arching homing curve as it tracks the target, then accelerate into a high-speed dash once locked within the **Smooth Angle Target**.

**Parameters**
- **Rotate Type:** Determines how the Rigidbody rotates toward the target (single selection):
  - **Snap:** Instantly rotates the Rigidbody to face the target upon entering this movement, then immediately advances to the next movement in the sequence.
  - **Smooth:** Gradually rotates toward the target over time while applying linear velocity along the transform's forward (`right`) and `up` axes. If no valid target is available, the Rigidbody continues spinning at the configured rotation speed. Once aligned within the **Smooth Angle Target**, the mover automatically advances to the next movement in the sequence.
- **Rotate Smooth Speed:** Angular turn speed used when **Rotate Type** is set to **Smooth**.
- **Smooth Angle Target:** Angle threshold (in degrees) used in **Smooth** mode to consider the Rigidbody facing the target and advance the sequence.

---

### Rotating

Moves the Rigidbody along a continuous looping arc or circle while rotating its heading. Depending on **Forward Speed**, this can create stationary tight loops or sweeping corkscrew/boomerang trajectories.

<img width="787" height="435" alt="image" src="https://github.com/user-attachments/assets/06a7a6d1-4441-43f6-ac47-efbfb80b88c4" />

**Parameters**
- **Rotation Speed:** How fast the Rigidbody rotates as it moves, curving its path into a full circle.
- **Forward Speed:** Upon entering this movement, the mover snapshots its initial orientation-safe forward (`right`) basis vector—calculated internally by the Movement Manager from the Rigidbody's rotation and active `worldSpaceRight` direction rather than `transform.right`, so horizontal flipping never requires inverting `transform.localScale.x`—and applies this constant drift velocity along that axis while rotating.

---

### Sin

Moves the Rigidbody along its orientation-safe relative forward (`Right`) basis vector while oscillating perpendicularly along its relative `Up` basis vector in a smooth sine wave—useful for wavy projectiles, floating hazards, or hovering enemy patterns. Because both axes are derived mathematically from the Rigidbody's physics rotation and the active `worldSpaceRight` / `worldSpaceUp` multipliers rather than `transform.right` or `transform.up`, horizontal sprite flipping and gravity-inverted layouts work automatically without inverting `transform.localScale`.

<img width="885" height="432" alt="image" src="https://github.com/user-attachments/assets/a82646d4-7588-4c9d-8c0c-13cb810886b2" />

**How Move Speed Works**
- **Move Speed X:** Forward travel speed along the relative `Right` basis vector.
- **Move Speed Y:** Wave amplitude (peak perpendicular velocity of the sine oscillation along the relative `Up` basis vector).

**Parameters**
- **Direction:** Multiplier (`1` or `-1`) that flips the initial phase direction of the wave (e.g., starting upward vs. downward).
- **Wave Frequency:** Controls how rapidly the sine wave oscillates over time (higher values create tighter, faster waves).

---

### Spiral Orbit

Orbits the Rigidbody around a purely virtual, math-driven phantom center point with no physical anchor GameObject required in the scene. When paired with Combat projectile formations (such as a square formation), every projectile orbits that shared virtual center simultaneously—keeping the entire formation rotating as a cohesive shape while traveling.

<img width="880" height="428" alt="image" src="https://github.com/user-attachments/assets/18c2e1d1-909d-40ad-a4f4-4f1aa964f720" />

**Parameters**
- **Rotation Speed:** Angular speed at which the Rigidbody orbits around the virtual phantom center.
- **Update Origin Point on Exit:** When enabled, updates the Movement Manager's internal origin point to the phantom center's final position when this movement finishes. This lets you chain multiple **Spiral Orbit** movements together (e.g., moving straight -> up -> down -> left) while keeping a projectile formation rotating cleanly around its moving center.


