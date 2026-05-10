# 心法、招式、装备与战斗效果 Mod 官方指南

本指南面向战斗内容 Mod 作者，覆盖当前版本支持的四类内容：

- 心法
- 招式
- 装备
- 独立战斗效果

同时包含这四类内容共用的：

- manifest.json 写法
- 本地化与图标键
- 表达式上下文
- 全部 Tracker 与 BuildData 参考

## 1. manifest.json

Mod 通过根目录下的 `manifest.json` 加载。本指南涉及的条目只有三类：

- `ConfigMaps`：注册配置文件
- `LocalizationMaps`：注册本地化文件
- `SpriteMaps`：注册图标资源

示例：

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

规则：

- `FullName` 是运行时查找键，必须与内容类型规则一致。
- `Path` 是相对 Mod 根目录的路径。
- 心法、招式、装备通过 `ConfigMaps` 直接注册。
- 独立战斗效果通过 `ConfigMaps` 注册为 `Battle/Effect/效果名`。

## 2. 共享规则

### 2.1 FullName

四类内容使用以下标识规则：

- 心法：`Xinfa.Player.追锋诀`
- 角色心法：`Xinfa.角色名.心法名`
- 招式：`Battle.Action.武器类型.招式名`
- 武器：`Item.Equipment.Weapon.武器类型.装备名`
- 饰品：`Item.Equipment.Accessory.装备名`
- 独立战斗效果注册键：`Battle/Effect/效果名`

独立战斗效果在其他配置里引用时使用效果名本身，不写 `Battle/Effect/` 前缀。例如：

```json
"FlexibleEffectNames": ["战意"]
```

### 2.2 本地化

本地化文件使用 `Tables` 数组。当前版本战斗内容对应四张表：

- `Xinfa`
- `BattleAction`
- `Item`
- `Effect`

示例：

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

文本键规则：

- 心法名称：`Xinfa...`
- 心法描述：`Xinfa....Description`
- 心法等级效果：`Xinfa....Effect.0`、`Effect.1`、`Effect.2`
- 招式名称：`Battle.Action...`
- 招式说明：`Battle.Action....Description`

- 装备名称：`Item.Equipment...`
- 装备描述：`Item.Equipment....Description`
- 装备摘要：`Item.Equipment....Effect`
- 效果名称：`Effect.效果名`
- 效果描述：`Effect.效果名.Description`

### 2.3 图标键

图标键固定为：

- `${fullName}.Icon.Sprite`

这一规则适用于：

- 心法
- 招式
- 装备

独立战斗效果不使用这一套图标键。

### 2.4 表达式上下文

`If`、`Expression`、`Call`、`CallList`、`CallDict` 使用同一套战斗表达式上下文。当前版本可直接使用的变量有：

- `unit`：当前单位型回调的单位
- `source`：事件发起方
- `action`：当前招式
- `targets`：目标列表
- `currentTarget`：逐目标处理时的当前目标
- `reason`：原因对象
- `reasonAsAction`：原因被解释为招式时的对象
- `reasonAsEffect`：原因被解释为效果时的对象
- `target`：单目标对象
- `effect`：当前效果
- `owner`：效果拥有者
- `effectSource`：效果来源者
- `context`：当前柔性效果上下文
- `damage`：当前伤害值
- `stage`：阶段值
- `increaseValue`：绝式能量改变量
- `layer`：层数
- `type`：伤害类型字符串


### 2.5 BuildData 共通规则

所有 BuildData 都有三项基础字段：

- `Uuid`
- `If`
- `Target`

`BuildDataName` 固定写法：

```json
"BuildDataName": "完整类型名_UUID"
```

例如：

```json
"BuildDataName": "WuXia.Battle.SaveDoubleBuildData_f13c3d1b-d5aa-4083-bbea-4820bd849ba9"
```

`Target` 的基础目标选择：

- `Source`
- `Target`
- `SourceTeam`
- `TargetTeam`
- `Owner`
- `EffectSource`
- `Named`

`Target` 可以继续拼接过滤器，用 `;` 分隔：

- `Self`
- `Opponent`
- `LowestHP`
- `predicate:表达式`

