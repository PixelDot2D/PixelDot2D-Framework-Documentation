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
