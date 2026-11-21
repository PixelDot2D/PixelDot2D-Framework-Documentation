# Hitbox

The Hitbox module handles all spatial overlap attacks and hazards—including sword swings, explosions, environmental traps, and rotating spikes. If an attack or hazard requires an active area check, build it with a Hitbox.

Make sure your scene has the **Combat Managers** prefab (specifically the **HitBox Sprite Visual Object Pool**) and matching layer enums configured. See [Template Combat Entity and Prefabs](TemplateCombatEntityAndPrefabs.md) for scene setup and decoupled damage handling.

## Step 1: Bulk Create the Hitbox Structure

1. Create and select a target folder in the Project window (for example, `Hitboxes/SwordSwing`) so the Creation Hub knows where to output the assets.
2. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
3. Navigate to **Macros -> Combat -> Bulk Creation -> Build Hitbox Matrix**.

<img width="1141" height="746" alt="image" src="https://github.com/user-attachments/assets/0dbf200e-f588-49c2-8dca-2e1b2e96d635" />

## Step 2: Populate the Weapon Modules

The macro generates a folder structure with three empty folders for your weapon stages: `Aim`, `Gate`, and `VT` (Virtual Transform).

<img width="435" height="223" alt="image" src="https://github.com/user-attachments/assets/b7f59b0c-0479-4749-a4b8-fcd665204a54" />

1. Select each folder one at a time.
2. In the Creation Hub, open **Combat -> Weapons** and create the matching asset inside each folder from:
   - **Aiming Types** (into `Aim`)
   - **Execution Gate Types** (into `Gate`)
   - **Virtual Transform Types** (into `VT`)
3. Expand the **Weapons** folder and select your weapon asset. Drag the newly created `Aim`, `Gate`, and `VT` assets into their slots (the **Execution** slot is already pre-wired).
4. Select **Multi_Weapon** in the same folder. Your weapon is already hooked up here. The Weapon Manager uses `Multi_Weapon` to build attacks, allowing a single trigger to execute multiple mixed weapons (hitboxes, projectiles, or hitscans) together.

## Step 3: Configure Hitbox Execution

Expand the **Execution** folder and select the Hitbox ScriptableObject:

<img width="922" height="957" alt="image" src="https://github.com/user-attachments/assets/18145f28-37f6-4f83-828b-265839b696df" />

### Damage & Scaling
- **Flat Damage:** Base damage dealt per hit.
- **Stat Scaling:** Character stat used to scale damage.
- **Scaling Amount:** Multiplier applied to the scaled stat (`1` = 100%, `0.5` = 50%).
- **School Flags:** Elemental or damage type flags (such as `Fire`).

*(Note: Raw damage and scaling values are passed into `DamageData` without pre-calculating final damage. See [Template Combat Entity and Prefabs](TemplateCombatEntityAndPrefabs.md) for how `IWeaponizable` and `IDamageable` resolve hits.)*

### Spatial & Timing Settings
- **Visual:** Visual asset rendered for this hitbox (explained in Step 4).
- **Hit Box Rotation Update Type:**
  - **Snapshot:** Captures the Virtual Transform's rotation at the moment of execution and holds that angle for the rest of the duration.
  - **Continuous:** Updates the hitbox's rotation continuously to match the Virtual Transform as it moves or aims.
- **Hit Box Position Update Type:**
  - **Snapshot:** Locks the hitbox at the initial execution position.
  - **Continuous:** Moves the hitbox continuously along with the Virtual Transform's position.
- **Duration:** How long the hitbox remains active after firing, even if the Entity Sets the attack flag to false.
- **Line Of Sight:** Optional. If left `null`, the hitbox hits through walls. If assigned (create via **Creation Hub -> Core -> Line of Sight**), the system checks from the hitbox's center to the closest point on any valid target to verify unobstructed line of sight before dealing damage.
  - *Layer Note:* You can leave the valid targets and obstacle layer arrays blank on the Line of Sight asset—the framework automatically pulls and overrides them at runtime from the attacker's `IWeaponizable` interface without mutating your ScriptableObject.
- **Hit Box Shape:** Defines the overlap geometry using Core Collision shapes. Create a shape via **Creation Hub -> Core -> Collision** (all shapes are supported) and assign it here.
  - *Layer Note:* Like Line of Sight, you can leave layer masks blank on the Collision asset; runtime target and obstacle layers are supplied automatically by the `IWeaponizable` entity.
- **On Stay Delay:**
  - **Below `0` (Default `-1`):** Triggers a single attack only when a target first enters the hitbox (`OnTriggerEnter` behavior).
  - **Above `0`:** Acts as a persistent repeating damage tick ("hot trigger"). When no target is inside, the timer counts down to `0` and stays primed—so the instant a valid target enters, it takes damage immediately and resets the timer.

## Step 4: Configure Hitbox Visuals

Hitboxes are purely virtual spatial checks decoupled from rendering. To attach a visual, open the Creation Hub and navigate to **Combat -> Visuals -> Hit box**:

<img width="1131" height="758" alt="image" src="https://github.com/user-attachments/assets/15abb7ec-be5f-41eb-8cb9-bfa571a414b0" />

The framework provides three default visual behaviors:

- **Sprite Animation:** Plays an assigned animation clip at the hitbox's position and rotation.
- **Sprite Default:** Displays a static sprite that tracks the hitbox's position and rotation. Pairing **Sprite Default** with **Continuous** rotation update lets you create clean melee swing arcs using a single sprite without needing frame-by-frame animation.
- **Rotate Sprite:** Continuously spins the sprite while the hitbox is active.

## Module Compatibility & Emergent Behavior

While the Combat system is built from modular LEGO-like pieces, **not every module combination produces sensible behavior**—and some combinations create unique, emergent mechanics:

- **Conflicting Combinations:** Pairing **Continuous** rotation update with an **Alternating** Aiming module forces the hitbox to receive a brand-new angle on every `FixedUpdate` tick, causing it to jitter wildly between angles rather than sweeping smoothly. Similarly, certain **Snapshot** settings may ignore continuous updates expected by specific Virtual Transform types.
- **Emergent Gameplay (Demo Hazard Example):** You can inspect an emergent setup under **PixelDot2D -> Combat -> SO -> Weapons -> Hazard**. Try dragging this asset into **Element 1** of the Player's **Weapons** array in the Combat Demo scene:
  - Holding the fire button (**Left Click** by default) spawns a 4-spiked hazard that locks in place while spinning continuously.
  - Releasing the trigger despawns it, and clicking again relocates and re-spawns the spinning hazard at the player's new position—all achieved purely through module configuration with zero custom code.


