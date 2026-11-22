# Template Combat Entity and Prefabs

Because the Combat system is decoupled from visuals and character controllers, scenes require a few lightweight manager prefabs (primarily object poolers) to handle runtime execution.

You can find all required prefabs under **PixelDot2D -> Combat -> Prefabs**.

## Scene Manager Prefabs

For ease of setup, drop the **Combat Managers** prefab into your scene. It includes the following decoupled poolers:

- **HitBox Sprite Visual Object Pool:** Because hitboxes are purely virtual spatial checks decoupled from visuals, this pool spawns and recycles visual indicators independently.
- **HitScan Visual:** Reports start point, end point, and width/thickness so a decoupled visual (such as the provided demo `LineRenderer`) can draw the trace.
- **Projectile Object Pool:** Pre-allocates projectile shells and reuses them at runtime by swapping out blueprint and execution data rather than instantiating new GameObjects.

## Combat Entity Template

Located at **PixelDot2D -> Combat -> Prefabs -> Combat Entity Template**, this prefab serves as a general-purpose combat actor. Because its behavior is driven entirely by swappable data assets, it can function as an enemy, turret, boss, or moving hazard.

<img width="848" height="841" alt="image" src="https://github.com/user-attachments/assets/d0528d50-abb4-409e-9261-f3b8c540e384" />

### Inspector Fields

- **Movement Patterns:** The sequence driving how this entity moves and when it executes weapons (if applicable).
- **Points Parent:** Used with the `RB2DMovement` **Follow Array of Vector2s Path** movement type. Each parent holds child GameObjects whose positions are snapshotted into `Vector2` coordinates during `Awake` for patrol routes or shmup attack paths.
- **Fire Point:** The `Transform` origin used for firing weapons. Defaults to the entity itself, or can point to a custom child Transform.
- **Target:** The target `Transform` referenced by target-seeking Virtual Transforms, Aiming modules, or movement types. If left `null`, target-dependent modules gracefully fall back to static behavior (such as firing straight right or moving along the spawn angle).
- **RB & SR:** References to the entity's `Rigidbody2D` and `SpriteRenderer`.
- **Obstacles & Valid Layer Enum Array:** Defines which physics layers count as walls/obstacles or valid hit targets.
  - *Important Setup:* Open **PixelDot2D -> Core -> Scripts -> Enums -> Enum_EditorLayers.cs** and ensure the enum integer values match your Unity Project Settings layers (for example, if `Player` is Layer `8` in Unity, `Enum_EditorLayers.Player` must equal `8`).
 
## Creating Entity Movement Patterns

To generate the assets assigned to the entity's **Movement Patterns** field, use the Combat Sequence Matrix macro:

1. Create and select a target folder in the Project window.
2. Press **Ctrl + E** or right-click in the Project window and select **PixelDot2D -> Open Advanced Creation Hub**.
3. Navigate to **Macros -> Combat -> Bulk Creation -> Build Combat Sequence Matrix**.

<img width="1132" height="752" alt="image" src="https://github.com/user-attachments/assets/2fb14087-d34f-4998-b204-c6b221233298" />

### Generated Folder Structure

<img width="408" height="167" alt="image" src="https://github.com/user-attachments/assets/f8817fac-9939-48c2-b4f6-5e4dbc5a7572" />

- **Movements:** Starts empty. Populate it with any Core movement assets created via **Creation Hub -> Core -> RB2D Movement Manager**. All movement types work here except the Blueprint asset (which is only a container).
- **Combat_Sequence:** The main sequencer asset you drag into the entity's **Movement Patterns** field to drive both movement and weapon execution.

<img width="1135" height="766" alt="image" src="https://github.com/user-attachments/assets/425a3da6-bdc6-4523-b994-ac8a46307aac" />

### Combat_Sequence Fields

Each outer index in `Combat_Sequence` represents a phase or step in the entity's behavior:

- **Index Name:** Designer-facing label to keep phases organized in the Inspector.
- **Animation:** The animation clip played during this outer index.
- **Movement Data:** An array of Core `RB2DMovement` assets played sequentially during this phase.
- **Frames To Try Execute:** Specific animation frames on which to attempt firing the weapon. If left empty, the entity attempts to execute the weapon every frame while respecting the weapon's **Gate** rules.
- **Weapon:** The `Multi_Weapon` asset executed during this outer index.

