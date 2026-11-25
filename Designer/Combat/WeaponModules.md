# Weapon Modules: Virtual Transform

A **Virtual Transform** acts as a simulated 2D point in the world (an X/Y position and a rotation angle) for your weapon to fire from.

When you assign a physical Unity `Transform` as a weapon's fire point, the system reads its world position and rotation as a read-only anchor. Instead of moving or rotating actual GameObjects in your scene hierarchy, the Virtual Transform handles offsets, orbits, and positional math virtually, then reports the final position and angle back to the weapon whenever an Execution fires.

To create a Virtual Transform module:
1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. Navigate to **Combat -> Weapons -> Virtual Transform Types**.

<img width="1015" height="770" alt="image" src="https://github.com/user-attachments/assets/3ceb3829-f9ac-44ec-b026-35ad403919ba" />


## Virtual Transform Types

Virtual Transforms define the origin position and base rotation angle for a weapon. Every Virtual Transform type includes a **Positional Offset** (`Vector2`) to shift the origin relative to its anchor point.

Many directional Virtual Transforms query `IWeaponizable_GetTargetCoordinates()` on the attacking entity:

```csharp
public (bool isValidCoordinates, Vector2 coordinates) IWeaponizable_GetTargetCoordinates()
{
    if (_target == null)
    {
        return (false, Vector2.zero);
    }

    return (true, _target.transform.position);
}
```

