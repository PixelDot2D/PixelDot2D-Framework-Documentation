# Saving & Loading

The **Core Saving & Loading** system is built to handle any game's save data without forcing rigid data models or hardcoded return types. Instead of freezing the main thread during file I/O, saving and loading are split into three distinct lifecycle phases:

1. **Pre-Save / Pre-Load (Main Thread):** Runs on Unity's main thread before background file operations start. Use this phase to read Unity objects (such as `transform.position`) into thread-safe primitive fields (`float`, `int`, `bool`, `string`) or prepare scene state before loading.
2. **Save / Load (Background Thread):** Runs off the main thread using `BinaryWriter` and `BinaryReader`. Only read/write your cached primitive fields here—never touch Unity `Transform`, `GameObject`, or scene APIs inside this phase.
3. **On Save Complete / On Load Complete (Main Thread):** Runs back on Unity's main thread once file I/O finishes. Use this phase to apply loaded values back onto Unity objects (such as moving the player to their loaded position or refreshing UI).

## Scene Setup

1. Drag the **Core Managers** prefab into your scene from `PixelDot2D -> Core -> Prefabs -> Core Managers`, or attach the **SaveAndLoadManager** script directly to a single GameObject.
2. **Important:** Only **one** `SaveAndLoadManager` should exist in the scene at a time.
3. Drag any GameObject containing an `ISaveableAndLoadable` script into the **SaveAndLoadManager** Inspector list. Execution order and phase timing are handled automatically by the framework.

<img width="925" height="682" alt="image" src="https://github.com/user-attachments/assets/e41594e2-17ca-4400-bdad-9d9da9803b9a" />


> **Pre-Build Validation Tip:** In the **SaveAndLoadManager** Inspector, click **Clean I Saveable and Loadable Objects Collection** after assigning objects—and especially before making a build. This editor tool validates your list, automatically removes duplicate entries, and pings a warning message of any invalid objects in the Hierarchy so you can fix them early. Because this validation check is stripped from standalone builds for maximum runtime speed, running it beforehand prevents missing or duplicate save references.

### Inspector Version Parameters

In the **SaveAndLoadManager** Inspector, you can configure three version numbers that are saved into the file header and passed into every `ISaveableAndLoadable` lifecycle method:

- **Major Patch Version:** Increment this when shipping a major feature update or expansion (e.g., `1.x.x -> 2.x.x`). Adding major features often introduces new save data that will also require bumping **Load Version**.
- **Load Version:** Tracks changes to how save files are **loaded and migrated**. Because saving always rewrites the entire file from scratch in the latest format, `Save` never needs legacy branches—only `Load` needs to check `loadVersion` to read older files and migrate them forward. Increment this whenever you add new fields or change how existing save data is read.
- **Minor Patch Version:** Increment this for small bug fixes, hotfixes, or balance tweaks that do not affect save data structure (e.g., `1.0.1 -> 1.0.2`).

## Registering a Save Key

Every saveable system requires a unique identifier in **`Enum_ISaveableAndLoadableKey.cs`**, located at `PixelDot2D -> Core -> Scripts -> SaveAndLoading -> Enums`.

Open the file and append your new key to the bottom of the enum:

```diff
  public enum Enum_ISaveableAndLoadableKey
  {
      Keybinds = 0,
      InventoryExample,
+     PlayerData,
  }
```

> **Critical Versioning Rules:**
> - **Never reorder or re-index existing keys** once production builds are released. Changing the enum order misaligns the binary stream and will immediately corrupt player save files.
> - **Never delete an obsolete key.** If a feature is removed later, keep its enum entry, suffix it with `_Deprecated` (for example, `OldQuestSystem_Deprecated`), and continue reading its old bytes in `Load` so the binary stream position stays aligned for subsequent keys.

## Implementing `ISaveableAndLoadable` in C#

A single `ISaveableAndLoadable` script does not have to represent just one object—it can act as a **coordinator** that references and serializes an entire group of objects or subsystems (such as a player's position, health, currency, and unlocked abilities) under a single key.

Implement **`ISaveableAndLoadable`** on a MonoBehaviour and populate the three save/load phases:

```csharp
using PixelDot2D.Core.Enums;
using PixelDot2D.Core.Interfaces;
using System.IO;
using UnityEngine;

public class PlayerSaveHandler : MonoBehaviour, ISaveableAndLoadable
{
    #region Fields
    //Private
    private Vector2 _position;
    #endregion

    #region ISaveableAndLoadable
    //=========================================================================
    public Enum_ISaveableAndLoadableKey ISaveableAndLoadable_GetKey()
    {
        return Enum_ISaveableAndLoadableKey.PlayerData;
    }
    //=========================================================================
    public void ISaveableAndLoadable_OnPreLoad(int majorPatchVersion, int loadVersion, int minorPatchVersion)
    {
    }
    //=========================================================================
    public void ISaveableAndLoadable_Load(BinaryReader reader, int majorPatchVersion, int loadVersion, int minorPatchVersion)
    {
        _position.x = reader.ReadSingle();
        _position.y = reader.ReadSingle();
    }
    //=========================================================================
    public void ISaveableAndLoadable_OnLoadComplete(int majorPatchVersion, int loadVersion, int minorPatchVersion)
    {
        transform.position = _position;
    }
    //=========================================================================
    public void ISaveableAndLoadable_OnPreSave(int majorPatchVersion, int loadVersion, int minorPatchVersion)
    {
        _position.x = transform.position.x;
        _position.y = transform.position.y;
    }
    //=========================================================================
    public void ISaveableAndLoadable_Save(BinaryWriter writer, int majorPatchVersion, int loadVersion, int minorPatchVersion)
    {
        writer.Write(_position.x);
        writer.Write(_position.y);
    }
    //=========================================================================
    public void ISaveableAndLoadable_OnSaveComplete(int majorPatchVersion, int loadVersion, int minorPatchVersion)
    {
    }
    #endregion
}
```

### Key Serialization Rules

- **Match Read & Write Order:** Every `writer.Write(...)` call inside `ISaveableAndLoadable_Save` must have an exact matching `reader.Read...()` call in the exact same order inside `ISaveableAndLoadable_Load`.
- **Patch Versioning Parameters:** All six lifecycle methods receive `majorPatchVersion`, `loadVersion`, and `minorPatchVersion`, allowing you to conditionally read legacy formats or migrate older save files when you add new fields in future game updates.