You can add as many movement steps inside each inner array and as many outer phases as needed—they always execute sequentially in order with no cap on array length.

<img width="865" height="781" alt="image" src="https://github.com/user-attachments/assets/4ad83c59-0490-4e05-a4b2-50f4f7211156" />


## C# Wiring & Combat Interfaces

`CombatEntityTemplate.cs` demonstrates the minimal C# setup required to run `Sequence_CombatManager` and implement the two core combat interfaces: `IWeaponizable` (for entities that attack) and `IDamageable` (for entities that receive hits).

```csharp
using PixelDot2D.Combat.Enums;
using PixelDot2D.Combat.Interfaces;
using PixelDot2D.Combat.Movement;
using PixelDot2D.Core.Animation;
using PixelDot2D.Core.Enums;
using PixelDot2D.Core.Extensions;
using UnityEngine;

namespace PixelDot2D.Combat
{
    public class CombatEntityTemplate : MonoBehaviour, IWeaponizable, IDamageable
    {
        #region Fields
        [SerializeField] private SO_Sequence_CombatMovement _movementPatterns;
        [SerializeField] private GameObject _pointParent;
        [SerializeField] private GameObject _target;
        [SerializeField] private Transform _firePoint;
        [SerializeField] private Rigidbody2D _rb;
        [SerializeField] private SpriteRenderer _sr;
        [SerializeField] private Enum_EditorLayers[] _obstacles;
        [SerializeField] private Enum_EditorLayers[] _validTargets;

        private AnimationPlayer2D _animationPlayer;
        private int _obstaclesMask = 0;
        private int _validTargetsMask = 0;
        private int _currentJaggedArrayIndex;
        private Vector2[][] _followPoints;
        private GameObject[] _topLayerChildren;
        private Sequence_CombatManager _movement;
        private readonly Vector2 _moveSpeedMultiplier = new(1.0f, 1.0f);
        #endregion

        #region MonoBehaviour
        //=========================================================================
        private void OnDestroy()
        {
            _movement?.Destroy();
            _animationPlayer?.Destroy();
        }
        //=========================================================================
        private void Awake()
        {
            _animationPlayer = new();
            _animationPlayer.Init(_sr);
            _validTargetsMask = _validTargetsMask.GetEditorLayerBitShifted(_validTargets);
            _obstaclesMask = _obstaclesMask.GetEditorLayerBitShifted(_obstacles);
            _movement = new();
            _movement.Init(this, _firePoint, _animationPlayer, _rb);
            _topLayerChildren = _pointParent.transform.GetChildren();
            SavePathPoints();
            _currentJaggedArrayIndex = -1;
            _movement.m_Movement.m_GetMultiPointPath -= GetPathToFollow;
            _movement.m_Movement.m_GetMultiPointPath += GetPathToFollow;
            _movement.Enter(_movementPatterns);
        }
        //=========================================================================
        private void FixedUpdate()
        {
            _movement.FixedUpdate(Time.deltaTime, Vector2.up, Vector2.right, _moveSpeedMultiplier);
            _animationPlayer.Update(Time.deltaTime);
        }
        #endregion

        #region nonMonoBehaviour
        //=========================================================================
        public DamageData IWeaponizable_GetDamage(DamageData damageData)
        {
            damageData.m_SchoolFlags = Enum_SchoolFlags.Ice;
            // Base damage and school flags are already pre-filled from your Weapon ScriptableObject.
            // Modify or add (+=) to them here to apply character stats, buffs, or overrides.
            damageData.m_Damage += 1.0f;

            return damageData;
        }
        //=========================================================================
        public Enum_SideScrollerFacingDirection IWeaponizable_GetFacingDirection()
        {
            return Enum_SideScrollerFacingDirection.Left;
        }
        //=========================================================================
        public float IDamageable_TakeDamage(DamageData damageData)
        {
            IWeaponizable source = damageData.m_Attacker;

            if (damageData.m_Damage > 0.0f)
            {
                DamageReport damageReport = new()
                {
                    m_DamageReceiver = this,
                    m_DamageDone = damageData.m_Damage,
                    m_SchoolFlags = damageData.m_SchoolFlags,
                    m_ContactPoint = damageData.m_ContactPoint,
                };

                source.IWeaponizable_OnSuccessfulDamageDone(damageReport);
            }

            return damageData.m_Damage;
        }
        //=========================================================================
        public void IWeaponizable_OnSuccessfulDamageDone(DamageReport damageReport)
        {
        }
        //=========================================================================
        private void SavePathPoints()
        {
            _followPoints = new Vector2[_topLayerChildren.Length][];

            for (int i = 0; i < _followPoints.Length; i++)
            {
                _followPoints[i] = new Vector2[_topLayerChildren[i].transform.childCount];

                for (int c = 0; c < _topLayerChildren[i].transform.childCount; c++)
                {
                    _followPoints[i][c] = _topLayerChildren[i].transform.GetChild(c).gameObject.transform.position;
                }
            }
        }
        //=========================================================================
        public Vector2[] GetPathToFollow()
        {
            _currentJaggedArrayIndex++;
            _currentJaggedArrayIndex = _currentJaggedArrayIndex.ClampOverflow(0, _followPoints.Length - 1);

            return _followPoints[_currentJaggedArrayIndex];
        }
        //=========================================================================
        public int IWeaponizable_GetObstacleLayers()
        {
            return _obstaclesMask;
        }
        //=========================================================================
        public int IWeaponizable_GetValidTargetLayers()
        {
            return _validTargetsMask;
        }
        //=========================================================================
        public (bool isValidCoordinates, Vector2 coordinates) IWeaponizable_GetTargetCoordinates()
        {
            if (_target == null)
            {
                return (false, Vector2.zero);
            }

            return (true, _target.transform.position);
        }
        //=========================================================================
        public Enum_EntityType IDamageable_GetEntityType()
        {
            return Enum_EntityType.Inanimate_Object;
        }
        #endregion
    }
}
```

