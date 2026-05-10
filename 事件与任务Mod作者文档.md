# 事件与任务 MOD 官方指南

这份文档说明当前版本中，事件与任务 MOD 应该如何通过 manifest 接入游戏，并稳定地产生预期效果。

适用范围：

- 新增事件组或事件条目
- 覆盖现有事件配置
- 新增任务或覆盖现有任务配置
- 为任务补充 Naninovel 脚本
- 为任务补充本地化文本与任务飞鸽文本

## 1. MOD 通过 manifest 接入游戏

事件、任务、脚本和本地化都通过 manifest 描述，然后由游戏按逻辑名加载。

与事件和任务最相关的 manifest 字段如下：

```json
{
  "Name": "示例 MOD",
  "AffectsSave": true,
  "ConfigMaps": [
    {
      "FullName": "Event/Character/方屿迟",
      "Path": "Configs/Event/FangYuChi.json"
    },
    {
      "FullName": "Quest/支线/帮忙采药",
      "Path": "Configs/Quest/HelpCollectHerbs.json"
    }
  ],
  "ConfigPatchMaps": [
    {
      "FullName": "Event/Event",
      "Path": "Patches/EventList.json"
    },
    {
      "FullName": "Quests",
      "Path": "Patches/QuestList.json"
    }
  ],
  "NaninovelScriptMaps": [
    {
      "FullName": "Character/方屿迟/Basic",
      "Path": "Nani/Character.Basic.nani"
    }
  ],
  "LocalizationMaps": [
    {
      "Locale": "zh-Hans",
      "Path": "Localization/zh-Hans.json"
    }
  ]
}
```

字段作用如下：

- `ConfigMaps`：提供完整配置。`FullName` 是游戏读取时使用的逻辑名，`Path` 是该文件在 MOD 内的相对路径。
- `ConfigPatchMaps`：对现有逻辑名对应的 JSON 做补丁，适合把新事件、新任务登记进总表，或对现有总表做增量修改。
- `NaninovelScriptMaps`：提供 Naninovel 脚本。脚本的 `FullName` 必须和 `EventResult` 最终引用到的脚本名一致。
- `LocalizationMaps`：导入本地化文本。一个本地化文件可以同时写多个字符串表。
- `AffectsSave`：事件与任务 MOD 应保持为 `true`。

### 1.1 逻辑名规则

当前版本中，与事件和任务相关的逻辑名如下：

- 事件总表：`Event/Event`
- 单个事件组文件：`Event/{组名}/{文件名}`
- 任务总表：`Quests`
- 单个任务文件：`Quest/{组名}/{任务名}`
- Naninovel 脚本：脚本逻辑名本身，例如 `Main0201`、`Character/方屿迟/Basic`、`Quest/整治集市恶霸一`

覆盖现有事件或任务时，`ConfigMaps.FullName` 直接写现有逻辑名即可。新增事件或任务时，除了提供该逻辑名对应的完整配置，还要补丁总表，把新名字接入游戏入口。

### 1.2 配置覆盖与补丁的生效方式

同一个逻辑名在当前版本中的处理规则是：

- `ConfigMaps` 为这个逻辑名提供当前要读取的完整内容。
- `ConfigPatchMaps` 会在当前完整内容之上继续应用补丁。
- 如果一个逻辑名已经存在于游戏中，而 MOD 只是覆盖它的内容，不需要再次把它补进总表。
- 如果一个逻辑名是新增的，就必须把它补进对应总表，否则游戏不会主动加载它。

## 2. 用补丁把新内容接入总表

当前版本支持两种补丁写法：

- 简写补丁：基础写法。补丁文件顶层不写 `Operations`、`Patches` 或 `ops`，游戏会按简写规则自动展开。
- JSONPath 补丁：高级写法。手动写 `Operations` 数组和 `Path`。

### 2.1 简写补丁

当补丁文件顶层是普通对象时，游戏会从根节点开始递归展开补丁。

简写补丁的操作后缀如下：

- `Set`
- `ArrayAdd`
- `ArrayRemoveString`
- `ArrayRemoveObjectByKey`
- `ArrayModifyObjectByKey`

