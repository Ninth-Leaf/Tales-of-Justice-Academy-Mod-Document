# Official Mod Guide to Cultivation Methods, Skills, Equipment, and Combat Effects

This guide is for authors creating battle content Mods. It covers the four content types supported in the current version:

- Cultivation Methods
- Skills
- Equipment
- Standalone Combat Effects

It also covers the shared parts used by all four types:

- `manifest.json` format
- Localization and icon keys
- Expression context
- Full Tracker and BuildData reference

## 1. manifest.json

Mods are loaded through the root-level `manifest.json`. This guide only uses three entries:

- `ConfigMaps`: register config files
- `LocalizationMaps`: register localization files
- `SpriteMaps`: register icon resources

Example:

```json
{
  "ConfigMaps": [
    {
      "FullName": "Xinfa.Player.追锋诀",
      "Path": "Configs/Xinfa/Player/追锋诀.json"
    },
    {
      "FullName": "Battle.Action.轻剑.顺劈",
      "Path": "Configs/Battle/Action/轻剑/Battle.Action.轻剑.顺劈.json"
    },
    {
      "FullName": "Item.Equipment.Weapon.轻剑.破阵剑",
      "Path": "Configs/Item/Equipment/Weapon/轻剑/Item.Equipment.Weapon.轻剑.破阵剑.json"
    },
    {
      "FullName": "Battle/Effect/战意",
      "Path": "Configs/Battle/Effect/战意.json"
    }
  ],
  "LocalizationMaps": [
    {
      "Locale": "zh-Hans",
      "Path": "Localization/zh-Hans/content.json"
    }
  ],
  "SpriteMaps": [
    {
      "FullName": "Xinfa.Player.追锋诀.Icon.Sprite",
      "Path": "Sprites/追锋诀.png"
    },
    {
      "FullName": "Battle.Action.轻剑.顺劈.Icon.Sprite",
      "Path": "Sprites/顺劈.png"
    },
    {
      "FullName": "Item.Equipment.Weapon.轻剑.破阵剑.Icon.Sprite",
      "Path": "Sprites/破阵剑.png"
    }
  ]
}
```

Rules:

- `FullName` is the runtime lookup key and must follow the naming rules for its content type.
- `Path` is relative to the Mod root.
- Cultivation Methods, Skills, and Equipment are registered directly through `ConfigMaps`.
- Standalone Combat Effects are registered through `ConfigMaps` as `Battle/Effect/EffectName`.

## 2. Shared Rules

### 2.1 FullName

The four content types use these naming rules:

- Cultivation Method: `Xinfa.Player.追锋诀`
- Character Cultivation Method: `Xinfa.{Character}.{Name}`
- Skill: `Battle.Action.{Weapon}.{Name}`
- Weapon: `Item.Equipment.Weapon.{Weapon}.{Name}`
- Accessory: `Item.Equipment.Accessory.{Name}`
- Standalone Combat Effect registration key: `Battle/Effect/EffectName`

When a standalone Combat Effect is referenced from another config, use the effect name itself without the `Battle/Effect/` prefix. Example:

```json
"FlexibleEffectNames": ["战意"]
```

### 2.2 Localization

Localization files use a `Tables` array. The current version uses four tables for battle content:

- `Xinfa`
- `BattleAction`
- `Item`
- `Effect`

Example:

```json
{
  "Tables": [
    {
      "Table": "Xinfa",
      "Entries": [
        { "Key": "Xinfa.Player.追锋诀", "Value": "追锋诀" },
        { "Key": "Xinfa.Player.追锋诀.Description", "Value": "一门围绕追击构筑的心法。" },
        { "Key": "Xinfa.Player.追锋诀.Effect.0", "Value": "防御力提升30" },
        { "Key": "Xinfa.Player.追锋诀.Effect.1", "Value": "追击造成的伤害提升10%" }
      ]
    },
    {
      "Table": "BattleAction",
      "Entries": [
        { "Key": "Battle.Action.轻剑.顺劈", "Value": "顺劈" },
        { "Key": "Battle.Action.轻剑.顺劈.Description", "Value": "命中后获得1层战意。" }
      ]
    },
    {
      "Table": "Item",
      "Entries": [
        { "Key": "Item.Equipment.Weapon.轻剑.破阵剑", "Value": "破阵剑" },
        { "Key": "Item.Equipment.Weapon.轻剑.破阵剑.Description", "Value": "一柄偏重破甲的轻剑。" },
        { "Key": "Item.Equipment.Weapon.轻剑.破阵剑.Effect", "Value": "攻击力提升18，轻剑招式造成伤害提升10%" }
      ]
    },
    {
      "Table": "Effect",
      "Entries": [
        { "Key": "Effect.战意", "Value": "战意" },
        { "Key": "Effect.战意.Description", "Value": "每层使攻击力、防御力、速度提升1%" }
      ]
    }
  ]
}
```

Text key rules:

- Cultivation Method name: `Xinfa...`
- Cultivation Method description: `Xinfa....Description`
- Cultivation Method level effect: `Xinfa....Effect.0`, `Effect.1`, `Effect.2`
- Skill name: `Battle.Action...`
- Skill description: `Battle.Action....Description`
- Equipment name: `Item.Equipment...`
- Equipment description: `Item.Equipment....Description`
- Equipment summary: `Item.Equipment....Effect`
- Effect name: `Effect.EffectName`
- Effect description: `Effect.EffectName.Description`

### 2.3 Icon Keys

The icon key format is fixed:

- `${fullName}.Icon.Sprite`

This rule applies to:

