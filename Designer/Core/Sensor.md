# Sensor

The **Core Sensor** system acts as a fast, binary **Yes / No** detector for environmental checks such as ground detection, ceiling checks, wall detection, or quick interaction queries.

Unlike **Collision** (which tracks Enter/Stay/Exit events across multiple objects) or **Line of Sight** (which checks for obstacles between two points), a **Sensor** stops as soon as it detects a single valid hit. This `1`-hit early exit makes sensors ideal for frequent, per-frame physics queries.

> **Editor Performance Note:** Scene view performance in the Unity Editor is slower than in a standalone build because active sensors draw debug shapes in the Editor. This debug drawing is automatically stripped from standalone builds.

## Creating a Sensor Asset

1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. In the creation window, navigate to **Core -> Sensor** and pick your desired shape (**Box**, **Capsule**, **Circle**, or **Line**).

<img width="1061" height="767" alt="image" src="https://github.com/user-attachments/assets/96f9f880-b1ad-4ae1-9d00-7a8ef326ed0a" />

## ScriptableObject Parameters

Select your `Base_SO_Sensor` asset in the Project window to configure its detection settings in the Inspector:

<img width="1061" height="710" alt="image" src="https://github.com/user-attachments/assets/dea6a221-4e7e-4e62-9db7-850fe1081a6b" />

- **Valid Layers:** LayerMask defining what triggers a `true` (`Yes`) result.
- **Size:** The dimensions (width/height or radius) of the sensor shape.
- **Angle:** The base rotation angle offset (in degrees) of the shape.
- **Take Origin Rotation Into Account:** When enabled, adds the owner's `IVirtualTransform` rotation on top of the shape's **Angle**.
- **Position Offset:** Positional offset applied from the owner's `IVirtualTransform` world position.
- **Use Origin As Pivot:** By default, the shape is centered on the `IVirtualTransform` position (plus offset). When enabled, the position acts as the starting edge/pivot point and the shape expands outward—ideal for ground checks beneath a character's feet or wall checks extending forward.
- **Distance:** How far the shape casts along **Direction**. Like Collision, distance is driven strictly by **Direction** and does not automatically invert when an object flips.
- **Direction:** The normalized vector direction the shape travels when **Distance** is greater than `0`.
- **Detect Triggers:** When enabled, trigger colliders (`Is Trigger` checked) on **Valid Layers** count as a valid hit; when disabled, triggers are ignored.

## Wiring a Sensor in C#

To use a Sensor in a custom MonoBehaviour, implement **`IVirtualTransform`**, create a runtime instance from your `Base_SO_Sensor` asset in `Awake()`, and query it in your gameplay methods:

```csharp
using UnityEngine;
using PixelDot2D.Core;
using PixelDot2D.Core.Physics;
using PixelDot2D.Core.Interfaces;
using PixelDot2D.Core.Extensions;

public class CustomSensorDetector : MonoBehaviour, IVirtualTransform
{
    #region Fields
    //SerializeFields
    [SerializeField] private Base_SO_Sensor _sensorShape;
    private int _doorLayerMask;

    //Private
    private Base_State_Sensor _sensor;
    #endregion

    #region Unity Lifecycle
    //=========================================================================
    private void Awake()
    {
        _doorLayerMask = 0.GetEditorLayerBitShifted(Enum_EditorLayers.Door);
        _sensor = _sensorShape.CreateRuntimeInstance(this);
    }
    #endregion

    #region Public Methods
    //=========================================================================
    public bool IsGrounded()
    {
        return _sensor.HasCollision();
    }
    //=========================================================================
    public void TryInteractWithDoor()
    {
        IDoor door = _sensor.TryGetType<IDoor>(_doorLayerMask);

        if (door != null)
        {
            door.Open();
        }
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

## Available Query Methods

Once initialized, `Base_State_Sensor` exposes several ways to inspect what it detected:

- **`HasCollision()`:** Performs a cast and returns `true` if at least one valid collider was hit, or `false` otherwise.
- **`TryGetCollider()` / `TryGetRayCastHit()`:** Performs a fresh cast and outputs the hit `Collider2D` or `RaycastHit2D` directly.
- **`TryGetColliderFromOldResult()` / `TryGetRayCastHitFromOldResult()`:** Reads the `Collider2D` or `RaycastHit2D` cached from your most recent `HasCollision()` call without running a second physics cast.
- **`TryGetType<T>(LayerMask)`:** Casts against the specified mask and attempts to retrieve a component or interface of type `T` (such as `IDoor` or `IDamageable`) from the hit object.
