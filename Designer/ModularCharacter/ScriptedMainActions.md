
# Scripted State — Main Actions

**Main Actions** serve as the primary locomotion driver for a Scripted State. While the Main Action module itself stays lightweight, it acts as the bridge to the Core **RB2D Movement** system—allowing a single compact asset to drive virtually any movement pattern by pairing it with different movement sequences.

## Locating Main Actions

To create a Main Action asset:
1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. Navigate to **ModularCharacter -> States -> Scripted State -> Scripted Main Actions**.

<img width="1125" height="755" alt="image" src="https://github.com/user-attachments/assets/b6d298ac-0adf-4f21-b504-11cd405d9568" />

---

## Included Main Actions

### Direct Scripted State (Non-Input)
Exposes only the **Movement Sequence** reference, executing the configured movement directly without reading directional input.

<img width="647" height="323" alt="image" src="https://github.com/user-attachments/assets/a3cc2c94-9525-4189-ad3d-ac7bfe8f68af" />

### Scripted Input State
Wraps the Movement Sequence with directional input polling and velocity scaling controls:

<img width="832" height="611" alt="image" src="https://github.com/user-attachments/assets/bd6615cf-3140-4a58-b1e9-086bc6b49af8" />


- **Max Vector:** Clamps or masks input per axis. Setting an axis to `0` ignores input along that axis (e.g., `X = 1, Y = 0` locks movement to horizontal side-scroller input).
- **Move Speed Multiplier:** Multiplies the character's `Move_Speed` stat while in this state (`1` = normal speed, `< 1` = slower movement, `> 1` = faster movement—for example, setting this to `3.5` on a Dash lets dash distance scale dynamically with the character's speed stat).
- **Forced Input:** Forces directional input on an axis (`1` or `-1`) even when the player provides no input (default `X = 0, Y = 0`). Setting `X = 1` on a Dash guarantees the character always dashes forward regardless of whether a movement key is held.
- **Input Polling Type:** Controls when and how directional input is sampled:
  - `Continuous`: Polls both X and Y input continuously every frame.
  - `Continuous X` / `Continuous Y`: Polls only the specified axis continuously every frame.
  - `Snapshot`: Captures both X and Y input once when the state is entered and locks that direction until the state exits (useful for committed aerials or dashes).
  - `Snapshot X` / `Snapshot Y`: Captures and locks only the specified axis upon entering the state.
 


## Shared

> [!IMPORTANT]
> Shared components require the **Shared Character** setup, which bridges decoupled systems—such as combat, inventory, damageables, weapons, and currency—into a unified character without coupling their underlying assemblies. In order to use any of the items below, you **must** use the Shared system.

### Combat Scripted States (Direct & Input)

These Shared Main Actions mirror the standard **Direct Scripted State (Non-Input)** and **Scripted Input State** modules, but drive a **Combat Sequence** instead of a standalone Movement Sequence—allowing characters to execute weapon attacks and animations alongside movement during a state.

> [!WARNING]
> Assigning a Combat Scripted State to a standard Modular Character (without the **Shared Character** setup) will fail and throw an error at runtime. See the **Shared** documentation for full setup details.

#### Creating a Shared Input
1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. Navigate to **Macros -> ModularCharacter -> Build [Shared] Scripted** or **Build [Shared] Scripted Input State Matrix** to generate the sequence and folder structure.
3. Open your newly generated `ScriptedState` folder, navigate to **Main Action -> Movement_Sequence**, and click the **`+`** icon on the array to add a new entry.
4. To populate that entry's movement, open the Creation Hub again and navigate to **Core -> RB2D Movement**, selecting any movement module (excluding the Movement Blueprint itself). Place it inside the generated `Movements` folder to keep your project organized, then drag it into the **Movement Data** slot on the `Movement_Sequence` entry (alongside any valid weapon and animation set).

<img width="1286" height="776" alt="image" src="https://github.com/user-attachments/assets/90196297-dacf-488a-a81c-ea030f17afeb" />

<img width="482" height="306" alt="image" src="https://github.com/user-attachments/assets/2f5ec700-b39d-4d93-9ed9-f7273e6dd3fc" />

<img width="783" height="818" alt="image" src="https://github.com/user-attachments/assets/f38e729b-2e4f-4438-b4e8-cec3bdf8b367" />

#### Variants & Parameters

- **Combat Direct Scripted State (Non-Input):** Exposes only the **Combat Sequence** reference, executing the sequence directly without reading directional input.

<img width="812" height="327" alt="image" src="https://github.com/user-attachments/assets/b2270d86-9284-4898-8933-f711c03dcd45" />

- **Combat Scripted Input State:** Wraps the **Combat Sequence** with the same directional input polling and velocity scaling controls as the standard **Scripted Input State** (**Max Vector**, **Move Speed Multiplier**, **Forced Input**, and **Input Polling Type**).

<img width="812" height="626" alt="image" src="https://github.com/user-attachments/assets/707caaa8-3689-4667-9dcd-808bea016d40" />

- **Max Vector:** Clamps or masks input per axis. Setting an axis to `0` ignores input along that axis (e.g., `X = 1, Y = 0` locks movement to horizontal side-scroller input).
- **Move Speed Multiplier:** Multiplies the character's `Move_Speed` stat while in this state (`1` = normal speed, `< 1` = slower movement, `> 1` = faster movement—for example, setting this to `3.5` on a Dash lets dash distance scale dynamically with the character's speed stat).
- **Forced Input:** Forces directional input on an axis (`1` or `-1`) even when the player provides no input (default `X = 0, Y = 0`). Setting `X = 1` on a Dash guarantees the character always dashes forward regardless of whether a movement key is held.
- **Input Polling Type:** Controls when and how directional input is sampled:
  - `Continuous`: Polls both X and Y input continuously every frame.
  - `Continuous X` / `Continuous Y`: Polls only the specified axis continuously every frame.
  - `Snapshot`: Captures both X and Y input once when the state is entered and locks that direction until the state exits (useful for committed aerials or dashes).
  - `Snapshot X` / `Snapshot Y`: Captures and locks only the specified axis upon entering the state.
