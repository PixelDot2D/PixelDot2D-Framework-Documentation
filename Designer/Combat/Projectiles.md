# Projectiles

Projectiles are the most feature-rich execution type in PixelDot2D. Because a projectile wraps Core `RB2DMovement` inside a Combat sequencer, projectiles can follow multi-step movement paths and recursively fire additional weapons (other projectiles, hitscans, or hitboxes) while always routing hit attribution back to the original executioner.

You can inspect working examples in the project under **PixelDot2D -> Combat -> Demos -> Combat Demo**.

## Architecture Overview

A projectile's behavior and total lifetime are driven by up to three sequential states:

1. **Sleep (Optional):** Waits for a set duration before advancing.
2. **Formation (Optional):** Arranges multi-shot projectiles into a spatial pattern before releasing them.
3. **Movement (Required):** Executes sequential `RB2DMovement` steps and optional recursive weapon attacks.

**Total Projectile Lifetime:** A projectile's lifetime equals the combined duration of all active states on its blueprint. For example, if **Sleep** is `1s`, **Formation** is `1s`, and **Movement** is `2s`, the projectile automatically disables and returns to the pool after `4s`.

## Step 1: Bulk Create the Projectile Structure

To reduce setup friction, use the Creation Hub macro to generate the folder hierarchy and pre-wire core ScriptableObjects:

1. Create and select a target folder in the Project window (for example, `Projectiles/Fireball`) so the Creation Hub knows where to output the assets.
2. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
3. Navigate to **Macros -> Combat -> Bulk Creation -> Build Projectile Matrix Execute Script**.

<img width="1145" height="770" alt="image" src="https://github.com/user-attachments/assets/07059087-13a0-425b-bf59-e220e1127d83" />

<img width="493" height="438" alt="image" src="https://github.com/user-attachments/assets/939b576c-9e5a-4cbe-9eaf-65988fbf1750" />

## Step 2: Populate the Weapon Modules

The macro generates three empty folders for your weapon stages: `Aim`, `Gate`, and `VT` (Virtual Transform).

1. Select each folder one at a time.
2. In the Creation Hub, open **Combat -> Weapons** and create the matching asset inside each folder from:
   - **Aiming Types** (into `Aim`)
   - **Execution Gate Types** (into `Gate`)
   - **Virtual Transform Types** (into `VT`)
3. Expand the generated **Weapons** folder and select your weapon asset. Drag the newly created `Aim`, `Gate`, and `VT` assets into their slots (the **Execution** slot is already pre-wired).
4. Select **Multi_Weapon** in the same folder. Your weapon is already hooked up here. The Weapon Manager uses `Multi_Weapon` to build attacks, allowing a single trigger to fire multiple mixed executions (projectiles, hitscans, and hitboxes) simultaneously.

## Step 3: Configure Projectile States

Expand the **Execution** folder to find `States`, `Sequencer`, and `Movements`. By default, `Projectile_Movement_State` is already created inside `States`.

To add optional **Sleep** or **Formation** states, open the Creation Hub and navigate to **Combat -> Projectiles -> States**.

<img width="1140" height="776" alt="image" src="https://github.com/user-attachments/assets/ba5d8b81-428d-4d5d-ba3a-bbfc427292d5" />

### Sleep State

<img width="862" height="390" alt="image" src="https://github.com/user-attachments/assets/66526d37-f4fb-421c-847a-3665a38c9bc6" />

- **Duration:** How long the projectile waits before advancing to the next state.
- **First Projectile Sleeps:** When firing multiple shots, determines whether the first shot waits or launches immediately.
- **Take Index Into Account:** When disabled, all shots sleep for the exact **Duration**. When enabled, each shot sleeps for `Duration * ShotIndex` (creating a staggered wave).

### Formation State

<img width="862" height="691" alt="image" src="https://github.com/user-attachments/assets/7722d284-1144-4280-a6a2-b19b7a565884" />

- **Duration:** Time allowed for projectiles to move into their formation positions.
- **Sort Points:** Determines point ordering based on the selected sort mode (only noticeable when **Take Index Into Account** is enabled).
- **Wait For All Batch Formation:** Holds all shots in formation until every projectile in the batch reaches its point before advancing to the next state.
- **Rotate Formation:** Rotates the entire formation layout to match the Virtual Transform's firing angle (for example, rotating 90 degrees when aiming straight up).
- **Formation Points & Formation Parent:** Instead of typing coordinates manually, drag a prefab with child Transforms into **Formation Parent** at the top of the Inspector. The asset immediately snapshots all child positions into **Formation Points** and clears the prefab reference so you can delete the temporary prefab.

