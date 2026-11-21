# RB2D Movement

The **RB2D Movement** system is a universal physics-driven mover capable of driving any GameObject that has a `Rigidbody2D`. Instead of writing separate movement scripts for players, enemies, moving platforms, or projectiles, one `RB2DMovementManager` can execute an unlimited variety of movement behaviors—straight lines, sine waves, orbital/radial curves, and more—chained together into multi-step sequences.

Every sequence step can also drive a sprite animation automatically (for example: move right while playing a walk animation, pause at `(0, 0)` while playing an idle animation, then repeat). Across the entire framework—from combat attack lunges to scripted character states—movement is powered by this single system, meaning any new movement asset you create can immediately be used anywhere.

## Creating a Movement Sequence Blueprint

1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. In the creation window, navigate to **Core -> RB2D Movement**.
3. Click **Movement Sequence Blueprint** to create the main sequence asset, and create any individual movement assets you want to combine (there is no limit to how many you can create or pair together).

<img width="1067" height="772" alt="image" src="https://github.com/user-attachments/assets/8f5744c8-2b57-47b5-bf6e-270a8b76aeff" />

## Blueprint & Sequence Parameters

Select your **Movement Sequence Blueprint** (`Base_SO_MovementSequence`) in the Project window and click **`+`** to expand the outer sequence array:

<img width="942" height="900" alt="image" src="https://github.com/user-attachments/assets/789f904c-c194-487b-8462-abafc3a2a6b6" />

The blueprint uses a two-level array structure so you can pair one animation with multiple sequential movements, or switch animations across distinct phases:

- **Outer Sequence Array (Animation Phases):** Each entry in the outer array defines a phase with its own **Index Name** (a designer-only label like `Patrol` or `Attack`) and an optional **Animation**. Clicking **`+`** on the outer array adds a new phase where you can switch to a different animation and movement set.
- **Inner Movement Data Array (Sequential Movements):** Inside each outer entry is an array of movements that play one after another while keeping that phase's **Animation** active. For example, you can assign a single `Walk` animation to an outer entry and add 10 inner movements (right, left, right, left) to create a full patrol route without re-assigning the animation each time.

### Example Movement Parameters (Regular Movement)

While each movement type has its own specialized settings, **Regular Movement** (basic directional movement along a vector) illustrates the two core parameters shared across all movement assets:

<img width="952" height="447" alt="image" src="https://github.com/user-attachments/assets/467fa5b0-582f-4fa8-b477-30a3a35abd57" />

- **Duration:** How long (in seconds) this movement plays before advancing to the next movement in the inner array (or moving on to the next outer sequence phase once the inner array finishes).
  - *Single-Movement Infinite Behavior:* If **both** the outer sequence array and the inner movement array have a size of **1**, **Duration** is automatically treated as infinite so that single movement loops continuously.
  - *Chained Behavior:* If either array has **2 or more** entries, each movement runs for its configured **Duration** before handing off to the next movement or outer phase.
- **Move Speed:** The base `(X, Y)` speed vector for this movement, which is multiplied by the speed vector passed into `_movement.FixedUpdate(...)` at runtime. You can drive speed from either side:
  - **Code-Driven Speed (Stats / Debuffs):** Leave **Move Speed** at `(1, 1)` on the asset and pass your entity's runtime speed stat into `FixedUpdate` (for example, `new Vector2(moveStat, moveStat)`). This prevents speed values from being locked into the ScriptableObject so buffs and debuffs work automatically.
  - **Asset-Driven Speed (Fixed / Designer-Tuned):** Set a fixed speed directly on the asset (for example, `(5, 0)`) and pass `Vector2.one` into `FixedUpdate` so the ScriptableObject acts as the primary speed driver.

## Wiring RB2D Movement in C#

To drive a `Rigidbody2D` with a movement sequence (and optional `AnimationPlayer2D`), initialize **`RB2DMovementManager`** in `Awake()` and tick it inside `FixedUpdate()`:

```csharp
using PixelDot2D.Core.Animation;
using PixelDot2D.Core.Movement;
using UnityEngine;

public class CustomRB2DMover : MonoBehaviour
{
    #region Fields
    //SerializeFields
    [SerializeField] private Base_SO_MovementSequence _sequence = null;
    [SerializeField] private Rigidbody2D _rb = null;
    [SerializeField] private SpriteRenderer _sr = null;

    //Private
    private RB2DMovementManager _movement;
    private AnimationPlayer2D _animationPlayer;
    #endregion

    #region MonoBehaviour
    //=========================================================================
    private void Awake()
    {
        _animationPlayer = new();
        _animationPlayer.Init(_sr);

        _movement = new();
        _movement.Init(_rb, RigidbodyInterpolation2D.Interpolate, RigidbodyType2D.Dynamic, _animationPlayer);
        _movement.Enter(_sequence);
    }
    //=========================================================================
    private void FixedUpdate()
    {
        Vector2 worldSpaceRight = new(1.0f, 0.0f);
        Vector2 worldSpaceUp = new(0.0f, 1.0f);
        Vector2 speedMultiplier = Vector2.one;
        float timeScale = 1.0f;

        _movement.FixedUpdate(Time.deltaTime, worldSpaceUp, worldSpaceRight, speedMultiplier, timeScale);
        _animationPlayer.Update(Time.deltaTime, timeScale);
    }
    //=========================================================================
    private void OnDestroy()
    {
        _animationPlayer?.Destroy();
        _movement?.Destroy();
    }
    #endregion
}
```

### Key Runtime Controls

- **Directional Flipping Without Scaling:** Instead of inverting `transform.localScale.x` to turn an object left or right, pass `new Vector2(-1.0f, 0.0f)` for left or `new Vector2(1.0f, 0.0f)` for right into `worldSpaceRight` (and flip your `SpriteRenderer.flipX` separately). All movement math automatically mirrors relative to the world-space axes you pass in.
- **Dynamic Speed & Time Scale:** Pass your entity's current speed stat into `speedMultiplier` (which multiplies against the movement asset's **Move Speed**) and pass custom local time scaling into `timeScale` to slow down or speed up individual entities independently of `Time.timeScale`.
- **Automatic Animation Syncing:** When `_animationPlayer` is passed into `_movement.Init(...)`, the movement manager automatically switches animations whenever a sequence step changes:
  - Providing a 1-frame `Animation` asset (such as a static idle sprite) is fully valid and updates the `SpriteRenderer` to that frame.
  - Leaving a sequence step's **Animation** field empty leaves the last active animation untouched on the `SpriteRenderer`.
