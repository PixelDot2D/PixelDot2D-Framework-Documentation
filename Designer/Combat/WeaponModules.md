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
