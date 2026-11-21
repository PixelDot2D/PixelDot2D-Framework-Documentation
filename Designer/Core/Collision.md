# Collision

The **Core Collision** system provides customizable 2D shape casting and trigger detection without requiring standard Unity Collider2D components on the detector. A single `CollisionManager` can dynamically swap between any shape asset (such as circles, boxes, or capsules) at runtime—when `ChangeCollisionShape` is called, the system automatically reconfigures itself to match the new shape.

> **Editor Performance Note:** Scene view performance in the Unity Editor will always be slower than a standalone build. Internally, every active shape draws custom debug lines in the Editor, which adds `Debug.Draw` rendering overhead that is stripped out in builds.

## Creating a Collision Shape Asset

1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. In the creation window, navigate to **Core -> Collision** and select your desired shape type to create the ScriptableObject asset.

<img width="1087" height="763" alt="image" src="https://github.com/user-attachments/assets/ef6229b0-287c-41d5-b9ef-89b492a9440a" />


## ScriptableObject Parameters

Select your `Base_SO_CollisionShape` asset in the Project window to configure its detection settings in the Inspector:

<img width="1285" height="636" alt="image" src="https://github.com/user-attachments/assets/1217d9a4-132d-4371-85c6-63db23865d46" />

- **Collision Check Type:** Controls how multi-collider GameObjects are reported in the batched event payload:
  - **Per Game Object:** Reports each intersecting GameObject only once per check, ignoring any additional colliders attached to that same GameObject.
  - **Per Collider:** Reports every intersecting collider individually. If a single GameObject has 10 colliders overlapping the shape, that GameObject will appear 10 times in the batched event list.
- **Size:** The dimensions (width/height or radius) of the collision shape.
- **Angle:** The base rotation angle (in degrees) of the shape.
- **Take Virtual Transform Rotation Into Account:** When enabled, adds the owner's `IVirtualTransform` rotation on top of the shape's **Angle**.
- **Offset:** Positional offset applied from the owner's `IVirtualTransform` world position.
- **Use Origin As Pivot:** By default, the shape is centered on the `IVirtualTransform` position (plus offset). When enabled, the position acts as the starting edge/pivot point instead of the center—useful for checks extending outward from a point.
- **Distance:** How far the shape casts/travels along **Direction**. Note that distance is driven strictly by **Direction** and does not automatically invert when an object flips (for example, a direction of `(1, 0)` always casts right regardless of `0` or `180` degree facing angles).
- **Direction:** The normalized vector direction the shape travels when **Distance** is greater than `0`.
- **Valid Layers:** LayerMask defining which layers trigger `Enter`, `Stay`, and `Exit` events.
- **Obstacles:** LayerMask defining which layers block or obstruct the collision check.
- **Detect Triggers:** When enabled, trigger colliders (`Is Trigger` checked) are included in detection; when disabled, triggers are ignored.

## Wiring Collision in C#

To use collision in a custom MonoBehaviour, implement **`IVirtualTransform`**, store a serialized `Base_SO_CollisionShape` reference, and manage the **`CollisionManager`** lifecycle:

```csharp
using UnityEngine;
using PixelDot2D.Core;
using PixelDot2D.Core.Physics;
using PixelDot2D.Core.Interfaces;

public class CustomCollisionDetector : MonoBehaviour, IVirtualTransform
{
    #region Fields
    //SerializeFields
    [SerializeField] private Base_SO_CollisionShape _collision;

    //Private
    private CollisionManager _collisionManager;
    private bool _isQuitting;
    #endregion

    #region Unity Lifecycle
    //=========================================================================
    private void Awake()
    {
        _collisionManager = new();
        _collisionManager.Init(this);
        _collisionManager.ChangeCollisionShape(_collision);

        _collisionManager.m_OnTriggerEnterEvent -= OnEnter;
        _collisionManager.m_OnTriggerEnterEvent += OnEnter;

        _collisionManager.m_OnTriggerExitEvent -= OnExit;
        _collisionManager.m_OnTriggerExitEvent += OnExit;

        _collisionManager.m_OnTriggerStayEvent -= OnStay;
        _collisionManager.m_OnTriggerStayEvent += OnStay;
    }
    //=========================================================================
    private void OnDisable()
    {
        if (_isQuitting)
        {
            return;
        }

        _collisionManager?.Exit();
    }
    //=========================================================================
    private void OnDestroy()
    {
        _collisionManager?.Destroy();
    }
    //=========================================================================
    private void OnApplicationQuit()
    {
        _isQuitting = true;
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

> **Tip:** After pasting the event subscriptions in `Awake()`, click on `OnEnter`, `OnExit`, or `OnStay` and press **Alt + Enter** (or your IDE's quick-fix shortcut) to automatically generate the matching callback method signatures for your custom collision logic.