对应到简写键名时，写法如下：

- `字段.Set`：设置字段
- `字段.Add`：向数组追加内容。值既可以是单个元素，也可以是数组；如果目标数组里已经有完全相同的元素，会自动跳过
- `字段.Remove`：从字符串数组里移除指定字符串。值既可以是单个字符串，也可以是字符串数组
- `字段.Remove.键名`：从对象数组里移除某个键等于指定值的对象。值既可以是单个值，也可以是值数组
- `字段.Modify.键名`：从对象数组里找到某个键等于指定值的对象，然后用提供的新对象整条替换

例如：

```json
{
  "Groups.Add": {
    "Name": "我的角色任务",
    "QuestNames": ["初次来信"]
  }
}
```

这份补丁等价于：向 `Groups` 数组追加一个新对象。

批量追加时可以直接传数组：

```json
{
  "Groups": [
    {
      "Name": "支线",
      "QuestNames.Add": ["帮忙采药", "打听消息"]
    }
  ]
}
```

如果 `QuestNames` 里已经存在同名项，追加时会自动跳过，不会重复写入。

按键修改对象时，可以写成：

```json
{
  "Groups.Modify.Name": {
    "Name": "支线",
    "QuestNames": ["任务一", "任务二", "帮忙采药", "打听消息"]
  }
}
```

这份补丁会先在 `Groups` 数组里找到 `Name == "支线"` 的对象，然后用这里提供的整个对象替换原对象。

#### 2.1.1 用简写补丁把新事件接入事件总表

事件总表逻辑名是 `Event/Event`，它的结构核心是：

```json
{
  "List": [
    {
      "Name": "System",
      "List": ["事件名1", "事件名2"],
      "PriorityBase": 0
    }
  ]
}
```

向现有组追加一个事件文件名：

```json
{
  "List": [
    {
      "Name": "Character",
      "List.Add": "方屿迟"
    }
  ]
}
```

新增整个事件组：

```json
{
  "List.Add": {
    "Name": "MyEventGroup",
    "PriorityBase": 0,
    "List": ["事件一"]
  }
}
```

#### 2.1.2 用简写补丁把新任务接入任务总表

任务总表逻辑名是 `Quests`，它的结构核心是：

```json
{
  "Groups": [
    {
      "Name": "支线",
      "QuestNames": ["任务一", "任务二"]
    }
  ]
}
```

向现有组追加一个任务名：

```json
{
  "Groups": [
    {
      "Name": "支线",
      "QuestNames.Add": "帮忙采药"
    }
  ]
}
```

新增整个任务组：

```json
{
  "Groups.Add": {
    "Name": "我的角色任务",
    "QuestNames": ["初次来信"]
  }
}
```

#### 2.1.3 简写补丁的定位规则

当数组元素是对象时，游戏会优先查找这些字段作为选择器：

- `Name`
- `FullName`
- `Uuid`
- `Id`
- `Key`

例如这份补丁：

```json
{
  "Groups": [
    {
      "Name": "支线",
      "QuestNames.Add": "帮忙采药"
    }
  ]
}
```

会先用 `Name == "支线"` 在 `Groups` 数组里找到对应对象，再对这个对象的 `QuestNames` 执行追加。

### 2.2 JSONPath 补丁

当需要直接指定目标路径时，可以显式写 `Operations` 数组。当前版本支持的操作类型有：

- `Set`
- `ArrayAdd`
- `ArrayRemoveString`
- `ArrayRemoveObjectByKey`
- `ArrayModifyObjectByKey`

标准格式如下：

```json
{
  "Operations": [
    {
      "Type": "ArrayAdd",
      "Path": "$.Groups[?(@.Name == \"支线\")].QuestNames",
      "Value": ["帮忙采药", "打听消息"]
    }
  ]
}
```

`Path` 使用 JSONPath 语法。像 `$.Groups[?(@.Name == "支线")].QuestNames` 这种写法，表示从根对象进入 `Groups` 数组，筛出 `Name` 等于 `支线` 的对象，再取它的 `QuestNames`。