- Cultivation Methods
- Skills
- Equipment

Standalone Combat Effects do not use this icon key pattern.

### 2.4 Expression Context

`If`, `Expression`, `Call`, `CallList`, and `CallDict` all use the same battle expression context. The current version exposes these variables directly:

- `unit`: the current unit in a unit-based callback
- `source`: the event initiator
- `action`: the current Skill
- `targets`: the target list
- `currentTarget`: the current target in per-target processing
- `reason`: the reason object
- `reasonAsAction`: the reason interpreted as a Skill
- `reasonAsEffect`: the reason interpreted as an Effect
- `target`: the single target object
- `effect`: the current Effect
- `owner`: the owner of the Effect
- `effectSource`: the source unit of the Effect
- `context`: the current flexible effect context
- `damage`: the current damage value
- `stage`: the current stage value
- `increaseValue`: the Secret Action Power delta
- `layer`: layer count
- `type`: the damage type string

### 2.5 Shared BuildData Rules

All BuildData types have three base fields:

- `Uuid`
- `If`
- `Target`

`BuildDataName` uses this fixed format:

```json
"BuildDataName": "FullTypeName_UUID"
```

For example:

```json
"BuildDataName": "WuXia.Battle.SaveDoubleBuildData_f13c3d1b-d5aa-4083-bbea-4820bd849ba9"
```

Base target selectors for `Target`:

- `Source`
- `Target`
- `SourceTeam`
- `TargetTeam`
- `Owner`
- `EffectSource`
- `Named`

You can append filters to `Target` with `;`:

- `Self`
- `Opponent`
- `LowestHP`
- `predicate:{Expression}`

Example:

```json
"Target": "TargetTeam;LowestHP"
```

```json
"Target": "Target;predicate:predicateTarget.HasEffect(\"战意\")"
```

### 2.6 Flexible Effect Context

Cultivation Method effects, Equipment effects, and standalone Combat Effects all use `IEffect.Data`. Its `Context` field is:

```json
{
  "MutexGroup": "SampleMutexGroup",
  "MutexGroupGlobalCountLimit": 1,
  "Uuid": "b6a1f5cb-6dc4-43f0-b8d0-6f2eb9a7f1b8",
  "MaxLayer": 20,
  "ScaleAttributeWithLayer": true,
  "Priority": 100,
  "TypedBuildDatas": [],
  "ApplyRecoveryBuildDatas": [],
  "AddSecretActionPowerBuildDatas": [],
  "AddEffectBuildDatas": [],
  "RemoveEffectBuildDatas": [],
  "AddEffectLayerBuildDatas": [],
  "TriggerDotBuildDatas": [],
  "ExpressionBuildDatas": [],
  "SaveIntBuildDatas": [],
  "SaveDoubleBuildDatas": [],
  "SummonBuildDatas": [],
  "ModifyUnitProgressBuildDatas": [],
  "OnDamageBuildDatas": [],
  "UnitContinueAttackBuildDatas": [],
  "UnitInvokeFollowAttackBuildDatas": []
}
```

Rules:

- `TypedBuildDatas` decides which Tracker each BuildData is bound to.
- Higher `Priority` runs earlier inside the Tracker.
- When `MaxLayer > 0`, the Effect enters a stacking context.
- When `ScaleAttributeWithLayer = true`, attribute-type effects scale by layer count and must be used together with `MaxLayer`.
- `MutexGroup` and `MutexGroupGlobalCountLimit` control shared global limits for mutually exclusive groups.

## 3. Cultivation Methods

### 3.1 Basic Structure

```json
{
  "ObjectType": "WuXia.Attributes.Xinfa",
  "Name": "追锋诀",
  "Tier": 1,
  "Slot": 4,
  "FlexibleLevelEffects": [
    {
      "Description": "防御力提升30",
      "Attribute": {
        "Values": [
          {
            "Name": "Defense",
            "Add": 30
          }
        ]
      }
    },
    {
      "Description": "追击造成的伤害提升10%",
      "Context": {
        "TypedBuildDatas": [
          {
            "TrackerType": "WuXia.Battle.ICalculateApplyDamageTracker",
            "BuildDataName": "WuXia.Battle.SaveDoubleBuildData_f13c3d1b-d5aa-4083-bbea-4820bd849ba9"
          }
        ],
        "SaveDoubleBuildDatas": [
          {
            "Uuid": "f13c3d1b-d5aa-4083-bbea-4820bd849ba9",
            "Expression": "reasonAsAction != null && reasonAsAction.IsFollowAttack ? 0.1 : 0"
          }
        ]
      }
    }
  ],
  "Attributes": ["LiDao"]
}
```

### 3.2 Field Rules

- `ObjectType` is fixed as `WuXia.Attributes.Xinfa`
- `Name` is the Cultivation Method name without a prefix
- `Tier` is the rarity tier
- `Slot` is the slot cost
- `FlexibleLevelEffects` maps in order to level 1, level 2, level 3, and so on
- `Attributes` is used to auto-generate upgrade thresholds
- `LevelAttributeThresholds` is used to specify upgrade thresholds manually

### 3.3 Upgrade Thresholds

When `LevelAttributeThresholds` is empty and `Attributes` is not empty, the current version auto-generates thresholds from the game rule table:

- `Tier 1`: `[75]`
- `Tier 2`: `[100, 200]`
- `Tier 3`: `[200, 250, 300]`

Manual threshold format:

