# Combat Summary

The PixelDot2D Combat system is built out of modular, LEGO-like ScriptableObject parts rather than hardcoded weapon classes. By combining four independent stages, you can assemble nearly any weapon or attack behavior directly in the Unity Inspector without writing custom weapon scripts.

Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub** to generate new weapon configurations and execution modules.

## The 4 Combat Stages

Every weapon or attack runs through four modular layers:

1. **Virtual Transform:** Defines the origin position and a single float rotation angle for the attack, decoupled from actual Unity GameObject transforms.
2. **Aiming:** Modifies or offsets the Virtual Transform without overwriting its base orientation. For example, if the Virtual Transform angle is `0` and an Aiming module applies random spread between `-5` and `10`, the output evaluates as `0 + randomOffset` so the Virtual Transform never loses its true reference angle.
3. **Gate:** Controls when and how the weapon is allowed to fire (such as cooldown timers, ammo consumption, or charge-up thresholds).
4. **Execution:** Defines what the weapon actually spawns or evaluates once the gate opens:
   - **Hitscan:** Instant ray/line checks.
   - **Projectile:** Spawned, pooled objects moving through space.
   - **Hitbox:** Spatial overlap checks for melee swings or area effects.

## Composition Example: Flaming Sword of Speed

Instead of coding a dedicated `FlamingSword` class, you compose it in the editor from existing blocks:

- **Virtual Transform (Mirrored Static):** Uses a static position offset (for example, `X = 5` when facing right) and automatically mirrors both position and rotation angles to the left when a side-scroller character turns around.
- **Aiming (Static Angle):** Sets a fixed angle offset (such as `-90` for an overhead swing or `0` for a straight thrust).
- **Gate (Cooldown):** Enforces a minimum delay between swings.
- **Execution (Hitbox):** Performs a melee overlap check. Inside the Execution ScriptableObject, set the damage type flag enum to `Fire` and the stat scaling field to `Speed`.

The result is a fully functional Flaming Sword of Speed assembled entirely through data assets.

## Execution Guides

Each execution type requires a slightly different asset and scene setup. Refer to the dedicated guides for step-by-step instructions:

- [Hitscans](Hitscans.md)
- [Projectiles](Projectiles.md)
- [Hitboxes](Hitboxes.md)