`ArrayAdd`、`ArrayRemoveString`、`ArrayRemoveObjectByKey` 的 `Value` 都可以写成单个值或数组。`ArrayAdd` 遇到已存在的完全相同元素时会自动跳过。

如果想按某个键修改对象数组里的对象，也可以显式写：

```json
{
  "Operations": [
    {
      "Type": "ArrayModifyObjectByKey",
      "Path": "$.Groups",
      "Key": "Name",
      "Value": {
        "Name": "支线",
        "QuestNames": ["任务一", "任务二", "帮忙采药"]
      }
    }
  ]
}
```

这个操作会找到 `Groups` 中 `Name == "支线"` 的对象，然后用 `Value` 提供的整个对象替换原对象。

#### 2.2.1 用 JSONPath 把新事件接入事件总表

事件总表逻辑名是 `Event/Event`，它的结构核心是：

```json
{
  "List": [
    {
      "Name": "System",
      "List": ["事件名1", "事件名2"],
      "PriorityBase": 0
    }
  ]
}
```

向现有组追加一个事件文件名：

```json
{
  "Operations": [
    {
      "Type": "ArrayAdd",
      "Path": "$.List[?(@.Name == \"Character\")].List",
      "Value": "方屿迟"
    }
  ]
}
```

新增整个事件组：

```json
{
  "Operations": [
    {
      "Type": "ArrayAdd",
      "Path": "$.List",
      "Value": {
        "Name": "MyEventGroup",
        "PriorityBase": 0,
        "List": ["事件一"]
      }
    }
  ]
}
```

#### 2.2.2 用 JSONPath 把新任务接入任务总表

任务总表逻辑名是 `Quests`，它的结构核心是：

```json
{
  "Groups": [
    {
      "Name": "支线",
      "QuestNames": ["任务一", "任务二"]
    }
  ]
}
```

向现有组追加一个任务名：

```json
{
  "Operations": [
    {
      "Type": "ArrayAdd",
      "Path": "$.Groups[?(@.Name == \"支线\")].QuestNames",
      "Value": "帮忙采药"
    }
  ]
}
```

新增整个任务组：

```json
{
  "Operations": [
    {
      "Type": "ArrayAdd",
      "Path": "$.Groups",
      "Value": {
        "Name": "我的角色任务",
        "QuestNames": ["初次来信"]
      }
    }
  ]
}
```

## 3. 事件 MOD

### 3.1 事件组文件结构

一个事件组文件的最小结构如下：

```json
{
  "ObjectType": "WuXia.Events.EventGroup",
  "FlexibleTrackers": [],
  "InteractWithMultipleCharactersTrackers": []
}
```

当前版本中，事件主体通常写在 `FlexibleTrackers` 里。

### 3.2 `FlexibleTrackers` 的关键字段

```json
{
  "Uuid": "guid",
  "ContextObjectType": "WuXia.StartTurnContext",
  "ExpressionCondition": "",
  "TurnCondition": 0,
  "FlagCondition": "",
  "EventResult": "Main0001",
  "ActionResultExpression": "",
  "ActionResultExpressions": [],
  "Active": true,
  "Repeatable": false,
  "Priority": 0,
  "HideMarker": true
}
```

这些字段的作用如下：

- `Uuid`：事件追踪标识。应为合法且唯一的 GUID 字符串。
- `ContextObjectType`：事件监听的上下文类型。
- `TurnCondition`：用于精确回合触发。
- `ExpressionCondition`：用于复杂条件判断。
- `FlagCondition`：用于标记判断。
- `EventResult`：触发后返回一个事件给节点系统继续执行。
- `ActionResultExpression`、`ActionResultExpressions`：触发后立即执行表达式。
- `Repeatable`：决定是否可重复触发。
- `Priority`：和事件组的 `PriorityBase` 相加后作为最终优先级。
- `HideMarker`：控制标记显示。

### 3.3 当前版本支持的上下文类型

可直接使用的上下文类型如下：

