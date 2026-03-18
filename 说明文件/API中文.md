# API

## `DialogueManager`

### 信号（Signals）

- `dialogue_started(resource: DialogueResource)` - 当 `DialogueManager` 创建对话气泡并开始对话时触发。
- `passed_label(label: String)` - 当经过标签标记点时触发。
- `got_dialogue(line: DialogueLine)` - 当找到一条对话行时触发。
- `mutated(mutation: Dictionary)` - 当即将执行一条突变指令时触发（不包含 `set` 行）。
- `dialogue_ended(resource: DialogueResource)` - 当下一行对话为空时触发，并返回调用的资源。

### 方法（Methods）

#### `func show_dialogue_balloon(resource: DialogueResource, label: String = "", extra_game_states: Array = []) -> Node`

打开在设置中配置的对话气泡（若未设置，则使用示例气泡）。

返回气泡的根节点，方便你手动调用 `queue_free()`。

#### `func show_dialogue_balloon_scene(balloon_scene: Node | String, resource: DialogueResource, label: String = "", extra_game_states: Array = []) -> Node`

打开由 `balloon_scene` 指定的对话气泡场景。

返回气泡的根节点，方便你手动调用 `queue_free()`。

#### `func get_next_dialogue_line(resource: DialogueResource, key: String = "", extra_game_states: Array = [], mutation_behaviour: MutationBehaviour = MutationBehaviour.Wait) -> DialogueLine`

**必须配合 `await` 使用。**

传入资源与标签/ID，会查找下一条可显示的对话行（在运行过程中会执行突变操作）。

返回一个 `DialogueLine`，结构如下：

- `id: String` - 该行对话的 ID。
- `next_id: String` - 该行之后的下一行对话 ID。
- `character: String` - 说话角色名（为`""`则无角色）。
- `text: String` - 角色说的文本。
- `tags: PackedStringArray` - 标签列表。
- `translation_key: String` - 用于翻译的键（若未指定翻译ID，则直接使用整段文本本身作为键）。
- `responses: Array[DialogueResponse]` - 该对话的选项列表（为`[]`则无选项）。
  - `id: String` - 选项 ID。
  - `next_id: String` - 选择该选项后跳转的下一行 ID。
  - `is_allowed: bool` - 该选项是否通过条件检测。
  - `condition_as_text: String` - 用于判断是否显示该选项的原始条件（字符串形式）。
  - `character: String` - 选项对应的角色名（为`""`则无角色）。
  - `text: String` - 选项文本。
  - `tags: PackedStringArray` - 选项标签列表。
  - `translation_key: String` - 选项翻译键（若未指定翻译ID，则直接使用整段文本本身作为键）。
- `concurrent_lines: Array[DialogueLine]` - 与当前行同时显示的对话行列表。

若未找到下一行对话，将返回空字典 `{}`。

传入节点数组 `extra_game_states`，可临时添加到游戏状态快捷方式，供条件与突变使用。

你可以将 `mutation_behaviour`指定为 `DialogueManager.MutationBehaviour` 枚举中提供的任一值：
- `Wait`（默认）：等待所有突变(mutation)行执行完毕。
- `DoNoWait`：执行突变但不等待，直接进入下一行。
- `Skip`：完全跳过突变。

大多数情况下，你应使用默认值。**示例气泡仅支持 `Wait`。**

#### `func show_example_dialogue_balloon(resource: DialogueResource, label: String = "", extra_game_states: Array = []) -> CanvasLayer`

打开示例对话气泡。

如果游戏视口小于 400，则打开低分辨率气泡，否则打开普通气泡。

对话结束后会自动关闭。

返回示例气泡的根 `CanvasLayer`，方便你手动调用 `queue_free()`。

---

## `DialogueLabel`

### 导出属性（Exports）

- `seconds_per_step: float = 0.02` - 文字逐字显示的速度（单位：秒 / 步）。
- `pause_at_characters: String = ".?!"` - 遇到这些字符`.?!`时自动短暂停顿（英文符号`半角`）。
- `skip_pause_at_character_if_followed_by: String = ")\""` - 若停顿字符后紧跟这些字符`)\"`，则忽略自动停顿。
- `skip_pause_at_abbreviations: Array = ["Mr", "Mrs", "Ms", "Dr", "etc", "ex"]` - 在这些缩写后不自动停顿（仅当 `.` 在 `pause_at_characters` 中时有效）。
- `seconds_per_pause_step: float = 0.3` - 遇到停顿字符时的停顿时长（单位：秒）。

### 信号（Signals）

- `spoke(letter: String, letter_index: int, speed: float)` - 文字逐字显示过程中每一步触发。
- `started_typing()` - 标签开始打字时触发。
- `skipped_typing()` - 玩家跳过打字动画时触发。
- `finished_typing()` - 标签完成打字时触发。

### 方法（Methods）

#### `func type_out() -> void`

开始逐字显示标签文本。

#### `func skip_typing() -> void`

停止打字动画，直接显示完整文本。会触发 `skipped_typing`。