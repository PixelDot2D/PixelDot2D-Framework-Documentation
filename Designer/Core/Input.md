# Input

The **Core Input** system provides centralized keybind management across keyboard and gamepad devices.

## Scene Setup

For input to function, a **KeybindManager** instance must exist in your active scene. You can set this up in one of two ways:

- Drag the **Core Managers** prefab into your scene from `PixelDot2D -> Core -> Prefabs -> Core Managers`.
- Create an empty GameObject and attach the **KeybindManager** component directly.

## Reading Input in Code

Querying an action in your scripts requires referencing `KeybindManager.m_Instance` and checking a keybind against an `Enum_InputState` (such as `Pressed`, `Held`, or `Released`):

```csharp
private KeybindManager _keybindManager => KeybindManager.m_Instance;

private void GetInput()
{
    bool isAttackPressed = _keybindManager.IsKeybindBeingUsed(Enum_PlayerKeybinds.Attack, Enum_InputState.Pressed);
}
```

## Adding New Keybinds

To register a new action (for example, `SpecialAttack`):

1. Open **`Enum_PlayerKeybinds.cs`** located at `PixelDot2D -> Core -> Input -> Enums` and add your new enum entry:

```diff
  public enum Enum_PlayerKeybinds
  {
      Jump,
      Interact,
      Attack,
+     SpecialAttack,
  }
```

2. Open each input source file under `PixelDot2D -> Core -> Input -> Input_Sources` (`InputSource_Keyboard.cs` and `InputSource_Gamepad.cs`) and register the default binding inside the **`SetDefaultBinds`** method:

```diff
  protected override void SetDefaultBinds()
  {
      // Existing bindings...
+     _keybinds.Add(Enum_PlayerKeybinds.SpecialAttack, Key.LeftShift);
  }
```

```diff
  protected override void SetDefaultBinds()
  {
      // Existing bindings...
+     _keybinds.Add(Enum_PlayerKeybinds.SpecialAttack, GamepadButton.LeftTrigger);
  }
```

> **Note:** To keep the Designer documentation concise, instructions on creating custom input devices or extending the input architecture are covered in the [Programmer Input Guide](../../Programmer/Core/Input.md).

## Rebinding & Displaying Keybinds

`KeybindManager` includes built-in support for runtime key rebinding, resetting to defaults, and formatting active keybinds into display strings for UI menus.

A typical rebinding setup splits the responsibility across two small scripts:

1. **`KeybindRebindMenu`:** Triggered by UI buttons (e.g., clicking a "Rebind Jump" or "Reset Defaults" button) to open the listener or reset bindings.
2. **`KeybindListener`:** An active UI overlay or prompt object that polls `_keybindManager.ChangeKeybind(...)` each frame until the player presses a valid key or button, then automatically disables itself.

### 1. Rebind Menu Trigger (`KeybindRebindMenu.cs`)

Attach this to your Settings or Controls menu to start a rebind request or restore default bindings:

```csharp
using PixelDot2D.Core.Enums;
using PixelDot2D.Core.InputSource;
using UnityEngine;

public class KeybindRebindMenu : MonoBehaviour
{
    #region Fields
    //SerializeFields
    [SerializeField] private KeybindListener _keybindListener;

    //Accessors
    private KeybindManager _keybindManager => KeybindManager.m_Instance;

    //Private
    private readonly Enum_PlayerKeybinds[] _allKeybinds = new Enum_PlayerKeybinds[]
    {
        Enum_PlayerKeybinds.Jump,
        Enum_PlayerKeybinds.Interact,
        Enum_PlayerKeybinds.Attack,
        Enum_PlayerKeybinds.Dash,
    };
    #endregion

    #region Public Methods
    //=========================================================================
    public void RebindJump()
    {
        StartRebind(Enum_PlayerKeybinds.Jump);
    }
    //=========================================================================
    public void RebindInteract()
    {
        StartRebind(Enum_PlayerKeybinds.Interact);
    }
    //=========================================================================
    public void ResetAllKeybinds()
    {
        _keybindManager.SetDefaultBinds();

        string formattedBinds = _keybindManager.GetKeybindAsStringWithKeybindName(" / ", ", ", _allKeybinds);
        Debug.Log($"Reset keybinds! {formattedBinds}");
    }
    #endregion

    #region Private Methods
    //=========================================================================
    private void StartRebind(Enum_PlayerKeybinds keyToChange)
    {
        if (_keybindListener.gameObject.activeInHierarchy)
        {
            return;
        }

        _keybindListener.Init(keyToChange);
    }
    #endregion
}
```

### 2. Active Input Listener (`KeybindListener.cs`)

Attach this to your "Press any key..." UI prompt object. It stays disabled until `Init(...)` is called, listens for the next input via `ChangeKeybind`, and hides itself once the binding succeeds:

```csharp
using PixelDot2D.Core.Enums;
using PixelDot2D.Core.InputSource;
using UnityEngine;

public class KeybindListener : MonoBehaviour
{
    #region Fields
    //Accessors
    private KeybindManager _keybindManager => KeybindManager.m_Instance;

    //Private
    private Enum_PlayerKeybinds _keyToChange = Enum_PlayerKeybinds.Interact;
    #endregion

    #region MonoBehaviour
    //=========================================================================
    private void Awake()
    {
        gameObject.SetActive(false);
    }
    //=========================================================================
    private void Update()
    {
        if (_keybindManager == null)
        {
            gameObject.SetActive(false);
            return;
        }

        if (_keybindManager.ChangeKeybind(_keyToChange))
        {
            string updatedBind = _keybindManager.GetKeybindAsStringWithKeybindName(" / ", ", ", _keyToChange);
            Debug.Log($"Successfully changed key! {updatedBind}");

            gameObject.SetActive(false);
        }
    }
    #endregion

    #region Public Methods
    //=========================================================================
    public void Init(Enum_PlayerKeybinds keyToChange)
    {
        _keyToChange = keyToChange;
        gameObject.SetActive(true);
    }
    #endregion
}
```
### Helper Methods Reference

- **`ChangeKeybind(Enum_PlayerKeybinds)`:** Listens for the next key or gamepad button press and assigns it to the specified action, returning `true` on the frame a new bind is captured.
  - *Automatic Swap Behavior:* If the newly pressed key or button is already assigned to another action, `KeybindManager` automatically swaps their bindings so no action is ever left unbound (for example, if **Attack** is bound to `Left Click` and the player rebinds **Interact** to `Left Click`, **Attack** automatically receives **Interact**'s previous binding).
- **`SetDefaultBinds()`:** Restores all keybinds back to the defaults defined in your `InputSource_*` classes.
- **`GetKeybindAsStringWithKeybindName(deviceSeparator, bindSeparator, params Enum_PlayerKeybinds[])`:** Formats one or more actions alongside their bound keys into a UI-ready string (for example, separating Keyboard and Gamepad binds with `" / "` and multiple actions with `", "`).