```json
"LevelAttributeThresholds": [
  {
    "Values": [
      { "Name": "JianRen", "Value": 90 },
      { "Name": "BingYi", "Value": 90 }
    ]
  },
  {
    "Values": [
      { "Name": "JianRen", "Value": 180 },
      { "Name": "BingYi", "Value": 180 }
    ]
  }
]
```

For units without growth attribute data, the active Cultivation Method level is simply the number of `FlexibleLevelEffects`.

### 3.4 Text Keys

The Cultivation Method UI reads:

- `Xinfa.{Character}.{Name}`
- `Xinfa.{Character}.{Name}.Description`
- `Xinfa.{Character}.{Name}.Effect.0`
- `Xinfa.{Character}.{Name}.Effect.1`

## 4. Skills

### 4.1 Basic Structure

```json
{
  "ObjectType": "WuXia.Battle.Actions.FlexibleAction",
  "Name": "顺劈",
  "Weapon": "轻剑",
  "Cost": 1,
  "Duan": 1,
  "DuanUpdate": "Increase",
  "AttackScale": 1.0,
  "PostureDamage": 10,
  "Range": "Single",
  "IsSelfEnable": false,
  "IsOpponentEnable": true,
  "IsDiedEnable": false,
  "BeforeStartBuildDataNames": [],
  "AfterStartBuildDataNames": [],
  "InBuildDataNames": [],
  "BeforeEndBuildDataNames": [],
  "AfterEndBuildDataNames": [],
  "AddEffectBuildDatas": []
}
```

### 4.2 Field Rules

- `ObjectType` is fixed as `WuXia.Battle.Actions.FlexibleAction`
- `Name` is the Skill name
- `Weapon` is the weapon type
- `Cost` is the Inner Energy cost
- When `SecretActionPowerCost > 0`, the Skill is a Step EX
- When `IsCounterAttack = true`, the Skill is a Counterattack
- When `IsFollowAttack = true`, the Skill is a Follow-Up
- Regular active Skills use `Duan`
- `DuanUpdate` supports `Keep`, `Increase`, `Decrease`, and `Final`
- `AttackScale` is the base damage multiplier
- `PostureDamage` is the Poise Damage value

### 4.3 Target Range

`Range` is `string -> ActionSelectionRange`.

`ActionSelectionRange` supports:

| Value | Meaning |
| --- | --- |
| `Self` | Target self |
| `Single` | Single target |
| `Row` | Targets in the same row as the current tile |
| `Column` | Targets in the same column as the current tile |
| `All` | All valid targets on the currently selectable side |
| `Front` | The front-row position that matches the current tile |
| `SelfLowestHP` | The unit with the lowest HP on the currently selectable side |
| `Expression` | Return an `ActionSelectionRange` enum through an expression |

### 4.4 Stages

Each Skill has five fixed stages, executed in order:

1. `BeforeStart`
2. `AfterStart`
3. `In`
4. `BeforeEnd`
5. `AfterEnd`

The matching fields are:

- `BeforeStartBuildDataNames`
- `AfterStartBuildDataNames`
- `InBuildDataNames`
- `BeforeEndBuildDataNames`
- `AfterEndBuildDataNames`

### 4.5 Supported BuildData Arrays on Skills

The current version supports these arrays in Skill configs:

- `ApplyRecoveryBuildDatas`
- `OnDamageBuildDatas`
- `AddSecretActionPowerBuildDatas`
- `AddEffectBuildDatas`
- `RemoveEffectBuildDatas`
- `AddEffectLayerBuildDatas`
- `TriggerDotBuildDatas`
- `UnitContinueAttackBuildDatas`
- `UnitInvokeFollowAttackBuildDatas`
- `ModifyUnitProgressBuildDatas`
- `SummonBuildDatas`
- `ExpressionBuildDatas`
- `SaveIntBuildDatas`
- `SaveDoubleBuildDatas`

Rules:

- Stage arrays contain `BuildDataName`
- The actual definitions are written in the matching typed arrays
- Keep every `Uuid` unique

### 4.6 Text Keys

The Skill UI reads:

- `Battle.Action.{Weapon}.{Name}`
- `Battle.Action.{Weapon}.{Name}.Description`

In the current version:

- `.Description` holds extra effects, trigger conditions, and added statuses
- Base info such as range, multiplier, and Step/Stage info is generated by the UI from Skill data

## 5. Equipment

### 5.1 Weapons

```json
{
  "ObjectType": "WuXia.Items.Equipments.Weapon",
  "FullName": "Item.Equipment.Weapon.轻剑.破阵剑",
  "Price": 600,
  "Level": 1,
  "Effects": [
    {
      "Name": "破阵剑",
      "Description": "攻击力提升18",
      "Attribute": {
        "Values": [
          { "Name": "Attack", "Add": 18 }
        ]
      }
    },
    {
      "Description": "轻剑招式造成伤害提升10%",
      "Context": {
        "TypedBuildDatas": [
          {
            "TrackerType": "WuXia.Battle.ICalculateApplyDamageTracker",
            "BuildDataName": "WuXia.Battle.SaveDoubleBuildData_97e1d999-b2a1-4b34-aee4-f7e9a6cf6c58"
          }
        ],
        "SaveDoubleBuildDatas": [
          {
            "Uuid": "97e1d999-b2a1-4b34-aee4-f7e9a6cf6c58",
            "Expression": "reasonAsAction != null && reasonAsAction.Weapon == \"轻剑\" ? 0.1 : 0"
          }
        ]
      }
    }
  ]
}
```

### 5.2 Accessories

