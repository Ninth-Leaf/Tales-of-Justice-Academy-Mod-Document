# Official Guide to Event and Quest Mods

This guide explains how Event and Quest Mods should connect to the current version through the manifest so they produce stable, expected behavior.

Scope:

- Add new Event groups or Event entries
- Override existing Event configs
- Add new Quest configs or override existing Quest configs
- Add Naninovel scripts for Quests
- Add localization text and Quest Pigeon Post text

## 1. Connect Mods Through the manifest

Events, Quests, scripts, and localization are all described in the manifest and then loaded by the game through their logical names.

The manifest fields most relevant to Events and Quests are:

```json
{
  "Name": "Sample MOD",
  "AffectsSave": true,
  "ConfigMaps": [
    {
      "FullName": "Event/Character/方屿迟",
      "Path": "Configs/Event/FangYuChi.json"
    },
    {
      "FullName": "Quest/支线/HelpCollectHerbs",
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

These fields work as follows:

- `ConfigMaps`: provides full config content. `FullName` is the logical name used by the game at runtime, and `Path` is the relative path inside the Mod.
- `ConfigPatchMaps`: applies patches on top of the current full JSON for a logical name. Use this to register new Events or Quests into the master list, or to incrementally patch an existing master list.
- `NaninovelScriptMaps`: provides Naninovel scripts. The script `FullName` must exactly match the script name referenced by `EventResult`.
- `LocalizationMaps`: imports localization text. One localization file can contain multiple string tables.
- `AffectsSave`: Event and Quest Mods should keep this as `true`.

### 1.1 Logical Name Rules

In the current version, the logical names related to Events and Quests are:

- Event master list: `Event/Event`
- Single Event group file: `Event/{groupName}/{fileName}`
- Quest master list: `Quests`
- Single Quest file: `Quest/{groupName}/{questName}`
- Naninovel script: the script logical name itself, such as `Main0201`, `Character/方屿迟/Basic`, or `Quest/整治集市恶霸一`

If you override an existing Event or Quest, write the current logical name directly in `ConfigMaps.FullName`. If you add a new Event or Quest, you must provide the full config for that logical name and also patch the master list so the game can discover it.

### 1.2 How Overrides and Patches Take Effect

The current version handles one logical name with these rules:

- `ConfigMaps` provides the current full content for that logical name.
- `ConfigPatchMaps` then applies additional patches on top of that current full content.
- If the logical name already exists in the game and the Mod only overrides its content, you do not need to add it into the master list again.
- If the logical name is new, you must patch it into the matching master list, or the game will not load it on its own.

## 2. Use Patches to Connect New Content to the Master List

The current version supports two patch styles:

- Shorthand patches: the basic format. If the patch file root does not contain `Operations`, `Patches`, or `ops`, the game expands it automatically as shorthand.
- JSONPath patches: the advanced format. You write the `Operations` array and `Path` explicitly.

### 2.1 Shorthand Patches

When the patch file root is a normal object, the game recursively expands patches starting from the root node.

The shorthand operation suffixes are:

- `Set`
- `ArrayAdd`
- `ArrayRemoveString`
- `ArrayRemoveObjectByKey`
- `ArrayModifyObjectByKey`

Mapped to shorthand keys, the format is:

- `field.Set`: set the field
- `field.Add`: append to an array. The value can be one element or an array. If the target array already contains an identical element, it is skipped automatically.
- `field.Remove`: remove a string from a string array. The value can be one string or an array of strings.
- `field.Remove.keyName`: remove objects from an object array where a key equals the given value. The value can be one value or an array of values.
- `field.Modify.keyName`: find an object in an object array where a key equals the given value, then replace the whole object with the provided new object.

For example:

```json
{
  "Groups.Add": {
    "Name": "MyCharacterQuest",
    "QuestNames": ["FirstLetter"]
  }
}
```

This patch is equivalent to appending a new object to the `Groups` array.

For batch append, you can pass an array directly:

```json
{
  "Groups": [
    {
      "Name": "支线",
      "QuestNames.Add": ["HelpCollectHerbs", "InquireAboutInformation"]
    }
  ]
}
```

If `QuestNames` already contains the same name, the append is skipped automatically and no duplicate is written.

To modify an object by key, you can write:

```json
{
  "Groups.Modify.Name": {
    "Name": "支线",
    "QuestNames": ["HelpCollectHerbs", "InquireAboutInformation"]
  }
}
```

This patch first finds the object in `Groups` where `Name == "支线"`, then replaces the original object with the object provided here.

#### 2.1.1 Use Shorthand Patches to Connect a New Event to the Event Master List

The Event master list logical name is `Event/Event`, and its core structure is:

```json
{
  "List": [
    {
      "Name": "System",
      "List": ["EventName1", "EventName2"],
      "PriorityBase": 0
    }
  ]
}
```

Append an Event file name to an existing group:

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

Add a whole new Event group:

```json
{
  "List.Add": {
    "Name": "MyEventGroup",
    "PriorityBase": 0,
    "List": ["EventName1"]
  }
}
```

#### 2.1.2 Use Shorthand Patches to Connect a New Quest to the Quest Master List

The Quest master list logical name is `Quests`, and its core structure is:

```json
{
  "Groups": [
    {
      "Name": "支线",
      "QuestNames": ["QuestName1", "QuestName2"]
    }
  ]
}
```

Append a Quest name to an existing group:

```json
{
  "Groups": [
    {
      "Name": "支线",
      "QuestNames.Add": "HelpCollectHerbs"
    }
  ]
}
```

Add a whole new Quest group:

```json
{
  "Groups.Add": {
    "Name": "MyCharacterQuest",
    "QuestNames": ["FirstLetter"]
  }
}
```

#### 2.1.3 Selector Rules for Shorthand Patches

When array elements are objects, the game prefers these fields as selectors:

- `Name`
- `FullName`
- `Uuid`
- `Id`
- `Key`

For example, this patch:

```json
{
  "Groups": [
    {
      "Name": "支线",
      "QuestNames.Add": "HelpCollectHerbs"
    }
  ]
}
```

first finds the object in `Groups` where `Name == "支线"`, then appends to that object's `QuestNames`.

### 2.2 JSONPath Patches

When you need to point directly at a target path, you can write an explicit `Operations` array. The current version supports these operation types:

- `Set`
- `ArrayAdd`
- `ArrayRemoveString`
- `ArrayRemoveObjectByKey`
- `ArrayModifyObjectByKey`

The standard format is:

```json
{
  "Operations": [
    {
      "Type": "ArrayAdd",
      "Path": "$.Groups[?(@.Name == \"支线\")].QuestNames",
      "Value": ["HelpCollectHerbs", "InquireAboutInformation"]
    }
  ]
}
```

`Path` uses JSONPath syntax. A path like `$.Groups[?(@.Name == "支线")].QuestNames` means: from the root object, enter the `Groups` array, filter objects where `Name` equals `支线`, then take their `QuestNames`.

For `ArrayAdd`, `ArrayRemoveString`, and `ArrayRemoveObjectByKey`, `Value` can be a single value or an array. `ArrayAdd` automatically skips identical elements that already exist.

If you want to modify an object in an object array by key, you can also write:

```json
{
  "Operations": [
    {
      "Type": "ArrayModifyObjectByKey",
      "Path": "$.Groups",
      "Key": "Name",
      "Value": {
        "Name": "支线",
        "QuestNames": ["QuestName1", "QuestName2", "HelpCollectHerbs"]
      }
    }
  ]
}
```

This operation finds the object in `Groups` where `Name == "支线"`, then replaces it with the object in `Value`.

#### 2.2.1 Use JSONPath to Connect a New Event to the Event Master List

The Event master list logical name is `Event/Event`, and its core structure is:

```json
{
  "List": [
    {
      "Name": "System",
      "List": ["EventName1", "EventName2"],
      "PriorityBase": 0
    }
  ]
}
```

Append an Event file name to an existing group:

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

Add a whole new Event group:

```json
{
  "Operations": [
    {
      "Type": "ArrayAdd",
      "Path": "$.List",
      "Value": {
        "Name": "MyEventGroup",
        "PriorityBase": 0,
        "List": ["EventName1"]
      }
    }
  ]
}
```

#### 2.2.2 Use JSONPath to Connect a New Quest to the Quest Master List

The Quest master list logical name is `Quests`, and its core structure is:

```json
{
  "Groups": [
    {
      "Name": "支线",
      "QuestNames": ["QuestName1", "QuestName2"]
    }
  ]
}
```

Append a Quest name to an existing group:

```json
{
  "Operations": [
    {
      "Type": "ArrayAdd",
      "Path": "$.Groups[?(@.Name == \"支线\")].QuestNames",
      "Value": "HelpCollectHerbs"
    }
  ]
}
```

Add a whole new Quest group:

```json
{
  "Operations": [
    {
      "Type": "ArrayAdd",
      "Path": "$.Groups",
      "Value": {
        "Name": "MyCharacterQuest",
        "QuestNames": ["FirstLetter"]
      }
    }
  ]
}
```

## 3. Event Mods

### 3.1 Event Group File Structure

The minimum structure of an Event group file is:

```json
{
  "ObjectType": "WuXia.Events.EventGroup",
  "FlexibleTrackers": [],
  "InteractWithMultipleCharactersTrackers": []
}
```

In the current version, the main Event body is usually written in `FlexibleTrackers`.

### 3.2 Key Fields in `FlexibleTrackers`

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

These fields work as follows:

- `Uuid`: Event tracking identifier. Use a legal and unique GUID string.
- `ContextObjectType`: the context type that the Event listens to.
- `TurnCondition`: used for exact turn triggers.
- `ExpressionCondition`: used for complex conditions.
- `FlagCondition`: used for flag checks.
- `EventResult`: returns an Event to the node system after the trigger.
- `ActionResultExpression`, `ActionResultExpressions`: run expressions immediately after the trigger.
- `Repeatable`: controls whether the Event can trigger more than once.
- `Priority`: added to the Event group's `PriorityBase` to form the final priority.
- `HideMarker`: controls marker visibility.

### 3.3 Context Types Supported in the Current Version

The following context types can be used directly:

- `WuXia.StartTurnContext`
- `WuXia.EndTurnContext`
- `WuXia.InteractWithCharacterContext`
- `WuXia.EnterLocationContext`
- `WuXia.StopSceneContext`
- `WuXia.UsePigeonLetterContext`

`ContextObjectType` must use the full type name.

### 3.4 `EventResult` Format

The current version supports three formats:

- `"Main0201"`
- `"Type;Name"`
- `"Type;Name;Arg"`

Event types include:

- `Simple`
- `Nani`
- `Location`
- `Scroll`
- `Battle`
- `Menu`
- `Highlight`
- `MiniGame`

When `EventResult` contains only one string segment, the game treats it as a Naninovel script name. In that case, `NaninovelScriptMaps` must provide a resource with the same name.

### 3.5 Fixed-Turn Trigger Events

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

This pattern is suitable for:

- Auto-playing a story scene on a specific turn
- Unlocking a location or system on a specific turn
- Opening new character interaction content on a specific turn

### 3.6 Enter a Script on Character Interaction

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

This pattern is suitable for:

- Generic location chatter for a character
- Overriding the default script during interaction
- Routing the same character into different scripts under different conditions

### 3.7 Side Effects Only

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

This pattern is suitable for:

- Refreshing Quests
- Switching states
- Calling side-effect expressions at specific times

### 3.8 Notes

- Fill in `EventResult` when you need to play a story script or enter a scene.
- Fill in `ActionResultExpression` or `ActionResultExpressions` when you only need to run logic expressions.
- Final priority equals `EventGroup PriorityBase + Tracker.Priority`.

## 4. Quest Mods

### 4.1 Minimum Quest File Structure

```json
{
  "ObjectType": "WuXia.Quest.Quest",
  "Name": "QuestName",
  "Type": "Main"
}
```

Available `Type` values:

- `Main`
- `Side`
- `WuXiangGe`
- `Hidden`

They directly affect display and runtime behavior:

- `Main`: higher priority in the main story flow.
- `WuXiangGe`: participates in Wuxiang Pavilion refresh, acceptance, and expiration flow.
- `Hidden`: hides Quest prompts and does not appear in the Pigeon Post Quest list.

### 4.2 Top-Level Fields Read in the Current Version

The current version reads these top-level fields from a Quest file:

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

### 4.3 Quest Localization Keys

The Quest UI reads localization text with these keys:

Quest name:

- `Quest.{questName}`

Quest description:

- `Quest.{questName}.Description.{DescriptionIndex}`

Quest type:

- `Quest.Type.{Type}`

Quest Pigeon Post:

- When `Index <= 0`:
  - `Quest.{questName}.Opening`
  - `Quest.{questName}.Context`
  - `Quest.{questName}.Sign`
- When `Index > 0`:
  - `Quest.{questName}.{Index}.Opening`
  - `Quest.{questName}.{Index}.Context`
  - `Quest.{questName}.{Index}.Sign`

The localization file pointed to by `LocalizationMaps` has this format:

```json
{
  "Tables": [
    {
      "Table": "Quest",
      "Entries": [
        {
          "Key": "Quest.HelpCollectHerbs",
          "Value": "Help Collect Herbs"
        },
        {
          "Key": "Quest.HelpCollectHerbs.Description.1",
          "Value": "Talk to FromCharacte"
        }
      ]
    }
  ]
}
```

Fill `Table` with the name of the string table you want to write into, and organize each `Key` in `Entries` using the rules above.

### 4.4 Progression Rules for Normal Quests

When `ComplexStages` is empty, the Quest merges these stage arrays into one unified flow:

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

After merging, the game sorts and advances them by `(Status, SubStatus)`.

When arranging a normal Quest, place every stage on a unique `(Status, SubStatus)`. `quest.Forward()` moves to the next stage in that sorted order.

### 4.5 Common Quest Structures

#### Turn Trigger -> Continue on Entering a Location -> Finish

```json
{
  "ObjectType": "WuXia.Quest.Quest",
  "Name": "QuestName1",
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

Use this structure for:

- A Quest entry appearing on a target turn
- Continuing the story after entering a target location
- Advancing to finished status in the script

#### Accept from a Character -> Turn In When Condition Is Met -> Finish

```json
{
  "ObjectType": "WuXia.Quest.Quest",
  "Name": "HelpCollectHerbs",
  "Type": "Side",
  "InteractableCharacterStages": [
    {
      "Uuid": "7fc1de6d-2c89-4b51-93b2-b2204912cfa0",
      "Status": "Unknown",
      "SubStatus": 0,
      "CharacterFullName": "Character.FromCharacter",
      "RelatedLocationFullName": "Map.书院.药园",
      "EventResult": "Quest/HelpCollectHerbs.接取"
    },
    {
      "Uuid": "fd729863-20a0-4d24-b843-88e8070a9f7b",
      "Status": "Ongoing",
      "SubStatus": 1,
      "CharacterFullName": "Character.FromCharacter",
      "RelatedLocationFullName": "Map.书院.药园",
      "Condition": "GameManager.Instance.Player.Bag.HasItem(\"Item.Material.白芷\", 3)",
      "EventResult": "Quest/HelpCollectHerbs.Finish"
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

#### Trigger by Pigeon Post -> Meet Someone -> Finish

```json
{
  "ObjectType": "WuXia.Quest.Quest",
  "Name": "FirstLetter",
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
      "CharacterFullName": "Character.FromCharacter",
      "RelatedLocationFullName": "Map.开封.潘楼街",
      "EventResult": "Quest/FirstLetter"
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

### 4.6 Behavior of Each Stage Type

#### `WaitTurnStartStage`

- Listens at turn start.
- Only triggers when `CurrentTurn == Turn`.
- `Condition` is checked together on that exact turn.
- Becomes invalid after it triggers.

#### `WaitTurnEndStage`

- Listens at turn end.
- Only triggers when `CurrentTurn == Turn`.
- Supports both `EventResult` and `ActionResultExpression`.

#### `InteractableCharacterStage`

- Listens on character interaction.
- Use `CharacterFullName` for one target character.
- Use `Characters` for multiple target characters.
- `RelatedLocationFullName` is used for Quest hints.
- `AddToLocation` can place the character into the location when entering the stage.
- When `Repeatable = false`, it becomes invalid after the first trigger.

Common fields:

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

- Listens on entering a location.
- Uses `LocationFullName` for exact full-name matching.
- Supports `Condition`.
- Becomes invalid after it triggers.

#### `PigeonLetterStage`

- This stage itself does not register a Tracker.
- While the current stage is this stage and `Condition` is true, the Quest Pigeon Post is shown in the UI.
- After the player reads it, the Quest immediately runs `quest.Forward()`.
- `Index` decides whether the Pigeon Post localization key includes a numeric suffix.

Note:

- A `PigeonLetterStage` must be followed by a reachable next stage.

#### `RequestItemsStage`

- `RequestItems` defines the turn-in requirements.
- The Quest checks whether the player has enough items.
- When leaving this stage, if the Quest does not end by expiration, the game automatically removes the submitted items.

#### `CountStatisticStage`

- Records the starting statistic value when entering the stage.
- Completion condition is `current statistic >= starting statistic + Count`.

#### `TrackAttributeStage`

- The target value is not absolute. It is `month-start snapshot + Value`.
- The stage completes when the current attribute reaches that target.
- The Quest description can use these variables:
  - `currentValue`
  - `monthStartValue`
  - `targetValue`
  - `value`

Note:

- `TrackAttributeStage` describes “growth gained this month”.

#### `ReachAttributeStage`

- Checks whether the current absolute attribute value has reached `Value`.

#### `WuXiangGeStage`

- Defines the group and time window used when the Quest participates in Wuxiang Pavilion refresh.
- After being selected, the Quest enters `Available`.
- After acceptance, `ExpiredTurn` is set to the end of the current month.

#### `PassStage`

- Once entered, it can be treated as immediately meeting the completion condition.

### 4.7 Complex Quests with `ComplexStages`

When a Quest uses `ComplexStages`, it switches to the complex Quest flow:

- The normal stage arrays no longer act as the main progression chain.
- Active stages are determined by `CurrentStages`.
- Complex stage completion depends on matching conditions or explicit script-driven exit.

The current version supports these complex stage subtypes:

- `Stage`
- `InteractableCharacterStage`
- `EnterLocationStage`
- `WaitTurnStartStage`
- `WaitTurnEndStage`

Common fields used by complex Quests:

- `AllConditions`: all of these stage states must appear
- `AnyConditions`: at least one of these stage states must appear
- `StatusToFinishs`: when the current complex stage finishes, these stages are also marked finished

### 4.8 Quest Script Progression Rules

Event and Quest JSON place the entry point at the correct time, location, or character. After the story ends, the Quest status should be advanced explicitly in the script.

For normal Quests, use:

- `quest.Update(...)`
- `quest.Forward()`
- `quest.Forward(true)`

For complex Quests, use:

- `quest.LeaveComplexStage(...)`

When a script name is referenced through `EventResult`, that script file must be provided through `NaninovelScriptMaps` with the same `FullName`.

### 4.9 Notes

- In a normal Quest, place only one stage on each `(Status, SubStatus)`.
- `DescriptionIndex` is the Quest log entry number, not “the only description sentence for the current stage”.
- Both `LocationFullName` and `RelatedLocationFullName` should use the full location name.

## 5. Minimal Production Workflow

### 5.1 Add a New Event

1. Prepare the Event group config file and name its logical key as `Event/{groupName}/{fileName}`.
2. Register that Event group file in `ConfigMaps` inside the manifest.
3. Prepare a patch for `Event/Event` and add `{fileName}` into the matching group's `List`. If the group does not exist, append a new group object first.
4. If the Event triggers a Naninovel script, provide that script in `NaninovelScriptMaps`.

### 5.2 Add a New Quest

1. Prepare the Quest config file and name its logical key as `Quest/{groupName}/{questName}`.
2. Register that Quest file in `ConfigMaps` inside the manifest.
3. Prepare a patch for `Quests` and add `{questName}` into the matching group's `QuestNames`. If the group does not exist, append a new group object first.
4. Add the needed Naninovel scripts and localization text for the Quest.
5. Advance the status explicitly in the Quest script.

### 5.3 Override an Existing Event or Quest

1. Use the existing logical name in `ConfigMaps`.
2. Provide the new full config content.
3. If the registration name does not change, you do not need to patch the master list.

## 6. Pre-Submit Checklist

- Every `FullName` in the manifest matches the logical name actually read by the game.
- New Events have been connected into the master list through a patch to `Event/Event`.
- New Quests have been connected into the master list through a patch to `Quests`.
- All `Uuid` values are legal and unique.
- Quest localization keys are complete.
- Quest Pigeon Post uses the correct key naming rules.
- Every script pointed to by `EventResult` has been provided through `NaninovelScriptMaps`.
- Quest scripts advance status explicitly.