- `WuXia.StartTurnContext`
- `WuXia.EndTurnContext`
- `WuXia.InteractWithCharacterContext`
- `WuXia.EnterLocationContext`
- `WuXia.StopSceneContext`
- `WuXia.UsePigeonLetterContext`

`ContextObjectType` 必须写完整类型名。

### 3.4 `EventResult` 的写法

当前版本支持三种格式：

- `"Main0201"`
- `"Type;Name"`
- `"Type;Name;Arg"`

事件类型包括：

- `Simple`
- `Nani`
- `Location`
- `Scroll`
- `Battle`
- `Menu`
- `Highlight`
- `MiniGame`

当 `EventResult` 只写一段字符串时，游戏会把它当成 Naninovel 脚本名。此时，脚本需要通过 `NaninovelScriptMaps` 提供同名资源。

### 3.5 固定回合触发事件

```json
{
  "ObjectType": "WuXia.Events.EventGroup",
  "FlexibleTrackers": [
    {
      "Uuid": "2d8ccf14-4a9f-4568-8e18-50f2b8c92a10",
      "ContextObjectType": "WuXia.StartTurnContext",
      "TurnCondition": 8,
      "EventResult": "Side00401",
      "Active": true,
      "Repeatable": false,
      "HideMarker": true
    }
  ]
}
```

这一写法适合：

- 某回合自动播剧情
- 某回合解锁地点或系统
- 某回合开启新的角色交互内容

### 3.6 角色交互时切入脚本

```json
{
  "ObjectType": "WuXia.Events.EventGroup",
  "FlexibleTrackers": [
    {
      "Uuid": "6dc9e9d7-c6f0-4e15-8fd6-c2b9dc173812",
      "ContextObjectType": "WuXia.InteractWithCharacterContext",
      "ExpressionCondition": "context.Character.Name==\"方屿迟\" && !context.PreferLocationActionScript",
      "EventResult": "Character/方屿迟/Basic",
      "Active": true,
      "Repeatable": true,
      "HideMarker": true
    }
  ]
}
```

这一写法适合：

- 为角色提供通用地点闲聊
- 在交互时覆盖默认脚本
- 按条件把同一角色切进不同脚本

### 3.7 只执行副作用

```json
{
  "ObjectType": "WuXia.Events.EventGroup",
  "FlexibleTrackers": [
    {
      "Uuid": "68f98885-3ac2-4a8e-bc5c-ff9b3509f5e1",
      "ContextObjectType": "WuXia.StartTurnContext",
      "ExpressionCondition": "GameManager.CurrentTurn == 21",
      "ActionResultExpression": "QuestManager.UpdateAvailableWuXiangGeQuests()",
      "Active": true,
      "Repeatable": false,
      "HideMarker": true
    }
  ]
}
```

这一写法适合：

- 刷新任务
- 执行状态切换
- 在特定时机调用表达式副作用

### 3.8 注意事项

- 需要播剧情或进入场景时，填写 `EventResult`。
- 只需要执行逻辑表达式时，填写 `ActionResultExpression` 或 `ActionResultExpressions`。
- 最终优先级等于 `事件组 PriorityBase + Tracker.Priority`。

## 4. 任务 MOD

### 4.1 任务文件最小结构

```json
{
  "ObjectType": "WuXia.Quest.Quest",
  "Name": "任务名",
  "Type": "Main"
}
```

`Type` 可用值如下：

- `Main`
- `Side`
- `WuXiangGe`
- `Hidden`

它们直接影响任务显示方式和运行逻辑：

- `Main`：主线任务优先级更高。
- `WuXiangGe`：参与无相阁刷新、接取与过期流程。
- `Hidden`：不显示任务提示，也不出现在飞鸽任务列表里。

### 4.2 当前版本会读取的顶层字段

任务文件中，当前版本会读取这些顶层字段：