*(Tip: Because this method simply expects world coordinates, you can either pass a target Transform's position or calculate a virtual point from directional input—such as `transform.position + inputDirection`—to support twin-stick or 8-way directional aiming.)*

### 8 Directional

<img width="1056" height="452" alt="image" src="https://github.com/user-attachments/assets/294a28e0-1035-4840-b68d-75ea099be780" />

Snaps the origin and angle to discrete cardinal or diagonal directions toward `IWeaponizable_GetTargetCoordinates()`:

- **Positional Offset:** Base offset applied to the Virtual Transform's origin.
- **Aim Mode:**
  - **Classic 8 Way:** Snaps to all 8 cardinal and diagonal directions (`0°`, `45°`, `90°`, etc.).
  - **Cross 4 Way:** Snaps only to cardinal directions (Up, Down, Left, Right).
  - **X 4 Way:** Snaps only to diagonal directions (`45°`, `135°`, `225°`, `315°`).
- **Radius:** `Vector2` extent (`X` and `Y`) defining the bounding box around the source where the directional points sit. Using equal `X` and `Y` values creates a uniform ring, while unequal values create an oval or rectangular offset.

### Camera Position

<img width="1057" height="372" alt="image" src="https://github.com/user-attachments/assets/1f4c671b-f9b4-4160-9cb3-d5b5550bb065" />

Anchors the Virtual Transform directly to a screen/camera edge or corner—ideal for screen-edge hazards, off-screen projectile spawners, or UI-aligned attacks:

- **Positional Offset:** Offset applied relative to the selected camera anchor.
- **Anchor Position:** Choose from `Top_Left`, `Top`, `Top_Right`, `Left`, `Center`, `Right`, `Bottom_Left`, `Bottom`, or `Bottom_Right`.

### Look At Target

<img width="1061" height="412" alt="image" src="https://github.com/user-attachments/assets/63ab6190-4050-4138-a423-fbf57aeccf7c" />

Positions and aims the Virtual Transform toward the coordinates returned by `IWeaponizable_GetTargetCoordinates()`:

- **Positional Offset:** Base offset applied to the Virtual Transform's origin.
- **Radius Offset:** `Vector2` offset pushing the origin outward toward the target. Set only `X` for a standard circular radius, or add `Y` for a skewed, elliptical offset.
- **Angle:** Fallback angle used when `IWeaponizable_GetTargetCoordinates()` returns `isValidCoordinates = false` (no target).

### Mirrored Static

<img width="1060" height="376" alt="image" src="https://github.com/user-attachments/assets/5ecf8b84-4bca-4089-a0ba-67c30f9b10c5" />

Designed for side-scrollers, automatically flipping horizontal offsets and angles based on `IWeaponizable_GetFacingDirection()`:

- **Positional Offset:** Offset used when facing `Right`. When the entity faces `Left`, the horizontal offset automatically mirrors (for example, `(5, 0)` becomes `(-5, 0)`).
- **Angle:** Base rotation angle when facing `Right`, automatically mirrored when facing `Left`.

### Orbit Around Source

<img width="1060" height="498" alt="image" src="https://github.com/user-attachments/assets/73d96af2-d966-4389-ba03-24d229ffa6e2" />

Orbits the Virtual Transform continuously in a circle around the attacking entity:

- **Positional Offset:** Center offset relative to the source entity.
- **Radius Offset:** `Vector2` orbit distance. Set `X` for a standard circular orbit (leaving `Y` at `0`), or add `Y` to create a skewed elliptical orbit.
- **Aiming Angle:** Fixed base angle of the Virtual Transform itself. Because **Aiming Angle** stays fixed rather than rotating with the orbit, leave it at `0` and pair this Virtual Transform with a **Look At Source** Aiming module:
  - Set **Inverse = true** on **Look At Source** to aim outward away from the player (such as radial projectiles).
  - Set **Inverse = false** to aim inward toward the player (such as orbiting shields or inward-facing hazards).
- **Start Angle:** Initial angle along the orbit path.
- **Rotation Speed:** Orbit speed in degrees per second.
- **Rotate Only When Executing:** When enabled, resets the orbit back to `0` when idle and only advances rotation while the weapon is actively executing.

### Radius Anchored Orbit

<img width="1060" height="501" alt="image" src="https://github.com/user-attachments/assets/a99cda79-9e69-46f6-9067-6ae75ba594f0" />

Orbits around the source while tracking `IWeaponizable_GetTargetCoordinates()`, clamping the origin onto a perimeter around the entity:

- **Positional Offset:** Center offset relative to the source entity.
- **Radius:** `Vector2` orbit perimeter (`X` for standard circular radius; non-zero `Y` introduces skewed tracking).
- **Rotation Speed:** Speed at which the orbit point rotates toward the target's angle.
- **Default Angle:** Resting angle when no target is present.
- **Snap To Target When Not Executing:** Keeps the Virtual Transform primed directly on the target while idle so fast-moving targets cannot outrun the initial rotation when firing starts.
- **Reset Angle On No Target:** When enabled, returns to **Default Angle** if `IWeaponizable_GetTargetCoordinates()` reports no valid target.

### Single Static Point

<img width="1058" height="382" alt="image" src="https://github.com/user-attachments/assets/63d64e86-e86c-4c8a-8fe0-07603da8abc4" />

Places the Virtual Transform at a fixed offset and angle with no dynamic tracking or mirroring:

- **Positional Offset:** Fixed offset relative to the source entity.
- **Angle:** Fixed rotation angle in degrees.

---










































# Weapon Modules

Weapons in PixelDot2D are not hardcoded—they are assembled like LEGO bricks from interchangeable modules. Because every module can be mixed and matched freely, not every combination produces standard behavior:

- **Unintended Interactions:** Combining modules that fight each other (such as an **Alternating** Aiming module with **Continuous** rotation updates) can produce erratic angles.
- **Emergent Gameplay:** Combining modules creatively can also unlock unique mechanics with zero custom code—such as pairing an **Orbit Around Source** Virtual Transform with a **Look At Source** Aiming module to create orbiting, outward-facing spikes or radial projectile spinners.

Below is a reference for each built-in module. To browse or create any of these assets, press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**, then navigate to **Combat -> Weapons** (`Aiming Types`, `Execution Gate Types`, `Execution Types`, and `Virtual Transform Types`).

---

## Aiming Types

Aiming modules modify or offset the Virtual Transform's base rotation without overwriting its underlying reference angle.

### Alternating

<img width="920" height="612" alt="image" src="https://github.com/user-attachments/assets/c6100b58-d26f-4b52-83af-567d92675c7f" />

Cycles through an array of preset angle offsets on each execution:

- **Cycle Type:**
  - **Increment:** Steps through the angle array in sequential order (`0, 1, 2...`).
  - **Random:** Picks a random angle element from the array on each shot.
  - **Pingpong:** Steps forward through the array, then reverses back down (`0, 1, 2, 1, 0...`).

### Look At Source

<img width="920" height="385" alt="image" src="https://github.com/user-attachments/assets/b22447cb-e75c-4ec7-bf53-07307da8bb9a" />

Orients the attack relative to the weapon's source entity:

- **Angle Offset:** Fixed degree offset added after calculating the angle toward the source.
- **Inverse:** When enabled, aims directly away from the source instead of toward it.

### Look At Target

<img width="1051" height="427" alt="image" src="https://github.com/user-attachments/assets/d8dd54b1-e9ad-4a99-8944-c2d5cf5cb901" />

Tracks the assigned `Target` Vector2 from `IWeaponizable`:

- **Default Angle:** Fallback angle used when no target is assigned.
- **Aim Speed:** How quickly (in degrees per second) the aiming module rotates toward the target.
- **Reset Angle On No Target:** When enabled, resets back to **Default Angle** if the target is lost or `null`.
- **Snap To Target When Not Executing:** Immediately snaps onto the target while the weapon is idle, then switches to **Aim Speed** tracking the moment firing begins. Pairing this with a slow **Aim Speed** creates classic telegraphing behavior—if the player stands still, the shot lands, but moving allows them to dodge.

### Rotating

<img width="1052" height="360" alt="image" src="https://github.com/user-attachments/assets/a25c0ea1-a4a2-43ec-9d31-3b4735c4a5f7" />

Spins the aiming angle continuously over time:

- **Reset Rotation On Execution:** Resets the accumulated spin back to `0` (matching the base Virtual Transform angle) when the weapon fires.
- **Speed:** Rotation speed in degrees per second.

### Static

<img width="1057" height="336" alt="image" src="https://github.com/user-attachments/assets/427ee4ea-0f65-42df-bc43-d9b9d6c87e93" />

Applies a fixed angle regardless of movement or targets:

- **Angle Offset:** Constant angle offset in degrees.

### Virtual Transform Rotation

<img width="1053" height="342" alt="image" src="https://github.com/user-attachments/assets/66a2077f-724d-498a-9c58-34f3df4c99ab" />

Directly mirrors the Virtual Transform's current rotation:

- **Angle Offset:** Additional degree offset added on top of the Virtual Transform's rotation.

---

## Execution Gate Types

Execution Gates act as the firing condition and resource controller, determining when a weapon is allowed to trigger.

### Ammo

<img width="1057" height="641" alt="image" src="https://github.com/user-attachments/assets/a79ef406-d1bd-4ee9-88dc-0d70cf309ce9" />

Manages ammunition counts, reload behaviors, and charge regeneration:

- **Reload Mode:**
  - **Exhaust:** Automatically reloads only after all ammo is depleted.
  - **Input:** Reloads only when the player presses the reload input (requires the Keybind Manager in the scene).
  - **Hybrid:** Reloads automatically when ammo hits `0`, or manually at any time via input (requires the Keybind Manager in the scene).
- **Reload Keybind:** Input binding used when **Reload Mode** is set to **Input** or **Hybrid**.
- **Ammo Mode:**
  - **Magazine:** Standard clip behavior—ammo stays depleted until a reload completes.
  - **Regenerating:** Skill-charge behavior—automatically regenerates `1` ammo every **Ammo Recovery Cooldown** seconds without needing a full reload.
- **Ammo Count:** Maximum ammunition or charge capacity.
- **Execution Cooldown:** Minimum delay between individual shots while ammo is available.
- **Ammo Recovery Cooldown:** In **Magazine** mode, acts as the full reload duration (in seconds). In **Regenerating** mode, acts as the time required to recover `1` charge.
- **Ammo Consumption Rate:** How much ammo is consumed per single execution.
- **Allow Cooldown While Executing:** When enabled, cooldown/recovery timers tick immediately during multi-shot bursts rather than waiting for the burst to finish.

### Gate Charge

<img width="1057" height="371" alt="image" src="https://github.com/user-attachments/assets/9a7002fe-73a8-4daa-80b5-cdc8e3583f86" />

Holds execution until the player charges the weapon to full capacity:

- **Charge Time:** Time in seconds required to reach full charge while the execute flag is held active.
- **Charge Decay:** Rate at which charge drains when the trigger is released before reaching full charge (`0` = no decay).

### Cooldown

<img width="1058" height="368" alt="image" src="https://github.com/user-attachments/assets/6bedb137-3db9-4359-96c9-4d905f2b5cbb" />

Enforces a standard timer delay between executions:

- **Cooldown:** Minimum delay in seconds between attacks.
- **Allow Cooldown During Execution:** When enabled, the cooldown timer starts ticking immediately when execution begins rather than waiting for multi-shot bursts or hitbox durations to finish.

---

## Virtual Transform Types

Virtual Transforms define the origin position and base rotation angle for a weapon. Every Virtual Transform type includes a **Positional Offset** (`Vector2`) to shift the origin relative to its anchor point.

Many directional Virtual Transforms query `IWeaponizable_GetTargetCoordinates()` on the attacking entity:

```csharp
public (bool isValidCoordinates, Vector2 coordinates) IWeaponizable_GetTargetCoordinates()
{
    if (_target == null)
    {
        return (false, Vector2.zero);
    }

    return (true, _target.transform.position);
}
```

*(Tip: Because this method simply expects world coordinates, you can either pass a target Transform's position or calculate a virtual point from directional input—such as `transform.position + inputDirection`—to support twin-stick or 8-way directional aiming.)*

### 8 Directional

<img width="1056" height="452" alt="image" src="https://github.com/user-attachments/assets/294a28e0-1035-4840-b68d-75ea099be780" />

Snaps the origin and angle to discrete cardinal or diagonal directions toward `IWeaponizable_GetTargetCoordinates()`:

- **Positional Offset:** Base offset applied to the Virtual Transform's origin.
- **Aim Mode:**
  - **Classic 8 Way:** Snaps to all 8 cardinal and diagonal directions (`0°`, `45°`, `90°`, etc.).
  - **Cross 4 Way:** Snaps only to cardinal directions (Up, Down, Left, Right).
  - **X 4 Way:** Snaps only to diagonal directions (`45°`, `135°`, `225°`, `315°`).
- **Radius:** `Vector2` extent (`X` and `Y`) defining the bounding box around the source where the directional points sit. Using equal `X` and `Y` values creates a uniform ring, while unequal values create an oval or rectangular offset.

### Camera Position

<img width="1057" height="372" alt="image" src="https://github.com/user-attachments/assets/1f4c671b-f9b4-4160-9cb3-d5b5550bb065" />

Anchors the Virtual Transform directly to a screen/camera edge or corner—ideal for screen-edge hazards, off-screen projectile spawners, or UI-aligned attacks:

- **Positional Offset:** Offset applied relative to the selected camera anchor.
- **Anchor Position:** Choose from `Top_Left`, `Top`, `Top_Right`, `Left`, `Center`, `Right`, `Bottom_Left`, `Bottom`, or `Bottom_Right`.

### Look At Target

<img width="1061" height="412" alt="image" src="https://github.com/user-attachments/assets/63ab6190-4050-4138-a423-fbf57aeccf7c" />

Positions and aims the Virtual Transform toward the coordinates returned by `IWeaponizable_GetTargetCoordinates()`:

- **Positional Offset:** Base offset applied to the Virtual Transform's origin.
- **Radius Offset:** `Vector2` offset pushing the origin outward toward the target. Set only `X` for a standard circular radius, or add `Y` for a skewed, elliptical offset.
- **Angle:** Fallback angle used when `IWeaponizable_GetTargetCoordinates()` returns `isValidCoordinates = false` (no target).

### Mirrored Static

<img width="1060" height="376" alt="image" src="https://github.com/user-attachments/assets/5ecf8b84-4bca-4089-a0ba-67c30f9b10c5" />

Designed for side-scrollers, automatically flipping horizontal offsets and angles based on `IWeaponizable_GetFacingDirection()`:

- **Positional Offset:** Offset used when facing `Right`. When the entity faces `Left`, the horizontal offset automatically mirrors (for example, `(5, 0)` becomes `(-5, 0)`).
- **Angle:** Base rotation angle when facing `Right`, automatically mirrored when facing `Left`.

### Orbit Around Source

<img width="1060" height="498" alt="image" src="https://github.com/user-attachments/assets/73d96af2-d966-4389-ba03-24d229ffa6e2" />

Orbits the Virtual Transform continuously in a circle around the attacking entity:

- **Positional Offset:** Center offset relative to the source entity.
- **Radius Offset:** `Vector2` orbit distance. Set `X` for a standard circular orbit (leaving `Y` at `0`), or add `Y` to create a skewed elliptical orbit.
- **Aiming Angle:** Fixed base angle of the Virtual Transform itself. Because **Aiming Angle** stays fixed rather than rotating with the orbit, leave it at `0` and pair this Virtual Transform with a **Look At Source** Aiming module:
  - Set **Inverse = true** on **Look At Source** to aim outward away from the player (such as radial projectiles).
  - Set **Inverse = false** to aim inward toward the player (such as orbiting shields or inward-facing hazards).
- **Start Angle:** Initial angle along the orbit path.
- **Rotation Speed:** Orbit speed in degrees per second.
- **Rotate Only When Executing:** When enabled, resets the orbit back to `0` when idle and only advances rotation while the weapon is actively executing.

### Radius Anchored Orbit

<img width="1060" height="501" alt="image" src="https://github.com/user-attachments/assets/a99cda79-9e69-46f6-9067-6ae75ba594f0" />

Orbits around the source while tracking `IWeaponizable_GetTargetCoordinates()`, clamping the origin onto a perimeter around the entity:

- **Positional Offset:** Center offset relative to the source entity.
- **Radius:** `Vector2` orbit perimeter (`X` for standard circular radius; non-zero `Y` introduces skewed tracking).
- **Rotation Speed:** Speed at which the orbit point rotates toward the target's angle.
- **Default Angle:** Resting angle when no target is present.
- **Snap To Target When Not Executing:** Keeps the Virtual Transform primed directly on the target while idle so fast-moving targets cannot outrun the initial rotation when firing starts.
- **Reset Angle On No Target:** When enabled, returns to **Default Angle** if `IWeaponizable_GetTargetCoordinates()` reports no valid target.

### Single Static Point

<img width="1058" height="382" alt="image" src="https://github.com/user-attachments/assets/63d64e86-e86c-4c8a-8fe0-07603da8abc4" />

Places the Virtual Transform at a fixed offset and angle with no dynamic tracking or mirroring:

- **Positional Offset:** Fixed offset relative to the source entity.
- **Angle:** Fixed rotation angle in degrees.

---

## Execution Types

Execution modules define what the weapon actually fires or evaluates once its Gate opens.

**Important Workflow Note:** While you can create standalone Execution assets under **Combat -> Weapons -> Execution Types**, it is **not recommended**. Each execution type expects a specific folder hierarchy and pre-wired helper assets (especially Projectiles). To stay organized, always generate executions using the bulk macros under **Creation Hub -> Macros -> Combat -> Bulk Creation**:

- **Hitbox (`Build Hitbox Matrix`):** Versatile spatial overlap checks for sword swings, hazards, bombs, and ticking area effects with flexible collision shapes. See [Hitbox](Hitbox.md) for full setup instructions.
- **Projectile (`Build Projectile Matrix`):** The most complex and flexible execution type, wrapping Core `RB2DMovement` and recursive weapon sequencers. See [Projectiles](Projectiles.md) for full setup instructions.
- **Hitscan (`Build Hitscan Matrix`):** The most straightforward execution type, performing instant directional 2D box casts for bullets, lasers, and beams. See [Hitscans](Hitscans.md) for full setup instructions.