### Movement State & Sequencer

Select `Projectile_Movement_State` inside **Execution -> States**. Here you can configure **Duration**, **Disable When Off Screen**, and **Movement Pattern** (already pre-wired).

Click **Movement Pattern** to ping and open `Projectile_Sequence` inside the `Sequencer` folder, then click **+** to add sequence steps:

<img width="853" height="1102" alt="image" src="https://github.com/user-attachments/assets/ff283afc-8581-4594-8676-9541f2a4422e" />

- **Index Name:** Designer-only label (such as `Stop`, `Dash Left`, or `Spiral Out`) to keep sequence steps readable in the Inspector.
- **Animation:** Animation clip to play during this step (requires the **Animation** Visual Behaviour).
- **Movement Data:** Click **+** to add one or more `RB2DMovement` assets that play in order. Create new movement assets via **Creation Hub -> Core -> RB2D Movement** (all movement types except Blueprint are supported).

<img width="1145" height="770" alt="image" src="https://github.com/user-attachments/assets/de37e39c-5897-4828-96a7-5d5fe5755d7f" />

- **Frames To Execute:** Specifies which animation frames trigger the recursive **Weapon**. If left empty, the recursive weapon fires every `FixedUpdate` (requires the **Animation** Visual Behaviour).
- **Weapon:** Accepts a `Multi_Weapon` asset. Each outer index in the sequence can pair its own set of movements and animations with a different `Multi_Weapon` (or leave it empty).

## Step 4: Configure Projectile Execution

Select the Projectile Execution asset to configure damage and multi-shot parameters:

<img width="860" height="650" alt="image" src="https://github.com/user-attachments/assets/b57359d0-c5fa-47ed-92be-5ae7c0622776" />

### Damage & Scaling
- **Flat Damage:** Base damage dealt per hit.
- **Stat Scaling:** Entity stat used to scale damage.
- **Scaling Amount:** Multiplier applied to the scaled stat (`1` = 100%, `0.5` = 50%).
- **School Flags:** Elemental or damage type flags (such as `Fire`).

*(Note: Raw damage and scaling values are passed into `DamageData` without pre-calculating final damage. See [Template Combat Entity and Prefabs](TemplateCombatEntityAndPrefabs.md) for how `IWeaponizable` and `IDamageable` resolve hits.)*

- **Projectile Blueprint:** Already pre-wired by the macro.
- **Number Of Shots:** Total projectiles fired per trigger.
  - *Note:* When using a **Formation State** on the blueprint, Unity's Inspector cannot recalculate the shot count immediately upon selection. Nudge the field (type or delete a digit) to trigger `OnValidate` and auto-populate the exact formation point count.
- **Delay Between Multi Shots:** Time delay between consecutive shots in a burst.
- **Spread:** Angle increment applied after the first shot. For example, a spread of `5` offsets subsequent shots by `5`, `-5`, `10`, `-10`, and so on.

## Step 5: Configure Projectile Blueprint

Select `Projectile_Blueprint` and expand **Projectile Object Data**:

<img width="867" height="512" alt="image" src="https://github.com/user-attachments/assets/b5a0ed97-4e61-4648-bf7c-66418d8350c9" />

- **Sprite:** The visual sprite rendered for the projectile.
- **Collision:** Create a collision shape via **Creation Hub -> Combat -> Projectiles -> Collisions** and drag it into this slot.

<img width="1140" height="757" alt="image" src="https://github.com/user-attachments/assets/491d3ad5-b185-4117-a631-9ed46e67e313" />

- **Visual Behaviour:** Create a visual behaviour asset via **Creation Hub -> Combat -> Projectiles -> Visual Behaviour** and assign it here:

<img width="1137" height="757" alt="image" src="https://github.com/user-attachments/assets/7f87c070-290d-4d30-8e1d-25d952d7b238" />

  - **Animation:** Plays the animation clips assigned inside `Projectile_Sequence`.
  - **Continuous Rotation:** Spins the sprite continuously.
  - **Look At Velocity:** Rotates the sprite to face its current movement vector.
  - **Velocity Sprite Flip:** Flips the sprite on X, Y, or both axes based on movement direction.

## Decoupled Damage & Standalone Usage

Combat is completely decoupled from the Modular Character system, allowing you to use **Core + Combat** on its own and plug in your own gameplay logic.

Values like **Flat Damage** and **Stat Scaling** are passed straight through inside the combat data struct without pre-calculating final damage for you. For a practical example of how a standalone entity receives and processes this struct, see [Template Combat Entity and Prefabs](TemplateCombatEntityAndPrefabs.md).