- `Name`
- `Type`
- `Source`
- `ImagePath`
- `RewardItems`
- `RewardGold`
- `RewardReputation`
- `IsRewardUnknown`
- `ExpiredTurn`
- `Stages`
- `PassStages`
- `PigeonLetterStages`
- `WuXiangGeStages`
- `RequestItemsStages`
- `CountStatisticStages`
- `ReachAttributeStages`
- `TrackAttributeStages`
- `WaitTurnStartStages`
- `WaitTurnEndStages`
- `InteractableCharacterStages`
- `EnterLocationStages`
- `ComplexStages`

### 4.3 任务本地化键

任务界面会按以下键名读取本地化文本：

任务名：

- `Quest.{任务名}`

任务描述：

- `Quest.{任务名}.Description.{DescriptionIndex}`

任务类型：

- `Quest.Type.{Type}`

任务飞鸽：

- `Index <= 0` 时：
  - `Quest.{任务名}.Opening`
  - `Quest.{任务名}.Context`
  - `Quest.{任务名}.Sign`
- `Index > 0` 时：
  - `Quest.{任务名}.{Index}.Opening`
  - `Quest.{任务名}.{Index}.Context`
  - `Quest.{任务名}.{Index}.Sign`

`LocalizationMaps` 指向的本地化文件格式如下：

```json
{
  "Tables": [
    {
      "Table": "Quest",
      "Entries": [
        {
          "Key": "Quest.帮忙采药",
          "Value": "帮忙采药"
        },
        {
          "Key": "Quest.帮忙采药.Description.1",
          "Value": "去见发布委托的人。"
        }
      ]
    }
  ]
}
```

`Table` 填写你要写入的字符串表名，`Entries` 中的 `Key` 按本节列出的规则组织即可。

### 4.4 普通任务的推进规则

当 `ComplexStages` 为空时，任务会把下列阶段数组合并为统一流程：

- `Stages`
- `PassStages`
- `PigeonLetterStages`
- `WuXiangGeStages`
- `RequestItemsStages`
- `CountStatisticStages`
- `ReachAttributeStages`
- `TrackAttributeStages`
- `WaitTurnStartStages`
- `WaitTurnEndStages`
- `InteractableCharacterStages`
- `EnterLocationStages`

合并完成后，游戏会按 `(Status, SubStatus)` 排序推进。

普通任务编排时，应把每一个阶段放在唯一的 `(Status, SubStatus)` 上。`quest.Forward()` 前进到的是排序后的下一个阶段。

### 4.5 常用任务结构

#### 回合触发 -> 进入地点继续 -> 完成

```json
{
  "ObjectType": "WuXia.Quest.Quest",
  "Name": "示例任务",
  "Type": "Main",
  "WaitTurnStartStages": [
    {
      "Uuid": "328e3544-23d2-4d9e-b1d6-9c1d198fd8fb",
      "Status": "Unknown",
      "SubStatus": 0,
      "Turn": 7,
      "EventResult": "Main0201"
    }
  ],
  "EnterLocationStages": [
    {
      "Uuid": "1e8187a0-c0cd-4b25-9bc7-4b85f317d814",
      "Status": "Ongoing",
      "SubStatus": 0,
      "LocationFullName": "Map.书院",
      "EventResult": "Main0202"
    }
  ],
  "Stages": [
    {
      "Uuid": "7dc344d5-bcd0-4991-b86f-2b0f6ef3e0ec",
      "Status": "Finished",
      "SubStatus": 0
    }
  ]
}
```

这一结构用于：

- 到指定回合出现入口
- 进入指定地点后继续剧情
- 在脚本里推进到完成状态

#### 角色接取 -> 条件满足后交付 -> 完成

