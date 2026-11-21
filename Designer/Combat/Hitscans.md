# Hitscans

Hitscans are the most straightforward execution type in PixelDot2D, designed for instant linear attacks such as bullets, lasers, or railguns. Rather than using a zero-width line cast, PixelDot2D performs a **2D box cast** starting at the Virtual Transform's position and traveling along its final aimed angle.

Make sure your scene includes the **Combat Managers** prefab (specifically the **HitScan Visual** pooler) and matching layer enums. See [Template Combat Entity and Prefabs](TemplateCombatEntityAndPrefabs.md) for scene setup and decoupled damage handling.

## Step 1: Bulk Create the Hitscan Structure

1. Create and select a target folder in the Project window (for example, `Hitscans/LaserRifle`) so the Creation Hub knows where to output the assets.
2. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
3. Navigate to **Macros -> Combat -> Bulk Creation -> Build Hitscan Matrix**.

<img width="1140" height="758" alt="image" src="https://github.com/user-attachments/assets/c16bb88c-f3f3-4eed-aeb7-a3065cff09b0" />

## Step 2: Populate the Weapon Modules

The macro generates a folder structure with three empty folders for your weapon stages: `Aim`, `Gate`, and `VT` (Virtual Transform).

<img width="423" height="252" alt="image" src="https://github.com/user-attachments/assets/e0994333-963e-4c76-9138-c6ea8e6ac6e0" />

1. Select each folder one at a time.
2. In the Creation Hub, open **Combat -> Weapons** and create the matching asset inside each folder from:
   - **Aiming Types** (into `Aim`)
   - **Execution Gate Types** (into `Gate`)
   - **Virtual Transform Types** (into `VT`)
3. Expand the **Weapons** folder and select your weapon asset. Drag the newly created `Aim`, `Gate`, and `VT` assets into their slots (the **Execution** slot is already pre-wired).
4. Select **Multi_Weapon** in the same folder. Your weapon is already hooked up here. The Weapon Manager uses `Multi_Weapon` to build attacks, allowing a single trigger to execute multiple mixed weapons (hitscans, projectiles, or hitboxes) simultaneously.

## Step 3: Configure Hitscan Execution

Expand the **Execution** folder and select `Hitscan_Execution`:

<img width="922" height="707" alt="image" src="https://github.com/user-attachments/assets/819e8ffa-2672-42b0-806f-3adc9db9c93a" />

### Damage & Scaling
- **Flat Damage:** Base damage dealt per hit.
- **Stat Scaling:** Character stat used to scale damage.
- **Scaling Amount:** Multiplier applied to the scaled stat (`1` = 100%, `0.5` = 50%).
- **School Flags:** Elemental or damage type flags (such as `Fire`).

*(Note: Raw damage and scaling values are passed into `DamageData` without pre-calculating final damage. See [Template Combat Entity and Prefabs](TemplateCombatEntityAndPrefabs.md) for how `IWeaponizable` and `IDamageable` resolve hits.)*

### Box Cast & Multi-Shot Settings
- **Size:** Dimensions of the box cast (`X` = maximum reach/length, `Y` = beam thickness/width). In most cases, use a very small `Y` value for a thin ray-like cast, or increase `Y` for wide beams.
- **Max Hits:** Maximum number of targets or obstacles the cast can penetrate before stopping. For example, if **Max Hits** is `1` and two enemies or walls are lined up, the cast stops at the first object hit.
- **Number Of Shots:** Total hitscan casts fired per trigger.
- **Delay Between Multi Shots:** Time delay (in seconds) between consecutive shots in a burst.
- **Spread:** Angle increment applied after the first shot. For example, a spread of `5` offsets subsequent shots by `5`, `-5`, `10`, `-10`, and so on.
- **Visual:** Visual tracer rendered for the cast (explained in Step 4).

## Step 4: Configure Hitscan Visuals

Hitscans are purely virtual physics queries decoupled from rendering. To attach a visual trace:

1. Open the Creation Hub and navigate to **Combat -> Visuals -> Hitscan**.
2. Create the default **Line Renderer** visual asset provided by the framework.
3. Drag the created visual asset into the **Visual** slot on `Hitscan_Execution`.

When fired, the hitscan reports its start point, end point, and thickness (`Size.y`) so the decoupled `LineRenderer` pooler can draw the beam accurately.