```json
{
  "ObjectType": "WuXia.Items.Equipments.Accessory",
  "FullName": "Item.Equipment.Accessory.破锋珠",
  "Price": 1000,
  "Level": 2,
  "Effects": [
    {
      "Name": "破锋珠",
      "Description": "攻击力提升42",
      "Attribute": {
        "Values": [
          { "Name": "Attack", "Add": 42 }
        ]
      }
    }
  ]
}
```

### 5.3 Field Rules

- Weapon `ObjectType` is fixed as `WuXia.Items.Equipments.Weapon`
- Accessory `ObjectType` is fixed as `WuXia.Items.Equipments.Accessory`
- `FullName` is the Equipment primary key
- The weapon type is parsed from the second-to-last segment of `FullName`
- Each item in `Effects` is an `IEffect.Data`
- At runtime, the first Equipment effect is always loaded with `showable = false`

### 5.4 Text Keys

The Equipment UI reads:

- `Item.Equipment....`
- `Item.Equipment....Description`
- `Item.Equipment....Effect`

`.Effect` is used for the Equipment summary text.

## 6. Standalone Combat Effects

### 6.1 Basic Structure

```json
{
  "ObjectType": "WuXia.Battle.Effects.IEffect",
  "Name": "战意",
  "Description": "Effect.战意.Description",
  "Attribute": {
    "Values": [
      { "Name": "Attack", "Scale": 0.01 },
      { "Name": "Defense", "Scale": 0.01 },
      { "Name": "Speed", "Scale": 0.01 }
    ]
  },
  "Context": {
    "MaxLayer": 20,
    "ScaleAttributeWithLayer": true
  }
}
```

### 6.2 Field Rules

- `ObjectType` is fixed as `WuXia.Battle.Effects.IEffect`
- `Name` is the Effect name and also the body of the `Effect.EffectName` text key
- `Description` can be raw text or an `Effect.xxx.Description` key
- `Attribute`, `ApplyDamage`, and `ReceiveDamage` can be used alone or in combination
- `Context` decides stacking, Trackers, duration, and mutex grouping
- `Showable` controls the display toggle limit

### 6.3 How to Reference

Reference a standalone Combat Effect in `AddEffectBuildData` like this:

```json
{
  "Uuid": "2f3042d6-1428-42f4-b56f-c466f29d146c",
  "Target": "Source",
  "TurnOrLayer": 2,
  "FlexibleEffectNames": ["战意"]
}
```

### 6.4 Status Panel Text

The status panel reads:

- `Effect.EffectName`
- `Effect.EffectName.Description`

If `Description` itself starts with `Effect.`, the UI resolves it as a localization key.

## 7. Tracker Reference

Below are all concrete Trackers currently available in the current version.

### 7.1 Lifecycle and Action Events

| TrackerType | Trigger Time | Current Callback Object | Typical Use |
| --- | --- | --- | --- |
| `WuXia.Battle.IStartBattleTracker` | After battle start | `unit` | Grant opening effects, record state, summon |
| `WuXia.Battle.IGlobalUnitCountTracker` | After the number of units on the field changes | `owner` | Refresh effects based on total unit count |
| `WuXia.Battle.IStartTurnTracker` | After a unit's turn starts | `unit` | Start-turn recovery, buffs, layer loss |
| `WuXia.Battle.IEndTurnTracker` | After a unit's turn ends | `unit` | End-turn resolution and removal |
| `WuXia.Battle.IApplyActionStageTracker` | After a Skill stage executes | `source`, `targets`, `action`, `stage` | Insert logic into the five stages |
| `WuXia.Battle.IApplyActionTracker` | After the source finishes a Skill | `source`, `targets`, `action` | Continue attacks, Follow-Up, energy gain, Buffs |
| `WuXia.Battle.IReceiveActionTracker` | After the target receives a Skill | `source`, `target`, `action` | On-hit triggers, counters, receive markers |
| `WuXia.Battle.IGlobalActionTracker` | After one action finishes | `source`, `targets`, `action` | Global listeners for action results |

This group is suitable for:

- `ExpressionBuildData`
- `AddEffectBuildData`
- `RemoveEffectBuildData`
- `AddEffectLayerBuildData`
- `TriggerDotBuildData`
- `ApplyRecoveryBuildData`
- `AddSecretActionPowerBuildData`
- `ModifyUnitProgressBuildData`
- `SummonBuildData`
- `OnDamageBuildData`
- `UnitContinueAttackBuildData`
- `UnitInvokeFollowAttackBuildData`

### 7.2 Numeric Calculation Trackers