```json
{
  "ObjectType": "WuXia.Quest.Quest",
  "Name": "帮忙采药",
  "Type": "Side",
  "InteractableCharacterStages": [
    {
      "Uuid": "7fc1de6d-2c89-4b51-93b2-b2204912cfa0",
      "Status": "Unknown",
      "SubStatus": 0,
      "CharacterFullName": "Character.发布人",
      "RelatedLocationFullName": "Map.书院.药园",
      "EventResult": "Quest/帮忙采药.接取"
    },
    {
      "Uuid": "fd729863-20a0-4d24-b843-88e8070a9f7b",
      "Status": "Ongoing",
      "SubStatus": 1,
      "CharacterFullName": "Character.发布人",
      "RelatedLocationFullName": "Map.书院.药园",
      "Condition": "GameManager.Instance.Player.Bag.HasItem(\"Item.Material.白芷\", 3)",
      "EventResult": "Quest/帮忙采药.交付"
    }
  ],
  "Stages": [
    {
      "Uuid": "8f508130-dda7-49cd-87a7-7426175a72d9",
      "Status": "Ongoing",
      "SubStatus": 0,
      "DescriptionIndex": 1
    },
    {
      "Uuid": "2c32eab5-f0c1-4b16-9457-c44c2e06bc7d",
      "Status": "Finished",
      "SubStatus": 0,
      "DescriptionIndex": 2
    }
  ]
}
```

#### 飞鸽触发 -> 见人 -> 完成

```json
{
  "ObjectType": "WuXia.Quest.Quest",
  "Name": "来信相邀",
  "Type": "Side",
  "PigeonLetterStages": [
    {
      "Uuid": "f69a367e-5164-43b0-a35a-0d5f38ca26d4",
      "Status": "Unknown",
      "SubStatus": 0,
      "Index": 1,
      "Condition": "true"
    }
  ],
  "InteractableCharacterStages": [
    {
      "Uuid": "8f5e8137-accc-4f9a-9508-9d3251a8a97b",
      "Status": "Available",
      "SubStatus": 0,
      "CharacterFullName": "Character.寄信人",
      "RelatedLocationFullName": "Map.城镇",
      "EventResult": "Quest/来信相邀"
    }
  ],
  "Stages": [
    {
      "Uuid": "7ee8a203-c989-4fab-a316-5da7dfbe7e96",
      "Status": "Ongoing",
      "SubStatus": 0,
      "DescriptionIndex": 1
    },
    {
      "Uuid": "af3a6357-d04f-4b71-8f8c-c97b487ea736",
      "Status": "Finished",
      "SubStatus": 0,
      "DescriptionIndex": 2
    }
  ]
}
```

### 4.6 各阶段类型的行为

#### `WaitTurnStartStage`

- 监听回合开始。
- 只有 `CurrentTurn == Turn` 时触发。
- `Condition` 在这个精确回合一起判断。
- 触发后会失效。

#### `WaitTurnEndStage`

- 监听回合结束。
- 只有 `CurrentTurn == Turn` 时触发。
- 支持 `EventResult` 和 `ActionResultExpression`。

#### `InteractableCharacterStage`

- 监听角色交互。
- 单个目标角色用 `CharacterFullName`。
- 多个目标角色用 `Characters`。
- `RelatedLocationFullName` 用于任务提示。
- `AddToLocation` 可在进入阶段时把角色摆进地点。
- `Repeatable=false` 时，首次触发后失效。

常用字段如下：

- `CharacterFullName`
- `Characters`
- `RelatedLocationFullName`
- `AddToLocation`
- `Condition`
- `EventResult`
- `Repeatable`
- `EnterAddItems`
- `LeaveRemoveItems`
- `ReachPoint`
- `SortingLayer`
- `SortingOrder`

#### `EnterLocationStage`

- 监听进入地点。
- 用 `LocationFullName` 精确匹配进入地点全名。
- 支持 `Condition`。
- 触发后失效。

#### `PigeonLetterStage`

- 这个阶段本身不注册 Tracker。
- 当前阶段是它，且 `Condition` 为真时，任务飞鸽会显示在界面上。
- 玩家阅读后，任务会立刻执行 `quest.Forward()`。
- `Index` 决定飞鸽本地化键是否带编号。

注意事项：

- `PigeonLetterStage` 后面必须接一个可到达的下一阶段。

#### `RequestItemsStage`

- `RequestItems` 定义交付需求。
- 任务检查的是玩家是否持有足够道具。
- 离开该阶段且不是过期结束时，游戏会自动扣除提交的道具。

#### `CountStatisticStage`

- 进入阶段时记录起始统计值。
- 完成条件是 `当前统计值 >= 起始值 + Count`。

