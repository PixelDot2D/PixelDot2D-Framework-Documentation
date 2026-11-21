# Line of Sight

The **Core Line of Sight (LoS)** system takes an inverse approach to standard raycasting: instead of checking whether a cast hits a target, it checks whether an **obstacle** blocks the path between Point A (the origin) and Point B (the target).

Because wider shapes collide with corners and geometry too easily—making line of sight overly restrictive—PixelDot2D ships exclusively with a **Line** shape for LoS checks.

> **Important Behavior Note:** Because this system strictly checks for obstacles along the cast and stops on the very first hit (`1`-hit early exit) for maximum speed, extreme or misaligned offsets can produce false positives. For example, if an offset aims the cast straight up (`90°`) while the target is to your left, and there are no obstacles above you, `HasLineOfSight` will return `true`. Keep offsets aligned with your character's eye/weapon height and target center.

## Creating a Line of Sight Asset

1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. In the creation window, navigate to **Core -> Line of Sight -> Line** to create the ScriptableObject asset.

<img width="1045" height="765" alt="image" src="https://github.com/user-attachments/assets/9913abda-cd95-4151-a855-e7d3ea8585fc" />


## ScriptableObject Parameters

Select your `Base_SO_LineOfSight` asset in the Project window to configure its settings in the Inspector:

<img width="841" height="566" alt="image" src="https://github.com/user-attachments/assets/729c0e19-2801-4a08-a987-9b3111310e5a" />

- **Obstacles:** LayerMask defining what blocks line of sight. If the cast hits any collider on these layers, line of sight is blocked (`false`).
- **Distance:** The maximum cast distance. When pairing Line of Sight with another detection system (such as a Combat **Hitbox**), you can set this to a very large value so the Hitbox shape handles range filtering while LoS only verifies unobstructed sight.
- **Positional Offset for Origin:** Offset applied to the owner's `IVirtualTransform` world position (Point A).
- **Positional Offset for Target:** Offset applied to the target's world position when aiming the cast (`targetPosition + offset`, Point B).
- **Detect Triggers:** When enabled, trigger colliders (`Is Trigger` checked) on the **Obstacles** layer will block line of sight; when disabled, triggers are ignored.

## Wiring Line of Sight in C#

To run Line of Sight checks in a custom MonoBehaviour, implement **`IVirtualTransform`**, create a runtime instance from your `Base_SO_LineOfSight` asset in `Awake()`, and call `HasLineOfSight`:

```csharp
using UnityEngine;
using PixelDot2D.Core;

public class CustomLineOfSightChecker : MonoBehaviour, IVirtualTransform
{
    #region Fields
    //SerializeFields
    [SerializeField] private Base_SO_LineOfSight _loSType;

    //Private
    private Base_State_LineOfSight _loSCheck;
    #endregion

    #region Unity Lifecycle
    //=========================================================================
    private void Awake()
    {
        _loSCheck = _loSType.CreateRunTimeInstance(this);
    }
    #endregion

    #region Public Methods
    //=========================================================================
    public bool CanSeeTarget(GameObject target)
    {
        return _loSCheck.HasLineOfSight(target);
    }
    #endregion

    #region IVirtualTransform
    //=========================================================================
    public float IVirtualTransform_GetRotation()
    {
        return transform.GetLocalRotation2D();
    }
    //=========================================================================
    public Vector2 IVirtualTransform_GetWorldPosition()
    {
        return transform.position;
    }
    #endregion
}
```

> **Tip:** `HasLineOfSight` includes **11 overloads**, allowing you to pass a `GameObject`, `Transform`, or raw `Vector2` target position directly depending on your gameplay script.
