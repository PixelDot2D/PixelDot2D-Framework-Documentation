# Template Combat Entity and Prefabs

Because the Combat system is decoupled from visuals and character controllers, scenes require a few lightweight manager prefabs (primarily object poolers) to handle runtime execution.

You can find all required prefabs under **PixelDot2D -> Combat -> Prefabs**.

## Scene Manager Prefabs

For ease of setup, drop the **Combat Managers** prefab into your scene. It includes the following decoupled poolers:

- **HitBox Sprite Visual Object Pool:** Because hitboxes are purely virtual spatial checks decoupled from visuals, this pool spawns and recycles visual indicators independently.
- **HitScan Visual:** Reports start point, end point, and width/thickness so a decoupled visual (such as the provided demo `LineRenderer`) can draw the trace.
- **Projectile Object Pool:** Pre-allocates projectile shells and reuses them at runtime by swapping out blueprint and execution data rather than instantiating new GameObjects.

## Combat Entity Template

Located at **PixelDot2D -> Combat -> Prefabs -> Combat Entity Template**, this prefab serves as a general-purpose combat actor. Because its behavior is driven entirely by swappable data assets, it can function as an enemy, turret, boss, or moving hazard.

<img width="848" height="841" alt="image" src="https://github.com/user-attachments/assets/d0528d50-abb4-409e-9261-f3b8c540e384" />

### Inspector Fields

- **Movement Patterns:** The sequence driving how this entity moves and when it executes weapons (if applicable).
- **Points Parent:** Used with the `RB2DMovement` **Follow Array of Vector2s Path** movement type. Each parent holds child GameObjects whose positions are snapshotted into `Vector2` coordinates during `Awake` for patrol routes or shmup attack paths.
- **Fire Point:** The `Transform` origin used for firing weapons. Defaults to the entity itself, or can point to a custom child Transform.
- **Target:** The target `Transform` referenced by target-seeking Virtual Transforms, Aiming modules, or movement types. If left `null`, target-dependent modules gracefully fall back to static behavior (such as firing straight right or moving along the spawn angle).
- **RB & SR:** References to the entity's `Rigidbody2D` and `SpriteRenderer`.
- **Obstacles & Valid Layer Enum Array:** Defines which physics layers count as walls/obstacles or valid hit targets.
  - *Important Setup:* Open **PixelDot2D -> Core -> Scripts -> Enums -> Enum_EditorLayers.cs** and ensure the enum integer values match your Unity Project Settings layers (for example, if `Player` is Layer `8` in Unity, `Enum_EditorLayers.Player` must equal `8`).