### Key Interface Methods Explained

#### 1. Outgoing Damage & Stat Scaling (`IWeaponizable_GetDamage`)
Before an attack resolves, the weapon passes a `DamageData` struct populated with the Execution ScriptableObject's raw values into `IWeaponizable_GetDamage`:

```csharp
public struct DamageData
{
    public float m_Damage;
    public float m_ScalingAmount;
    public Vector2 m_ContactPoint;
    public Enum_Stat m_StatScaling;
    public IWeaponizable m_Attacker;
    public IDamageable m_IDamageable;
    public Enum_SchoolFlags m_SchoolFlags;
}
```

Because Combat is decoupled from characters, the framework does not calculate stat scaling automatically. Inside `IWeaponizable_GetDamage`, you can read `m_StatScaling` and `m_ScalingAmount` to apply your own stat math—or override `m_Damage` and `m_SchoolFlags` directly as shown in the template.

#### 2. Facing Direction (`IWeaponizable_GetFacingDirection`)
Returns `Enum_SideScrollerFacingDirection` (`Left` or `Right`). Mirrored Virtual Transforms and Aiming modules query this method at fire time to flip offsets and angles automatically.

#### 3. The Blind Damage Handshake (`IDamageable_TakeDamage` & `IWeaponizable_OnSuccessfulDamageDone`)
Because attackers and targets do not know each other's concrete types (a projectile could hit a player, an enemy, or a destructible crate), damage uses a two-step blind handshake:

1. **Incoming Hit (`IDamageable_TakeDamage`):** The target receives `DamageData`, applies its own defense, resistances, or armor math, and subtracts HP.
2. **Confirmation Callback (`IWeaponizable_OnSuccessfulDamageDone`):** If actual damage was dealt (`> 0`), the receiver constructs a `DamageReport` containing the final `m_DamageDone`, `m_SchoolFlags`, and `m_ContactPoint`, and sends it back to `damageData.m_Attacker`. The attacker can then trigger on-hit effects (such as lifesteal, hit-stop, or combo counters) without ever knowing what kind of entity it hit.

#### 4. Entity Type Classification (`IDamageable_GetEntityType`)
Returns an `Enum_EntityType` value (such as `Inanimate_Object`). Attackers or systems can inspect this to distinguish between living targets and environmental props.

## Automatic Data Population

`DamageData` is populated internally by the framework at execution time. You do not need to manually copy or transfer values between weapon ScriptableObjects and hit receivers—the framework automatically routes the correct data (`m_Attacker`, `m_ContactPoint`, `m_SchoolFlags`, base damage, and scaling parameters) through the pipeline.

