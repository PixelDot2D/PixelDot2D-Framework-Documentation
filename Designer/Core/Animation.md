# Animation

The **Core Animation** system is a lightweight, sprite-based alternative to Unity's built-in Animator controller. Because PixelDot2D is built specifically for 2D sprite workflows, subsystems like **Modular Character** rely on this asset-driven system to play animations without state-machine overhead.

Animation assets created here are universal and can be plugged into any component or subsystem across the framework that requires sprite animation.

## Creating an Animation Asset

1. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
2. In the creation window, navigate to **Core -> Animation** and click **Create Asset**.

<img width="1120" height="726" alt="image" src="https://github.com/user-attachments/assets/3f579f68-7507-40a1-ae6f-baf7380f87da" />

## ScriptableObject Parameters

Select your newly created Animation asset in the Project window to configure its playback settings in the Inspector:

<img width="752" height="353" alt="image" src="https://github.com/user-attachments/assets/46ed9470-c0ff-44cc-b04a-5a002af9f5e2" />

- **Loop Style:** Determines how the animation plays through its sprite array:
  - **Loop:** Repeats continuously from start to finish like a standard animation.
  - **Single:** Plays through all sprites in the array once and stops on the final frame.
  - **Ping Pong:** Plays forward from the first sprite to the last (`0 -> Last`), then reverses back to the start (`Last -> 0`).
- **Duration Per Sprite:** The base time (in seconds) each sprite frame stays visible before advancing to the next sprite.
- **Min Duration Variation:** The minimum random time offset (in seconds) applied to **Duration Per Sprite**. Use a negative value to allow frames to play slightly faster than the base duration.
- **Max Duration Variation:** The maximum random time offset (in seconds) applied to **Duration Per Sprite**.

> **Tip — Preventing Synchronized Animations:** On each frame, a random value between **Min Duration Variation** and **Max Duration Variation** is added to **Duration Per Sprite**. For example, if **Duration Per Sprite** is `1.0`, **Min** is `-0.1`, and **Max** is `0.1`, each sprite displays for a random duration between `0.9` and `1.1` seconds. This prevents multiple instances sharing the same asset (such as torches, foliage, or crowds of enemies) from animating in exact lockstep.

## Wiring Animation in C#

To drive animations in a custom MonoBehaviour, create an **`AnimationPlayer2D`** instance, initialize it with a `SpriteRenderer`, and pass your `Animation` array to `Play`:

```csharp
using UnityEngine;
using PixelDot2D.Core.Animation;

public class CustomSpriteAnimator : MonoBehaviour
{
    #region Fields
    //SerializeFields
    [SerializeField] private SpriteRenderer _spriteRenderer;
    [SerializeField] private Animation[] _animations;

    //Private
    private AnimationPlayer2D _animator;
    #endregion

    #region MonoBehaviour
    //=========================================================================
    private void Awake()
    {
        _animator = new();
        _animator.Init(_spriteRenderer);
        _animator.Play(_animations);
    }
    //=========================================================================
    private void Update()
    {
        float timeScale = 1.0f;
        _animator.Update(Time.deltaTime, timeScale);

        if (_animator.GetCurrentSequence() == 0)
        {
            if (_animator.GetCurrentFrame() == 3)
            {
                PlayFootstepSound();
            }
        }
    }
    //=========================================================================
    private void OnDestroy()
    {
        _animator?.Destroy();
    }
    #endregion

    #region Private Methods
    //=========================================================================
    private void PlayFootstepSound()
    {
    }
    #endregion
}
```

- **Sequential Playback:** Passing an `Animation[]` array plays each animation in order from index `0` to the end, then automatically repeats the final animation using its configured **Loop Style** (such as `Loop` or `Ping Pong`).
- **Update Loop:** Call `_animator.Update(Time.deltaTime, timeScale)` inside `Update()` (or `FixedUpdate()` if syncing with physics) to advance frames.
- **Querying Sequence & Frame:** Use `GetCurrentSequence()` to check which index in the `_animations` array is currently playing (optional if your array only contains a single animation), and `GetCurrentFrame()` to trigger frame-specific actions such as footstep audio or VFX.