| TrackerType | Return Value Source | Meaning |
| --- | --- | --- |
| `WuXia.Battle.ICalculateCostTracker` | Sum of `SaveIntBuildData.Value` | Modify Skill cost |
| `WuXia.Battle.ICalculateDuanUpdateTracker` | Last written `SaveIntBuildData.Value` | Modify Step progression rule; the value is interpreted as `DuanUpdate` |
| `WuXia.Battle.IActionSelectTargetTracker` | Sum of `SaveDoubleBuildData.Value` | Modify AI target weight |
| `WuXia.Battle.ICalculateCriticalRateTracker` | Sum of `SaveDoubleBuildData.Value` | Modify critical rate |
| `WuXia.Battle.ICalculateCriticalScaleTracker` | Sum of `SaveDoubleBuildData.Value` | Modify critical damage multiplier |
| `WuXia.Battle.ICalculateApplyDamageTracker` | Sum of `SaveDoubleBuildData.Value` | Modify damage dealt multiplier |
| `WuXia.Battle.ICalculateReceiveDamageTracker` | Sum of `SaveDoubleBuildData.Value` | Modify damage received multiplier |
| `WuXia.Battle.IGlobalDamageTracker` | Sum of `SaveDoubleBuildData.Value` | Modify damage beyond self dealt/received cases |
| `WuXia.Battle.ICalculateApplyAdditionalDamageTracker` | Sum of `SaveDoubleBuildData.Value` | Additional damage multiplier |
| `WuXia.Battle.ICalculateContinueAttackDamageTracker` | Sum of `SaveDoubleBuildData.Value` | Continue attack damage |
| `WuXia.Battle.ICalculateApplyPostureDamageTracker` | Sum of `SaveIntBuildData.Value` | Poise Damage change |
| `WuXia.Battle.IModifyDamageTracker` | Sum of `SaveDoubleBuildData.Value` | Final damage modifier; `stage` is `ModifyDamageStage` |
| `WuXia.Battle.ICalculateApplyRecoveryTracker` | `SaveDoubleBuildData.Value` | Modify healing multiplier |
| `WuXia.Battle.ICalculateEffectMutexGroupGlobalCountLimitTracker` | Sum of `SaveIntBuildData.Value` | Modify global mutex group cap |
| `WuXia.Battle.ICalculateEffectMaxLayerTracker` | Sum of `SaveIntBuildData.Value` | Modify max effect layers |

Current `IModifyDamageTracker.ModifyDamageStage` values:

- `0`: `ApplySource`
- `10`: `ApplyTarget`
- `100`: `ReceiveSource`
- `110`: `ReceiveTarget`

In expressions for effect layer limits and mutex group caps, use `source`, `target`, and `reasonAsEffect` directly to limit the affected object. Example:

```json
"Expression": "source == target && reasonAsEffect != null && BattleManager.IsEffect(reasonAsEffect, \"战意\") ? 5 : 0"
```

### 7.3 Damage, Healing, Breakdown, and Kill Events

| TrackerType | Trigger Time | Current Callback Object | Typical Use |
| --- | --- | --- | --- |
| `WuXia.Battle.IApplyDamageTracker` | After dealing damage | `source`, `target`, `reason`, `damage`, `type` | Add effects on hit, gain energy, bonus damage |
| `WuXia.Battle.IApplyPostureDamageTracker` | After dealing Poise Damage | `source`, `target`, `reason` | Poise Damage triggers |
| `WuXia.Battle.IReceiveDamageTracker` | After receiving damage | `source`, `target`, `reason` | Gain energy on hit, prep counters, add layers on hit |
| `WuXia.Battle.IApplyRecoveryTracker` | After applying healing | `source`, `target`, `reason` | Extra logic after healing |
| `WuXia.Battle.IReceiveRecoveryTracker` | After receiving healing | `source`, `target`, `reason` | On-heal triggers |
| `WuXia.Battle.IApplyBreakdownTracker` | After causing Stagger | `source`, `targets`, `action` | Breakdown rewards and Follow-Up |
| `WuXia.Battle.IReceiveBreakdownTracker` | After entering Stagger | `unit` | Remove or add effects on Stagger |
| `WuXia.Battle.IGlobalBreakdownTracker` | After any unit enters Stagger | `source`, `target`, `action` | Global Stagger listener |
| `WuXia.Battle.IRecoveryFromBreakdownTracker` | After recovering from Stagger | `unit` | Reapply states after recovery |
| `WuXia.Battle.IApplyKillTracker` | After causing a kill | `source`, `target`, `reason` | Kill rewards |
| `WuXia.Battle.IReceiveKillTracker` | After being killed | `source`, `target`, `reason` | Death resolution |
| `WuXia.Battle.IGlobalKillTracker` | After any kill | `source`, `target`, `reason` | Global kill listener |

### 7.4 Secret Action Power and Effect Events

| TrackerType | Trigger Time | Current Callback Object | Typical Use |
| --- | --- | --- | --- |
| `WuXia.Battle.ISetSecretActionPowerTracker` | After Secret Action Power changes | `unit`, `increaseValue` | Trigger effects from Secret Action Power gain/loss |
| `WuXia.Battle.IAddEffectTracker` | After adding an Effect | `source`, `target`, `reasonAsEffect`, `layer` | Chain logic after Buff application |
| `WuXia.Battle.IRemoveEffectTracker` | After removing an Effect | `unit`, `reasonAsEffect`, `layer` | Trigger on removal |
| `WuXia.Battle.IRemoveSelfEffectTracker` | After self Effect removal | `unit`, `reasonAsEffect`, `layer` | Compensation logic after self-destruction |

## 8. BuildData Reference

### 8.1 Base Fields

All BuildData types share:

- `Uuid`: `string`, unique identifier; usually auto-generated, but explicit values help when other data needs to reference it
- `If`: `string`, enable condition expression; empty means always enabled
- `Target`: `string`, target selection expression; empty falls back to the BuildData's own `DefaultTarget`

### 8.2 Full Type Table