#### `TrackAttributeStage`

- 目标值不是绝对值，而是“月初快照值 + Value”。
- 当前属性达到这个目标后完成。
- 任务描述可使用这些变量：
  - `currentValue`
  - `monthStartValue`
  - `targetValue`
  - `value`

注意事项：

- `TrackAttributeStage` 描述的是“本月增长量”。

#### `ReachAttributeStage`

- 判断当前属性绝对值是否达到 `Value`。

#### `WuXiangGeStage`

- 定义任务参与无相阁刷新时的分组与时间窗。
- 被抽中后任务会进入 `Available`。
- 接取后，`ExpiredTurn` 会被设置到当月月末。

#### `PassStage`

- 进入后可直接视为满足完成条件。

### 4.7 复杂任务 `ComplexStages`

当任务写入 `ComplexStages` 时，任务会切换到复杂任务流程：

- 普通阶段数组不再作为主推进链。
- 当前激活阶段由 `CurrentStages` 决定。
- 复杂阶段结束依赖条件命中或脚本显式离开。

当前版本支持的复杂阶段子类型如下：

- `Stage`
- `InteractableCharacterStage`
- `EnterLocationStage`
- `WaitTurnStartStage`
- `WaitTurnEndStage`

复杂任务常用字段如下：

- `AllConditions`：这些阶段状态全部出现后才满足条件。
- `AnyConditions`：这些阶段状态至少出现一个后满足条件。
- `StatusToFinishs`：当前复杂阶段完成时，同时把这些阶段标记为完成。

### 4.8 任务脚本推进规则

事件和任务 JSON 负责把入口放到正确的时间、地点或角色上；剧情播完以后，任务状态应在脚本中显式推进。

普通任务脚本中使用：

- `quest.Update(...)`
- `quest.Forward()`
- `quest.Forward(true)`

复杂任务脚本中使用：

- `quest.LeaveComplexStage(...)`

任务脚本名通过 `EventResult` 指向时，脚本文件要在 `NaninovelScriptMaps` 中以同名 `FullName` 提供。

### 4.9 注意事项

- 普通任务中，每个 `(Status, SubStatus)` 只放一个阶段。
- `DescriptionIndex` 表示任务日志条目编号，不是“当前阶段唯一一句描述”。
- `LocationFullName` 和 `RelatedLocationFullName` 都应填写完整地点名。

## 5. 最小制作流程

### 5.1 新增事件

1. 准备事件组配置文件，逻辑名写成 `Event/{组名}/{文件名}`。
2. 在 manifest 的 `ConfigMaps` 中登记这份事件组文件。
3. 为 `Event/Event` 准备补丁，把 `{文件名}` 加进对应组的 `List`；如果组不存在，就先追加新组对象。
4. 如果事件会触发 Naninovel 脚本，在 `NaninovelScriptMaps` 中提供对应脚本。

### 5.2 新增任务

1. 准备任务配置文件，逻辑名写成 `Quest/{组名}/{任务名}`。
2. 在 manifest 的 `ConfigMaps` 中登记这份任务文件。
3. 为 `Quests` 准备补丁，把 `{任务名}` 加进对应组的 `QuestNames`；如果组不存在，就先追加新组对象。
4. 补充任务用到的 Naninovel 脚本与本地化文本。
5. 在任务脚本里显式推进状态。

### 5.3 覆盖现有事件或任务

1. 在 `ConfigMaps` 中使用现有逻辑名。
2. 提供新的完整配置内容。
3. 当注册名没有变化时，不需要修改总表补丁。

## 6. 提交前检查

- manifest 中每个 `FullName` 都与游戏实际读取的逻辑名一致。
- 新增事件已经通过 `Event/Event` 补丁接入总表。
- 新增任务已经通过 `Quests` 补丁接入总表。
- 所有 `Uuid` 都合法且唯一。
- 任务本地化键已经补齐。
- 任务飞鸽使用了正确的键名规则。
- 所有 `EventResult` 指向的脚本都已经通过 `NaninovelScriptMaps` 提供。
- 任务脚本已经显式推进状态。
