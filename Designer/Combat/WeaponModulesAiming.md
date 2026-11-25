# Weapon Modules: Aiming

An **Aiming** module controls how a weapon rotates its aiming angle (`right` direction) independently of the Virtual Transform's position. Separating aiming from position ensures your weapon never loses track of its current facing angle while moving or offsetting its fire point.

> **Combining Virtual Transforms & Aiming:** Some Aiming modules overlap with Virtual Transform behaviors (for example, both have *Look At Target* options). Stacking overlapping behaviors can cause unintended rotation conflicts, **but combining distinct ones unlocks complex patterns.** For example, pairing an **Orbit Around Source** Virtual Transform with an inverted **Look At Source** Aiming module creates a radial outward-facing ring around the character.

To create an Aiming module:
1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. Navigate to **Combat -> Weapons -> Aiming Types**.

<img width="1022" height="761" alt="image" src="https://github.com/user-attachments/assets/ceff790b-653c-400a-a430-79399ccfe5ad" />

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
