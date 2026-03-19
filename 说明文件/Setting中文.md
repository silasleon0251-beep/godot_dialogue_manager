# 对话管理器设置（中文版）
对话管理器（Dialogue Manager）的设置可在「项目设置（Project Settings）」中「常规（General）」标签页的最下方找到。

## 运行时（Runtime）

### 状态自动加载快捷方式（State Autoload Shortcuts）

一个自动加载（Autoload）名称数组，用于在对话中快速访问其属性。例如，若你有一个名为 `SomeGlobal` 的自动加载节点，且该节点包含 `some_property` 属性，你可以在对话中这样引用它：

```
if SomeGlobal.some_property > 0:
  Nathan: 有 {{SomeGlobal.some_property}} 个！
```

但如果你将 "SomeGlobal" 添加到「状态自动加载快捷方式」列表中，那么在对话里就可以简化为：

```
if some_property > 0:
  Nathan: 有 {{some_property}} 个！
```

### 警告方法、属性或信号名称冲突（Warn about method property or signal name conflicts）（高级选项）

若启用此选项，当顶层作用域（即额外游戏状态、当前场景或自动加载快捷方式）中存在多个同名的属性、方法或信号时，调试器面板（Debugger panel）会显示警告信息。

*注：即便启用该选项，在非调试模式构建（non-debug build）的运行环境下，此功能也不会生效。*

### 对话气泡路径（Balloon Path）

调用 `DialogueManager.show_dialogue_balloon` 方法时，要实例化的对话气泡场景路径。

### 忽略缺失的状态值（Ignore Missing State Values）（高级选项）

当状态中缺少指定的属性或变更逻辑时，抑制相关错误提示。

## 编辑器（Editor）

### 长行自动换行（Wrap Long Lines）

在对话编辑器中对超长行进行自动换行，而非横向滚动。

### 新建文件模板（New File Template）

新建对话文件时，默认使用此模板内容初始化文件。

### 缺失翻译视为错误（Missing Translations Are Errors）

所有未设置静态 ID 的对话行都会被标记为错误。

### 将角色名纳入可翻译字符串列表（Include Characters in Translatable Strings List）

导出 POT 翻译文件时，将所有角色名称包含在内。

### 默认 CSV 语言区域（Default Csv Locale）

首次导出翻译 CSV 文件时使用的默认语言区域。

### 在翻译导出中包含角色信息（Include Character in Translation Exports）（高级选项）

导出 CSV 文件时，新增一列名为 `_character` 的列，用于标注该行对话的说话角色。

### 在翻译导出中包含备注（Include Notes in Translation Exports）（高级选项）

导出 CSV 文件时，新增一列名为 `_notes` 的列，用于存放文档注释（doc comments）内容。

### 对话处理器路径（Dialogue Processor Path）（高级选项）

指向一个继承自 `DMDialogueProcessor` 的类路径。该类用于对原始对话行进行预处理，以及对编译后的对话行进行后处理。

### 自定义测试场景路径（Custom Test Scene Path）（高级选项）

在对话编辑器中执行「测试（Test）」功能时，使用指定的自定义测试场景。该场景必须继承自 `BaseDialogueTestScene` 类。

### 额外自动补全脚本源（Extra Auto Complete Script Sources）（高级选项）

添加需要参与顶层自动补全成员检测的脚本文件。
此处添加的所有脚本，都会被视为在所有对话文件中均可用（例如：你的气泡节点会在运行时自动插入到 `extra_game_states` 中）。