| Type | BuildDataName Prefix | Main Fields | Effect |
| --- | --- | --- | --- |
| `ApplyRecoveryBuildData` | `WuXia.Battle.ApplyRecoveryBuildData_` | `Base`, `Scale`, `Value`, `PostureValue`, `CostValue`, `IsExtra` | Apply healing, Poise recovery, Inner Energy recovery |
| `OnDamageBuildData` | `WuXia.Battle.OnDamageBuildData_` | `Damage`, `DamageScale`, `PostureDamage`, `Expand`, `Type`, `IsCritical` | Deal direct damage |
| `AddSecretActionPowerBuildData` | `WuXia.Battle.AddSecretActionPowerBuildData_` | `Value`, `Scale` | Add or reduce Secret Action Power |
| `AddEffectBuildData` | `WuXia.Battle.Effects.AddEffectBuildData_` | `EffectUuid`, `TurnOrLayer`, `AttributeEffects`, `Scale`, `TypeEffects`, `FlexibleEffects`, `FlexibleEffectNames`, `Showable`, `SkipTracker` | Add Effect |
| `RemoveEffectBuildData` | `WuXia.Battle.Effects.RemoveEffectBuildData_` | `TargetName`, `TargetNames`, `TargetCategory`, `TargetCategories`, `TargetUuid`, `Count`, `Layer` | Remove Effect or reduce layers |
| `AddEffectLayerBuildData` | `WuXia.Battle.Effects.AddEffectLayerBuildData_` | `TargetName`, `TargetNames`, `Layer`, `Multiple` | Add layers to an existing Effect |
| `TriggerDotBuildData` | `WuXia.Battle.Effects.TriggerDotBuildData_` | `DotNames`, `Layer` | Trigger damage over time |
| `ExpressionBuildData` | `WuXia.Battle.ExpressionBuildData_` | `ProcessPerTarget`, `Call`, `CallList`, `CallDict` | Run expression logic |
| `SaveIntBuildData` | `WuXia.Battle.SaveIntBuildData_` | `Expression`, `InitialValue` | Save an integer result |
| `SaveDoubleBuildData` | `WuXia.Battle.SaveDoubleBuildData_` | `Expression`, `InitialValue` | Save a floating-point result |
| `SummonBuildData` | `WuXia.Battle.SummonBuildData_` | `FullName`, `PositionPolicy`, `AttributePolicy`, `Count` | Summon units |
| `ModifyUnitProgressBuildData` | `WuXia.Battle.ModifyUnitProgressBuildData_` | `Scale` | Adjust action bar progress |
| `UnitContinueAttackBuildData` | `WuXia.Battle.UnitContinueAttackBuildData_` | `Count` | Add continue attacks |
| `UnitInvokeFollowAttackBuildData` | `WuXia.Battle.UnitInvokeFollowAttackBuildData_` | `InvokeRange`, `InvokeCount` | Trigger Follow-Up |

### 8.3 Current Rules by BuildData Type

#### ApplyRecoveryBuildData

```json
{
  "Uuid": "9d60e9eb-440d-4912-a510-cd36064a35c2",
  "Target": "Target",
  "Base": "Source",
  "Scale": 0.15,
  "Value": 30,
  "PostureValue": 0,
  "CostValue": 0,
  "IsExtra": false
}
```

- `Base` supports `Source`, `Target`, `Damage`, `Owner`, and `EffectSource`
- Final healing is `base HP * Scale + Value`

| Field | Type | Meaning |
| --- | --- | --- |
| `Base` | `string` | Healing base source; the current version supports `Source`, `Target`, `Damage`, `Owner`, and `EffectSource` |
| `Scale` | `double` | Multiplier on the selected base HP or damage value |
| `Value` | `int` | Flat healing added on top |
| `PostureValue` | `int` | Extra Poise recovered |
| `CostValue` | `int` | Extra Inner Energy recovered |
| `IsExtra` | `bool` | Whether this counts as extra healing |

#### OnDamageBuildData

```json
{
  "Uuid": "8de3af12-5a0a-4c3d-9e30-6fef6a7c8b80",
  "Target": "Target",
  "Damage": 12,
  "DamageScale": 0.05,
  "PostureDamage": 3,
  "Type": "Damage",
  "IsCritical": false
}
```

- `Type` is `string -> DamageType`

`DamageType` options:

| Value | Meaning |
| --- | --- |
| `Damage` | Normal damage. Uses the standard damage pipeline and is affected by defense and damage modifiers |
| `UseHP` | Removes HP directly, but always leaves at least `1` HP |
| `FixedDamage` | Fixed damage. Uses the true-damage path and skips the standard damage modifiers |
| `DotDamage` | Damage over time. Uses the true-damage path and is usually triggered by DoT effects |

| Field | Type | Meaning |
| --- | --- | --- |
| `Damage` | `int` | Flat damage value |
| `DamageScale` | `double` | Additional damage based on target current HP |
| `PostureDamage` | `int` | Poise Damage dealt at the same time |
| `Type` | `string -> DamageType` | Damage type string, parsed as the `DamageType` enum |
| `IsCritical` | `bool` | Whether this direct damage is treated as a critical hit |

#### AddSecretActionPowerBuildData

```json
{
  "Uuid": "8f26d7f7-17cf-4522-a90e-7d62274ef1c1",
  "Target": "Source",
  "Value": 20,
  "Scale": 0
}
```

- `Scale` is based on the target's required Secret Action Power
- `Value` adds a flat amount directly

| Field | Type | Meaning |
| --- | --- | --- |
| `Value` | `int` | Flat Secret Action Power added or reduced |
| `Scale` | `double` | Multiplier based on the target's `SecretActionPowerCost` |

#### AddEffectBuildData

```json
{
  "Uuid": "2f3042d6-1428-42f4-b56f-c466f29d146c",
  "Target": "Source",
  "EffectUuid": "eab08f35-c2ff-4a86-8ebd-6292ed13d0f7",
  "TurnOrLayer": 2,
  "Showable": true,
  "SkipTracker": false,
  "FlexibleEffectNames": ["战意"]
}
```

