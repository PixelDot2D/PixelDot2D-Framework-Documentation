
# Scripted State — Main Actions

**Main Actions** serve as the primary locomotion driver for a Scripted State. While the Main Action module itself stays lightweight, it acts as the bridge to the Core **RB2D Movement** system—allowing a single compact asset to drive virtually any movement pattern by pairing it with different movement sequences.

## Locating Main Actions

To create a Main Action asset:
1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. Navigate to **ModularCharacter -> States -> Scripted State -> Scripted Main Actions**.

<img width="1125" height="755" alt="image" src="https://github.com/user-attachments/assets/b6d298ac-0adf-4f21-b504-11cd405d9568" />

---

## Included Main Actions

### Standard Main Actions (Direct)
- **Movement:** An inputless mover that directly executes the assigned `Movement_Sequence` using Core RB2D Movement data. Best suited for AI, automated movement, or committed actions that do not poll player input.
- **`[Shared]` Movement:** Cannot be used inside a standard Modular Character state. See the **Shared** documentation for setup details.

### Input-Based Main Actions (`Input Based` Sub-Leaf)
- **Input Based Movement:** Polls directional input (continuous or snapshot) and drives the assigned `Movement_Sequence` using Core RB2D Movement rules and character stat multipliers.
- **`[Shared]` Input Based Movement:** Cannot be used inside a standard Modular Character state. See the **Shared** documentation for setup details.

> **Note:** For detailed Inspector parameter breakdowns comparing **Movement** vs. **Input Based Movement** (`Max Vector`, `Move Speed Multiplier`, `Forced Input`, and `Input Polling Type`), see [Character Blueprint](CharacterBlueprint.md#7-scripted-state-vs-scripted-input-state).