示例：

```json
"Target": "TargetTeam;LowestHP"
```

```json
"Target": "Target;predicate:predicateTarget.HasEffect(\"战意\")"
```

### 2.6 柔性效果 Context

心法效果、装备效果、独立战斗效果都使用 `IEffect.Data`。其 `Context` 字段为：

```json
{
  "MutexGroup": "示例互斥组",
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

规则：

- `TypedBuildDatas` 决定 BuildData 绑定到哪个 Tracker。
- `Priority` 越大，Tracker 执行优先级越高。
- `MaxLayer > 0` 时，效果进入叠层上下文。
- `ScaleAttributeWithLayer = true` 时，属性型效果按层数缩放，且必须与 `MaxLayer` 一起使用。
- `MutexGroup` 与 `MutexGroupGlobalCountLimit` 用于互斥组总量控制。

## 3. 心法

### 3.1 基本结构

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

### 3.2 字段规则

- `ObjectType` 固定为 `WuXia.Attributes.Xinfa`
- `Name` 是心法名，不带前缀
- `Tier` 是品阶
- `Slot` 是占用槽数
- `FlexibleLevelEffects` 按顺序对应 1 级、2 级、3 级……
- `Attributes` 用于自动生成升级阈值
- `LevelAttributeThresholds` 用于手动指定升级阈值

### 3.3 升级阈值

当 `LevelAttributeThresholds` 为空且 `Attributes` 非空时，当前版本按游戏规则表自动生成阈值：

- `Tier 1`：`[75]`
- `Tier 2`：`[100, 200]`
- `Tier 3`：`[200, 250, 300]`

手动阈值写法：

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

对于没有成长属性数据的单位，心法当前生效等级直接等于 `FlexibleLevelEffects` 的数量。

### 3.4 文本键

心法界面读取：

- `Xinfa.拥有者.心法名`
- `Xinfa.拥有者.心法名.Description`
- `Xinfa.拥有者.心法名.Effect.0`
- `Xinfa.拥有者.心法名.Effect.1`

## 4. 招式

### 4.1 基本结构

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

### 4.2 字段规则

- `ObjectType` 固定为 `WuXia.Battle.Actions.FlexibleAction`
- `Name` 是招式名
- `Weapon` 是武器类型
- `Cost` 是内力消耗
- `SecretActionPowerCost > 0` 时，该招式属于绝式
- `IsCounterAttack = true` 时，该招式属于反击
- `IsFollowAttack = true` 时，该招式属于追击
- 普通主动招式使用 `Duan`
- `DuanUpdate` 支持 `Keep`、`Increase`、`Decrease`、`Final`
- `AttackScale` 是伤害倍率基值
- `PostureDamage` 是削韧值

### 4.3 目标范围

`Range` 是 `string -> ActionSelectionRange`。

`ActionSelectionRange` 可选值：

| 值 | 说明 |
| --- | --- |
| `Self` | 以自己为目标 |
| `Single` | 单体目标 |
| `Row` | 与当前格同一横排的目标 |
| `Column` | 与当前格同一纵列的目标 |
| `All` | 当前可选阵营的全部目标 |
| `Front` | 当前格对应的前排位置 |
| `SelfLowestHP` | 当前可选阵营中生命值最低的单位 |
| `Expression` | 用表达式返回一个 `ActionSelectionRange` 枚举值 |


### 4.4 阶段

招式固定有五个阶段，按顺序执行：

1. `BeforeStart`
2. `AfterStart`
3. `In`
4. `BeforeEnd`
5. `AfterEnd`

对应字段：

- `BeforeStartBuildDataNames`
- `AfterStartBuildDataNames`
- `InBuildDataNames`
- `BeforeEndBuildDataNames`
- `AfterEndBuildDataNames`

### 4.5 招式支持的 BuildData 数组

当前版本招式配置支持以下数组：

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

写法规则：

- 阶段数组里写 `BuildDataName`
- 对应实体写在同类型数组里
- `Uuid` 保持唯一

### 4.6 文本键

招式界面读取：

- `Battle.Action.武器类型.招式名`
- `Battle.Action.武器类型.招式名.Description`

当前版本中：

- `.Description` 写额外效果、触发条件、附加状态
- 范围、倍率、段位等基础信息由界面根据招式数据生成

## 5. 装备

### 5.1 武器

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

### 5.2 饰品

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

### 5.3 字段规则

- 武器 `ObjectType` 固定为 `WuXia.Items.Equipments.Weapon`
- 饰品 `ObjectType` 固定为 `WuXia.Items.Equipments.Accessory`
- `FullName` 是装备主键
- 武器类型从 `FullName` 的倒数第二段解析
- `Effects` 数组中的每一项都是 `IEffect.Data`
- 装备的第 1 项效果在运行时始终以 `showable = false` 载入

### 5.4 文本键

装备界面读取：

- `Item.Equipment....`
- `Item.Equipment....Description`
- `Item.Equipment....Effect`

`.Effect` 用于装备摘要文本。

## 6. 独立战斗效果

### 6.1 基本结构

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

### 6.2 字段规则

- `ObjectType` 固定为 `WuXia.Battle.Effects.IEffect`
- `Name` 是效果名，也是 `Effect.效果名` 文本键的主体
- `Description` 可以直接写描述，或写成 `Effect.xxx.Description` 键
- `Attribute`、`ApplyDamage`、`ReceiveDamage` 三选一或组合使用
- `Context` 决定效果是否带 Tracker、叠层、持续时间、互斥组
- `Showable` 是显示开关上限

### 6.3 引用方式

在 `AddEffectBuildData` 中引用独立效果：

```json
{
  "Uuid": "2f3042d6-1428-42f4-b56f-c466f29d146c",
  "Target": "Source",
  "TurnOrLayer": 2,
  "FlexibleEffectNames": ["战意"]
}
```

### 6.4 状态面板文本

状态面板读取：

- `Effect.效果名`
- `Effect.效果名.Description`

如果 `Description` 本身以 `Effect.` 开头，界面会将其当作本地化键解析。

## 7. Tracker 参考

下面列出当前版本全部可用具体 Tracker。

### 7.1 生命周期与行动事件

| TrackerType | 触发时机 | 当前回调对象 | 典型用途 |
| --- | --- | --- | --- |
| `WuXia.Battle.IStartBattleTracker` | 战斗开始后 | `unit` | 开局赋予效果、记录状态、召唤 |
| `WuXia.Battle.IGlobalUnitCountTracker` | 场上单位数量变化后 | `owner` | 根据场上人数刷新效果 |
| `WuXia.Battle.IStartTurnTracker` | 单位回合开始后 | `unit` | 开始回合触发恢复、增益、扣层 |
| `WuXia.Battle.IEndTurnTracker` | 单位回合结束后 | `unit` | 结束回合触发结算、移除 |
| `WuXia.Battle.IApplyActionStageTracker` | 招式阶段执行后 | `source`、`targets`、`action`、`stage` | 按五段阶段插入逻辑 |
| `WuXia.Battle.IApplyActionTracker` | 发起方完成招式后 | `source`、`targets`、`action` | 连击、追击、回能、加 Buff |
| `WuXia.Battle.IReceiveActionTracker` | 目标接收招式后 | `source`、`target`、`action` | 受击触发、反制、受招标记 |
| `WuXia.Battle.IGlobalActionTracker` | 一次行动完成后 | `source`、`targets`、`action` | 全局监听行动结果 |

这组 Tracker 适合挂：

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

### 7.2 数值计算 Tracker

| TrackerType | 返回值来源 | 说明 |
| --- | --- | --- |
| `WuXia.Battle.ICalculateCostTracker` | `SaveIntBuildData.Value` 求和 | 修改招式消耗 |
| `WuXia.Battle.ICalculateDuanUpdateTracker` | `SaveIntBuildData.Value` 最后写入 | 修改段位推进规则，值按 `DuanUpdate` 枚举解释 |
| `WuXia.Battle.IActionSelectTargetTracker` | `SaveDoubleBuildData.Value` 求和 | 修改 AI 选目标权重 |
| `WuXia.Battle.ICalculateCriticalRateTracker` | `SaveDoubleBuildData.Value` 求和 | 修改暴击率 |
| `WuXia.Battle.ICalculateCriticalScaleTracker` | `SaveDoubleBuildData.Value` 求和 | 修改暴击伤害倍率 |
| `WuXia.Battle.ICalculateApplyDamageTracker` | `SaveDoubleBuildData.Value` 求和 | 修改造成伤害倍率 |
| `WuXia.Battle.ICalculateReceiveDamageTracker` | `SaveDoubleBuildData.Value` 求和 | 修改承受伤害倍率 |
| `WuXia.Battle.IGlobalDamageTracker` | `SaveDoubleBuildData.Value` 求和 | 修改自身造成和承受以外情况的伤害倍率 |
| `WuXia.Battle.ICalculateApplyAdditionalDamageTracker` | `SaveDoubleBuildData.Value` 求和 | 附加伤害倍率 |
| `WuXia.Battle.ICalculateContinueAttackDamageTracker` | `SaveDoubleBuildData.Value` 求和 | 连击伤害 |
| `WuXia.Battle.ICalculateApplyPostureDamageTracker` | `SaveIntBuildData.Value` 求和 | 削韧值增减 |
| `WuXia.Battle.IModifyDamageTracker` | `SaveDoubleBuildData.Value` 求和 | 最终伤害修正，`stage` 为 `ModifyDamageStage` |
| `WuXia.Battle.ICalculateApplyRecoveryTracker` | `SaveDoubleBuildData.Value` | 修改治疗倍率 |
| `WuXia.Battle.ICalculateEffectMutexGroupGlobalCountLimitTracker` | `SaveIntBuildData.Value` 求和 | 修改效果互斥组全局上限 |
| `WuXia.Battle.ICalculateEffectMaxLayerTracker` | `SaveIntBuildData.Value` 求和 | 修改效果层数上限 |

`IModifyDamageTracker.ModifyDamageStage` 当前值：

- `0`：`ApplySource`
- `10`：`ApplyTarget`
- `100`：`ReceiveSource`
- `110`：`ReceiveTarget`


效果层数与互斥组上限的表达式里直接使用 `source`、`target`、`reasonAsEffect` 约束作用对象。例如：

```json
"Expression": "source == target && reasonAsEffect != null && BattleManager.IsEffect(reasonAsEffect, \"战意\") ? 5 : 0"
```

### 7.3 伤害、治疗、击破、击杀事件

| TrackerType | 触发时机 | 当前回调对象 | 典型用途 |
| --- | --- | --- | --- |
| `WuXia.Battle.IApplyDamageTracker` | 造成伤害后 | `source`、`target`、`reason`、`damage`、`type` | 命中后追加效果、回能、追伤 |
| `WuXia.Battle.IApplyPostureDamageTracker` | 造成削韧后 | `source`、`target`、`reason` | 削韧触发 |
| `WuXia.Battle.IReceiveDamageTracker` | 受到伤害后 | `source`、`target`、`reason` | 受伤回能、反击准备、受击加层 |
| `WuXia.Battle.IApplyRecoveryTracker` | 造成治疗后 | `source`、`target`、`reason` | 治疗后追加逻辑 |
| `WuXia.Battle.IReceiveRecoveryTracker` | 受到治疗后 | `source`、`target`、`reason` | 受疗触发 |
| `WuXia.Battle.IApplyBreakdownTracker` | 造成崩势后 | `source`、`targets`、`action` | 击破奖励、追击 |
| `WuXia.Battle.IReceiveBreakdownTracker` | 自身进入崩势后 | `unit` | 崩势时移除或追加效果 |
| `WuXia.Battle.IGlobalBreakdownTracker` | 任意单位崩势后 | `source`、`target`、`action` | 全局监听崩势 |
| `WuXia.Battle.IRecoveryFromBreakdownTracker` | 从崩势恢复后 | `unit` | 恢复后重新加状态 |
| `WuXia.Battle.IApplyKillTracker` | 造成击杀后 | `source`、`target`、`reason` | 击杀奖励 |
| `WuXia.Battle.IReceiveKillTracker` | 被击杀后 | `source`、`target`、`reason` | 死亡时结算 |
| `WuXia.Battle.IGlobalKillTracker` | 任意击杀后 | `source`、`target`、`reason` | 全局击杀监听 |

### 7.4 绝式能量与效果事件

| TrackerType | 触发时机 | 当前回调对象 | 典型用途 |
| --- | --- | --- | --- |
| `WuXia.Battle.ISetSecretActionPowerTracker` | 绝式能量变化后 | `unit`、`increaseValue` | 根据绝式能量增减触发效果 |
| `WuXia.Battle.IAddEffectTracker` | 添加效果后 | `source`、`target`、`reasonAsEffect`、`layer` | 加 Buff 后连锁处理 |
| `WuXia.Battle.IRemoveEffectTracker` | 移除效果后 | `unit`、`reasonAsEffect`、`layer` | 被移除时触发 |
| `WuXia.Battle.IRemoveSelfEffectTracker` | 自身效果移除后 | `unit`、`reasonAsEffect`、`layer` | 自我销毁后的补偿逻辑 |

## 8. BuildData 参考

### 8.1 基础字段

所有 BuildData 共享：

- `Uuid`：`string`，唯一标识；默认会自动生成，显式填写时便于互相引用
- `If`：`string`，启用条件表达式；为空时始终启用
- `Target`：`string`，目标选择表达式；为空时回落到各自 BuildData 的 `DefaultTarget`

### 8.2 类型总表

| 类型 | BuildDataName 前缀 | 主要字段 | 作用 |
| --- | --- | --- | --- |
| `ApplyRecoveryBuildData` | `WuXia.Battle.ApplyRecoveryBuildData_` | `Base`、`Scale`、`Value`、`PostureValue`、`CostValue`、`IsExtra` | 施加治疗、韧性恢复、回内 |
| `OnDamageBuildData` | `WuXia.Battle.OnDamageBuildData_` | `Damage`、`DamageScale`、`PostureDamage`、`Expand`、`Type`、`IsCritical` | 直接造成伤害 |
| `AddSecretActionPowerBuildData` | `WuXia.Battle.AddSecretActionPowerBuildData_` | `Value`、`Scale` | 增减绝式能量 |
| `AddEffectBuildData` | `WuXia.Battle.Effects.AddEffectBuildData_` | `EffectUuid`、`TurnOrLayer`、`AttributeEffects`、`Scale`、`TypeEffects`、`FlexibleEffects`、`FlexibleEffectNames`、`Showable`、`SkipTracker` | 添加效果 |
| `RemoveEffectBuildData` | `WuXia.Battle.Effects.RemoveEffectBuildData_` | `TargetName`、`TargetNames`、`TargetCategory`、`TargetCategories`、`TargetUuid`、`Count`、`Layer` | 移除效果或扣层 |
| `AddEffectLayerBuildData` | `WuXia.Battle.Effects.AddEffectLayerBuildData_` | `TargetName`、`TargetNames`、`Layer`、`Multiple` | 为已有效果加层 |
| `TriggerDotBuildData` | `WuXia.Battle.Effects.TriggerDotBuildData_` | `DotNames`、`Layer` | 触发持续伤害 |
| `ExpressionBuildData` | `WuXia.Battle.ExpressionBuildData_` | `ProcessPerTarget`、`Call`、`CallList`、`CallDict` | 执行表达式逻辑 |
| `SaveIntBuildData` | `WuXia.Battle.SaveIntBuildData_` | `Expression`、`InitialValue` | 保存整数结果 |
| `SaveDoubleBuildData` | `WuXia.Battle.SaveDoubleBuildData_` | `Expression`、`InitialValue` | 保存浮点结果 |
| `SummonBuildData` | `WuXia.Battle.SummonBuildData_` | `FullName`、`PositionPolicy`、`AttributePolicy`、`Count` | 召唤单位 |
| `ModifyUnitProgressBuildData` | `WuXia.Battle.ModifyUnitProgressBuildData_` | `Scale` | 调整行动条 |
| `UnitContinueAttackBuildData` | `WuXia.Battle.UnitContinueAttackBuildData_` | `Count` | 追加连击 |
| `UnitInvokeFollowAttackBuildData` | `WuXia.Battle.UnitInvokeFollowAttackBuildData_` | `InvokeRange`、`InvokeCount` | 触发追击 |

### 8.3 各 BuildData 的当前规则

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

- `Base` 支持 `Source`、`Target`、`Damage`、`Owner`、`EffectSource`
- 最终治疗值为 `基准 HP * Scale + Value`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `Base` | `string` | 治疗基准来源；当前版本识别 `Source`、`Target`、`Damage`、`Owner`、`EffectSource` |
| `Scale` | `double` | 基准倍率；与 `Base` 对应的 HP 或伤害值相乘 |
| `Value` | `int` | 固定治疗附加值 |
| `PostureValue` | `int` | 额外恢复的韧性值 |
| `CostValue` | `int` | 额外恢复的内力值 |
| `IsExtra` | `bool` | 是否按额外治疗处理 |

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

- `Type` 是 `string -> DamageType`

`DamageType` 可选值：

| 值 | 说明 |
| --- | --- |
| `Damage` | 普通伤害，进入常规伤害结算，会受防御与伤害修正影响 |
| `UseHP` | 扣除生命值，但会至少保留 `1` 点生命 |
| `FixedDamage` | 固定伤害，按真伤路径结算，不走常规伤害修正 |
| `DotDamage` | 持续伤害，按真伤路径结算，通常由 DoT 效果触发 |

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `Damage` | `int` | 固定伤害值 |
| `DamageScale` | `double` | 按目标当前 HP 计算的附加伤害倍率 |
| `PostureDamage` | `int` | 同时造成的削韧值 |
| `Type` | `string -> DamageType` | 伤害类型字符串，按 `DamageType` 枚举解析 |
| `IsCritical` | `bool` | 这次直接伤害是否按暴击处理 |

#### AddSecretActionPowerBuildData

```json
{
  "Uuid": "8f26d7f7-17cf-4522-a90e-7d62274ef1c1",
  "Target": "Source",
  "Value": 20,
  "Scale": 0
}
```

- `Scale` 按目标绝式需求值计算
- `Value` 直接加固定值

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `Value` | `int` | 固定增减的绝式能量值 |
| `Scale` | `double` | 按目标 `SecretActionPowerCost` 计算的倍率 |

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

- `AttributeEffects` 生成属性型效果
- `TypeEffects` 按效果类型名构造效果对象
- `FlexibleEffects` 直接内嵌效果数据
- `FlexibleEffectNames` 通过独立效果名加载效果
- `TurnOrLayer > 0` 时，效果进入持续或叠层上下文

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `EffectUuid` | `string` | 写入效果 Worker 或 Context 的唯一标识，便于后续精确移除 |
| `TurnOrLayer` | `int` | 持续回合数或初始层数；具体由目标效果类型解释 |
| `AttributeEffects` | `List<AttributeEffectBuildData>` | 直接生成属性型效果 |
| `Scale` | `double` | 供 `TypeEffects` 构造器读取的倍率参数 |
| `TypeEffects` | `List<string>` | 通过效果类型名构造效果实例 |
| `FlexibleEffects` | `List<IEffect.Data>` | 直接内嵌的柔性效果数据 |
| `FlexibleEffectNames` | `List<string>` | 通过独立效果名异步加载效果数据 |
| `Showable` | `bool` | 新增效果是否显示在状态栏 |
| `SkipTracker` | `bool` | 添加效果时是否跳过 `IAddEffectTracker` 等后续 Tracker |

`AttributeEffectBuildData` 子项字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `Name` | `string` | 属性名 |
| `Scale` | `double` | 属性倍率加成 |
| `Add` | `int` | 属性固定加成 |

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

- 按 `TargetName`、`TargetNames`、`TargetCategory`、`TargetCategories`、`TargetUuid` 匹配
- `Layer > 0` 时，对可叠层效果执行扣层
- `Layer = 0` 时，移除整个效果

`EffectCategory` 可选值：

| 值 | 说明 |
| --- | --- |
| `Buff` | 正向效果；例如纯正收益的属性提升、增伤、减伤 |
| `Debuff` | 负向效果；例如纯负收益的属性下降、易伤 |
| `DotHot` | 持续伤害或持续恢复类效果 |
| `Special` | 无法归入以上三类的特殊效果 |

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `TargetName` | `string` | 单个效果名匹配 |
| `TargetNames` | `string[]` | 多个效果名匹配 |
| `TargetCategory` | `string -> EffectCategory` | 单个效果分类匹配，按 `EffectCategory` 枚举解析 |
| `TargetCategories` | `string[] -> EffectCategory[]` | 多个效果分类匹配，逐项按 `EffectCategory` 枚举解析 |
| `TargetUuid` | `string` | 按效果 Worker 或 Context 的 `Uuid` 精确匹配 |
| `Count` | `int` | 最多处理多少个匹配到的效果；`<= 0` 时视为全部 |
| `Layer` | `int` | 要扣除的层数；`> 0` 时仅对叠层效果扣层，否则移除整个效果 |

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

- `Layer` 为固定加层
- `Multiple` 为倍增式加层接口

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `TargetName` | `string` | 单个效果名匹配 |
| `TargetNames` | `string[]` | 多个效果名匹配 |
| `Layer` | `int` | 固定增加的层数 |
| `Multiple` | `int` | 倍增式加层参数；`Multiple > 0` 时优先于 `Layer` 生效 |

#### TriggerDotBuildData

```json
{
  "Uuid": "48b88d09-cc08-469f-a4fe-b5535a0fceb4",
  "Target": "Target",
  "DotNames": ["流血", "中毒"],
  "Layer": 1
}
```

- `DotNames` 为空时触发目标上的全部持续伤害效果
- `Layer` 表示连续触发次数

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `DotNames` | `string[]` | 要触发的持续伤害效果名列表；为空时匹配全部 DoT |
| `Layer` | `int` | 连续触发次数，默认值为 `1` |

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

- `Call` 执行单条表达式
- `CallList` 按顺序执行多条表达式
- `CallDict` 用条件分支执行表达式
- `ProcessPerTarget = true` 时，`currentTarget` 逐个切换

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `ProcessPerTarget` | `bool` | 是否按选中目标逐个执行；为 `true` 时会逐个设置 `currentTarget` |
| `Call` | `string` | 单条表达式 |
| `CallList` | `string[]` | 顺序执行的表达式列表；后续项可用 `value` 引用上一项结果 |
| `CallDict` | `CallDictItem[]` | 带条件与别名返回值的表达式列表 |

`CallDictItem` 子项字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `If` | `string` | 分支条件表达式；为空时无条件执行 `Call` |
| `Call` | `string` | 该分支的主表达式 |
| `Else` | `string` | `If` 不成立时执行的表达式 |
| `Return` | `string` | 将当前 `Call` 的结果写入上下文时使用的变量名，供后续分支继续引用 |

#### SaveIntBuildData / SaveDoubleBuildData

```json
{
  "Uuid": "f13c3d1b-d5aa-4083-bbea-4820bd849ba9",
  "Expression": "reasonAsAction != null && reasonAsAction.IsFollowAttack ? 0.1 : 0",
  "InitialValue": 0
}
```

- `Expression` 每次触发时刷新 `Value`
- `InitialValue` 是初始值
- 这两类 BuildData 用于数值型 Tracker，也可作为其他表达式引用的中间变量

`SaveIntBuildData` 字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `Expression` | `string` | 计算整数结果的表达式 |
| `InitialValue` | `int` | 初始整数值；未执行表达式时返回该值 |

`SaveDoubleBuildData` 字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `Expression` | `string` | 计算浮点结果的表达式 |
| `InitialValue` | `double` | 初始浮点值；未执行表达式时返回该值 |

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

- 当前实现至少召唤 1 个单位；`Count > 1` 时补足到 `Count` 个

`PositionPolicy` 可选值：

| 值 | 说明 |
| --- | --- |
| `Random` | 在己方可用召唤区域内随机选择一个空位召唤 |
| `AsSub` | 以当前 `source` 所在位置作为附属单位召唤 |
| 其他值或留空 | 走默认分支，位置回落到 `(0, 0)` |

`AttributePolicy` 可选值：

| 值 | 说明 |
| --- | --- |
| `Original` | 保持被召唤角色原始属性；这是 `SummonContext` 的默认值 |
| `ReplaceBySource` | 仅在 `AsSub` 分支下生效；将召唤物的战斗属性替换为 `source` 的对应属性 |
| `ReplaceBySource;属性名=倍率` | 在 `ReplaceBySource` 基础上，为指定战斗属性附加倍率，例如 `ReplaceBySource;Attack=0.5;HP=2` |

注意事项：

- `AttributePolicy` 只有在 `PositionPolicy = "AsSub"` 时才会进入属性替换逻辑。
- `ReplaceBySource` 当前只会覆盖战斗属性，不会复制文系、武系属性。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `FullName` | `string` | 被召唤单位的角色全名 |
| `PositionPolicy` | `string` | 召唤站位策略字符串；当前支持 `Random`、`AsSub` |
| `AttributePolicy` | `string` | 召唤属性继承策略字符串；当前支持 `Original`、`ReplaceBySource` 及其带倍率参数的写法 |
| `Count` | `int` | 期望召唤数量；当前实现至少召唤 1 个，`Count > 1` 时继续追加 |

#### ModifyUnitProgressBuildData

```json
{
  "Uuid": "18e8b881-6b4d-47cc-8d84-0f5aa5f5315f",
  "Target": "Source",
  "Scale": 0.5
}
```

- 当前版本按 `目标 ProgressToTurn * Scale` 调整行动条

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `Scale` | `double` | 对目标 `ProgressToTurn` 的缩放倍率 |

#### UnitContinueAttackBuildData

```json
{
  "Uuid": "4aa3d705-f72e-47e5-95c5-e98b7da2fd13",
  "Target": "Target",
  "Count": 1
}
```

- 对已命中的目标追加连击
- 连击伤害会继续叠加 `ICalculateContinueAttackDamageTracker`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `Count` | `int` | 基础追加连击次数；运行时还会叠加 `ExtraContinueAttack` 的层数 |

#### UnitInvokeFollowAttackBuildData

```json
{
  "Uuid": "e7342783-2c6c-460b-a1a6-d1c737dac794",
  "Target": "Target",
  "InvokeRange": "BackColumn",
  "InvokeCount": 1
}
```

- 对首个选中目标触发追击调用
- 追击中的追击不会再次递归触发

`InvokeRange` 可选值：

| 值 | 说明 |
| --- | --- |
| `FrontColumn` | 只从己方前列单位中检索追击者；会排除当前 `source` |
| `BackColumn` | 只从己方后列单位中检索追击者；会排除当前 `source` |
| `Owner` | 只从当前 Tracker 的 `Owner` 中检索追击者 |
| `EffectSource` | 只从当前 Tracker 的 `EffectSource` 中检索追击者 |
| 其他值或留空 | 走默认分支：从己方全部存活单位中检索，并排除当前 `source` |

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `InvokeRange` | `string` | 追击检索范围字符串；当前支持 `FrontColumn`、`BackColumn`、`Owner`、`EffectSource` |
| `InvokeCount` | `int` | 触发的追击次数，默认值为 `1` |

## 9. 当前版本的作者工作流

1. 为内容确定 `FullName`。
2. 在 `manifest.json` 中注册 `ConfigMaps`。
3. 为展示文本补齐本地化键。
4. 心法、装备、独立效果中的复杂机制统一写在 `IEffect.Data.Context`。
5. 招式中的直接流程写在五段阶段数组里。
6. 数值型 Tracker 使用 `SaveIntBuildData` 或 `SaveDoubleBuildData`。
7. 事件型 Tracker 使用会生成节点的 BuildData。
8. 需要复用的 Buff、Debuff、DoT 抽成独立战斗效果，再通过 `FlexibleEffectNames` 引用。