- `AttributeEffects` generates attribute-type effects
- `TypeEffects` constructs Effect instances from effect type names
- `FlexibleEffects` embeds effect data directly
- `FlexibleEffectNames` loads effect data by standalone effect name
- When `TurnOrLayer > 0`, the Effect enters a duration or stacking context

| Field | Type | Meaning |
| --- | --- | --- |
| `EffectUuid` | `string` | Unique ID written into the Effect worker or context so it can be removed precisely later |
| `TurnOrLayer` | `int` | Duration turns or initial layer count, depending on the target effect type |
| `AttributeEffects` | `List<AttributeEffectBuildData>` | Generate attribute-type effects directly |
| `Scale` | `double` | Scale parameter read by `TypeEffects` constructors |
| `TypeEffects` | `List<string>` | Construct effect instances from effect type names |
| `FlexibleEffects` | `List<IEffect.Data>` | Directly embedded flexible effect data |
| `FlexibleEffectNames` | `List<string>` | Load effect data asynchronously by standalone effect name |
| `Showable` | `bool` | Whether the new Effect is shown in the status bar |
| `SkipTracker` | `bool` | Whether to skip `IAddEffectTracker` and follow-up Trackers when adding the Effect |

Fields on each `AttributeEffectBuildData` item:

| Field | Type | Meaning |
| --- | --- | --- |
| `Name` | `string` | Attribute name |
| `Scale` | `double` | Attribute scale bonus |
| `Add` | `int` | Flat attribute bonus |

#### RemoveEffectBuildData

```json
{
  "Uuid": "c6eb5a0f-f2de-4f7a-b954-7d4e64e92f35",
  "Target": "Target",
  "TargetNames": ["战意"],
  "Count": 1,
  "Layer": 0
}
```

- Matches by `TargetName`, `TargetNames`, `TargetCategory`, `TargetCategories`, or `TargetUuid`
- When `Layer > 0`, it reduces layers on stackable effects
- When `Layer = 0`, it removes the whole Effect

`EffectCategory` options:

| Value | Meaning |
| --- | --- |
| `Buff` | Positive effects, such as pure attribute increases, damage up, or damage reduction |
| `Debuff` | Negative effects, such as pure attribute reduction or vulnerability |
| `DotHot` | Damage-over-time or healing-over-time effects |
| `Special` | Special effects that do not fit the above categories |

| Field | Type | Meaning |
| --- | --- | --- |
| `TargetName` | `string` | Match one effect name |
| `TargetNames` | `string[]` | Match multiple effect names |
| `TargetCategory` | `string -> EffectCategory` | Match one effect category, parsed as the `EffectCategory` enum |
| `TargetCategories` | `string[] -> EffectCategory[]` | Match multiple effect categories, each parsed as `EffectCategory` |
| `TargetUuid` | `string` | Exact match on the worker or context `Uuid` |
| `Count` | `int` | Maximum number of matched effects to process; `<= 0` means all |
| `Layer` | `int` | Layers to remove; when `> 0`, only reduces stackable effects, otherwise removes the whole Effect |

#### AddEffectLayerBuildData

```json
{
  "Uuid": "7a5d2cb9-9f1f-4978-b7d0-0e3bfcf5c435",
  "Target": "Source",
  "TargetName": "战意",
  "Layer": 2,
  "Multiple": 0
}
```

- `Layer` adds a fixed number of layers
- `Multiple` is the interface for multiplicative layer growth

| Field | Type | Meaning |
| --- | --- | --- |
| `TargetName` | `string` | Match one effect name |
| `TargetNames` | `string[]` | Match multiple effect names |
| `Layer` | `int` | Fixed layers to add |
| `Multiple` | `int` | Multiplicative layer parameter; when `Multiple > 0`, it takes priority over `Layer` |

#### TriggerDotBuildData

```json
{
  "Uuid": "48b88d09-cc08-469f-a4fe-b5535a0fceb4",
  "Target": "Target",
  "DotNames": ["流血", "中毒"],
  "Layer": 1
}
```

- If `DotNames` is empty, all damage-over-time effects on the target are triggered
- `Layer` is the number of consecutive triggers

| Field | Type | Meaning |
| --- | --- | --- |
| `DotNames` | `string[]` | Names of the damage-over-time effects to trigger; empty matches all DoTs |
| `Layer` | `int` | Number of consecutive triggers, default `1` |

#### ExpressionBuildData

```json
{
  "Uuid": "0204fe75-e7c4-438c-b2d0-be17f0931863",
  "Target": "Target",
  "ProcessPerTarget": false,
  "CallList": [
    "action.WithMark(\"MyMark\")",
    "source.AddMark(\"Ready\")"
  ]
}
```

- `Call` runs a single expression
- `CallList` runs multiple expressions in order
- `CallDict` runs expressions with branching conditions
- When `ProcessPerTarget = true`, `currentTarget` switches one by one

| Field | Type | Meaning |
| --- | --- | --- |
| `ProcessPerTarget` | `bool` | Whether to process selected targets one by one; when true, `currentTarget` is set per target |
| `Call` | `string` | One expression |
| `CallList` | `string[]` | Expressions run in order; later items can use `value` to reference the previous result |
| `CallDict` | `CallDictItem[]` | Expressions with conditions and named return values |

Fields on each `CallDictItem`:

| Field | Type | Meaning |
| --- | --- | --- |
| `If` | `string` | Branch condition expression; empty means `Call` always runs |
| `Call` | `string` | Main expression for the branch |
| `Else` | `string` | Expression to run when `If` is false |
| `Return` | `string` | Variable name used to write the current `Call` result into context for later branches |

