# 条件与状态修改

## 条件

### If/else

你可以使用条件块来进一步分支对话。以 `if` 开头编写条件行，然后写上表达式。你可以对变量或函数结果进行比较。

你可以使用 `elif` 来编写附加条件，还可以使用 `else` 来处理其他所有情况。

```
if SomeGlobal.some_property >= 10
    Nathan: 这个属性大于或等于 10
elif SomeGlobal.some_other_property == "some value"
    Nathan: 或者我们会走到这里。
else
    Nathan: 如果前面都不满足，我会说这句话。
```

_注意：如果你想转义条件行（比如想让对话行以 `if` 开头），可以在条件关键字前加 `\`。_

条件可以用 `and`/`or` 进行连接，还可以使用 `(`、`)` 分组。例如：

```
if something == "a value" and (score < 0 or score> 100)
    Nathan: 这个条件成立了！
```

选项也可以带 `if` 条件。将这些条件用 `[` 和 `/]` 包裹（有时称为`“自闭合”`）。

```
Nathan: 你想要什么？
- 这个 [if SomeGlobal.some_property == 0 or SomeGlobal.some_other_property == false /]
    Nathan: 啊，你想要这个？
- 另一个 [if SomeGlobal.some_method () /] => another_label
- 什么都不要 => END
```

如果在选项行同时使用`条件`和`跳转`，要确保跳转写在最后。

条件语句也可内嵌在对话行中使用，只需将其用 `[if 表达式]` 和 `[/if]`"包裹即可。

```
晓轩：这道题你[if has_redo]又重新做了[/if]，值得表扬！

# 如果 has_redo = true， 输出："晓轩：这道题你又重新做了，值得表扬！"
# 如果 has_redo = false，输出："晓轩：这道题你，值得表扬！"

```

对于简单的二选一条件，你可以这样写：

```
晓轩：我今天喝了 {{num_water}} 杯[if num_water == 1]水[else]水啦[/else]，超解渴！

# 如果num_water = 1，输出："晓轩：我今天喝了 1 杯水，超解渴！"
# 如果num_water = 3，输出："晓轩：我今天喝了 3 杯水啦，超解渴！"

```

随机行和随机跳转行也可以带条件。随机行的条件写在 `%` 符号后的方括号 `[]` 内，且要放在该行内容之前：

```
% => some_label
%2 => some_other_label
% [if SomeGlobal.some_condition] => another_label
```

### Match

为了简化一长串 if/elif/elif/elif，你可以使用 `match` 行：

```
match SomeGlobal.some_property
    when 1
        Nathan: 是 1。
    when > 5
        Nathan: 大于 5（但不是 1）。
    else
        Nathan: 是其他值。
```

### While

你也可以用 `while` 来开启一个条件语句块。这类语句块会在条件为真的期间持续循环执行。

```
while SomeGlobal.some_property < 10
    Nathan: 属性仍然小于 10 —— 当前是 {{SomeGlobal.some_property}}。
    do SomeGlobal.some_property += 1
Nathan: 现在我们可以继续了。
```

## 状态修改（Mutations）

你可以用 `set` 或 `do` 行来修改状态。

```
if SomeGlobal.has_met_nathan == false
    do SomeGlobal.animate ("Nathan", "Wave")
    Nathan: 你好，我是 Nathan。
    set SomeGlobal.has_met_nathan = true
Nathan: 我能为你做什么？
- 给我讲讲这个对话编辑器
```

在上面的例子中，对话管理器会要求一个名为 `SomeGlobal` 的全局对象实现一个方法，该方法的签名为 `func animate(string, string) -> void` 的方法。

状态修改也可以内嵌在使用对话行中使用。内嵌修改会在文字逐字显示到对应位置时执行。

```
Nathan: 我不确定我们见过 [do wave ()] 我是 Nathan。
Nathan: 我还能内联发送信号 [do SomeGlobal.some_signal.emit ()]。
```

如果内嵌修改的实现里用了 `await`，对话显示会暂停直到它完成。若要跳过等待（不阻塞文本显示），可以在 `do` 后面加 `!`，例如 `[do! something()]`。

### 额外游戏状态（Extra Game States）

在[请求一行对话](API中文.md#func-get_next_dialogue_lineresource-resource-key-string--0-extra_game_states-array-----dictionary)时，你可以将一个包含节点 / 对象 / 字典的数组作为 `extra_game_states` 参数传入，该参数对应的对象也会被检查是否包含可用的变更操作方法。类必须实例化，即使内容是静态的。程序会按照传入的顺序遍历这些对象，查找匹配的属性。示例：

```
func pirate():
    print("yarrr")

class GameStateClass:
    var pirate_name = "phil"
    func hello():
        print("ahoy")
    
func _ready() -> void:
    # 一定要写 GameStateClass.new(), 不能只写 GameStateClass !
    DialogueManager.show_example_dialogue_balloon(load("res://main.dialogue"), "start", [self, { "game" = GameStateClass.new() }]) 
```

在对话结束时，你可以直接调用传入对象的方法（例如 `self.pirate()` 也可以写成 `do pirate()`），也可以通过解引用成员变量或字典键的方式调用（例如 `do game.hello()`）：

```
~ start
do game.hello()
do pirate()
set game.pirate_name = "delilah"
do debug(game.pirate_name)
=> END
```

额外游戏状态是对话系统外部存在的对象、节点或字典的引用。对它们属性的修改会在对话结束后保留。
与局部变量的主要区别：额外游戏状态可通过名称直接访问（如 `level`），而用 `set locals.variable_name = value` 创建的变量必须始终带 `locals.` 前缀（如 `locals.counter`）。

### 信号（Signals）

信号的触发方式与它在 GDScript 中的触发方式类似 —— 通过对信号调用 `emit` 方法来实现

例如，若 `SomeGlobal` 这个全局对象有一个名为 `some_signal` 的信号，且该信号包含一个字符串类型的参数，那么你可以在对话脚本中这样触发它：

```
do SomeGlobal.some_signal.emit("some argument")
```

### 空值合并（Null coalescing）

在某些情况下，你可能需要引用一个可能已定义、也可能未定义的对象属性。这种场景下，你可以使用空值合并（null coalescing） `?.`语法：

```
if some_node_reference?.name == "SomeNode"
    Nathan: 注意到 ?. 语法了吗？
```

如果 `some_node_reference` 的值为 `null`，那么这个比较表达式的整个左侧部分都会是 `null`，因此它不会等于 "SomeNode"，比较结果会失败。如果此处不使用空值合并语法，且 `some_node_reference` 为 `null`，那么游戏将会崩溃。

### 状态简写（State shortcuts）

如果你想把 `SomeGlobal.some_property` 简写为 `some_property`，有两种方式：

1. 如果所有对话都用同一个状态，可以在[设置](./Settings.md)里配置全局状态简写。
2. 或者，如果你想每个对话文件用不同简写，可以在文件顶部加一行 `using SomeGlobal`（对应你使用的自动加载对象）。

## 特殊变量 / 状态修改（Special variables/mutations）

有几个内置的特殊状态修改可以使用：

- `do wait(float)` —— 等待指定秒数（该操作在内嵌对话时使用无任何效果）。
- `do debug(...)` —— 把内容打印到输出窗口。

还有一个特殊属性 `self`，可以在对话里表示当前正在运行的 `DialogueResource`。