#### SaveIntBuildData / SaveDoubleBuildData

```json
{
  "Uuid": "f13c3d1b-d5aa-4083-bbea-4820bd849ba9",
  "Expression": "reasonAsAction != null && reasonAsAction.IsFollowAttack ? 0.1 : 0",
  "InitialValue": 0
}
```

- `Expression` refreshes `Value` on every trigger
- `InitialValue` is the starting value
- These two BuildData types are used by numeric Trackers and can also serve as intermediate values for other expressions

`SaveIntBuildData` fields:

| Field | Type | Meaning |
| --- | --- | --- |
| `Expression` | `string` | Expression that computes an integer result |
| `InitialValue` | `int` | Initial integer value returned before the expression runs |

`SaveDoubleBuildData` fields:

| Field | Type | Meaning |
| --- | --- | --- |
| `Expression` | `string` | Expression that computes a floating-point result |
| `InitialValue` | `double` | Initial floating-point value returned before the expression runs |

#### SummonBuildData

```json
{
  "Uuid": "df822799-b7f0-45e9-b90a-b3233b16f11d",
  "FullName": "Character.傀儡护卫",
  "PositionPolicy": "Random",
  "AttributePolicy": "Original",
  "Count": 2
}
```

- The current implementation always summons at least 1 unit; when `Count > 1`, it keeps summoning until `Count` is reached

`PositionPolicy` options:

| Value | Meaning |
| --- | --- |
| `Random` | Summon into a random empty tile in the valid summon area on your side |
| `AsSub` | Summon as a sub-unit using the current `source` position |
| Other values or empty | Fall back to the default branch, with position `(0, 0)` |

`AttributePolicy` options:

| Value | Meaning |
| --- | --- |
| `Original` | Keep the summoned character's original attributes; this is the default in `SummonContext` |
| `ReplaceBySource` | Only works when `PositionPolicy = "AsSub"`; replaces the summon's battle attributes with those of `source` |
| `ReplaceBySource;属性名=倍率` | Based on `ReplaceBySource`, applies extra multipliers to specified battle attributes, for example `ReplaceBySource;Attack=0.5;HP=2` |

Notes:

- `AttributePolicy` only enters the replacement logic when `PositionPolicy = "AsSub"`.
- `ReplaceBySource` currently only overrides battle attributes and does not copy scholarly or martial attributes.

| Field | Type | Meaning |
| --- | --- | --- |
| `FullName` | `string` | Full character name of the summoned unit |
| `PositionPolicy` | `string` | Summon position policy string; currently supports `Random` and `AsSub` |
| `AttributePolicy` | `string` | Summon attribute inheritance policy string; currently supports `Original`, `ReplaceBySource`, and the multiplier form |
| `Count` | `int` | Desired summon count; the current implementation always summons at least 1, and continues when `Count > 1` |

#### ModifyUnitProgressBuildData

```json
{
  "Uuid": "18e8b881-6b4d-47cc-8d84-0f5aa5f5315f",
  "Target": "Source",
  "Scale": 0.5
}
```

- The current version adjusts action bar progress by `target ProgressToTurn * Scale`

| Field | Type | Meaning |
| --- | --- | --- |
| `Scale` | `double` | Multiplier applied to the target's `ProgressToTurn` |

#### UnitContinueAttackBuildData

```json
{
  "Uuid": "4aa3d705-f72e-47e5-95c5-e98b7da2fd13",
  "Target": "Target",
  "Count": 1
}
```

- Adds continue attacks to targets that were already hit
- Continue attack damage still stacks with `ICalculateContinueAttackDamageTracker`

| Field | Type | Meaning |
| --- | --- | --- |
| `Count` | `int` | Base number of extra continue attacks; runtime also adds layers from `ExtraContinueAttack` |

#### UnitInvokeFollowAttackBuildData

```json
{
  "Uuid": "e7342783-2c6c-460b-a1a6-d1c737dac794",
  "Target": "Target",
  "InvokeRange": "BackColumn",
  "InvokeCount": 1
}
```

- Triggers a Follow-Up call on the first selected target
- A Follow-Up inside a Follow-Up will not recurse again

`InvokeRange` options:

| Value | Meaning |
| --- | --- |
| `FrontColumn` | Only search follow-up units in the front row on your side; excludes the current `source` |
| `BackColumn` | Only search follow-up units in the back row on your side; excludes the current `source` |
| `Owner` | Only search in the current Tracker `Owner` |
| `EffectSource` | Only search in the current Tracker `EffectSource` |
| Other values or empty | Fall back to the default branch: search all living allied units except the current `source` |

| Field | Type | Meaning |
| --- | --- | --- |
| `InvokeRange` | `string` | Follow-Up search range string; currently supports `FrontColumn`, `BackColumn`, `Owner`, and `EffectSource` |
| `InvokeCount` | `int` | Number of Follow-Ups to trigger, default `1` |

## 9. Current Author Workflow

1. Determine the `FullName` for the content.
2. Register it in `ConfigMaps` inside `manifest.json`.
3. Add localization keys for the display text.
4. Put complex mechanics for Cultivation Methods, Equipment, and standalone Effects into `IEffect.Data.Context`.
5. Put direct Skill flow into the five stage arrays.
6. Use `SaveIntBuildData` or `SaveDoubleBuildData` for numeric Trackers.
7. Use node-generating BuildData for event-style Trackers.
8. Extract reusable Buffs, Debuffs, and DoTs into standalone Combat Effects, then reference them through `FlexibleEffectNames`.