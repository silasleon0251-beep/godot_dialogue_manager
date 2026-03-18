# 条件与状态修改

## 条件

### If/else

你可以使用条件块来进一步分支对话。以 `if` 开头编写条件行，然后写上表达式。你可以对变量或函数结果进行比较。

额外条件使用 `elif`，你可以使用 `else` 来处理其他所有情况。
if SomeGlobal.some_property >= 10
Nathan: 这个属性大于或等于 10
elif SomeGlobal.some_other_property == "some value"
Nathan: 或者我们会走到这里。
else
Nathan: 如果前面都不满足，我会说这句话。



> 注意：如果你想转义条件行（比如想让对话行以 `if` 开头），可以在条件关键字前加 `\`。

条件可以用 `and`/`or` 连接，并用 `(`、`)` 分组。例如：
if something == "a value" and (score < 0 or score> 100)
Nathan: 这个条件成立了！



选项也可以带 `if` 条件。用 `[` 和 `/]` 包裹（有时称为“自闭合”）。
Nathan: 你想要什么？
这个 [if SomeGlobal.some_property == 0 or SomeGlobal.some_other_property == false /]
Nathan: 啊，你想要这个？
另一个 [if SomeGlobal.some_method () /] => another_label
什么都不要 => END


如果在选项行同时使用条件和跳转，要确保跳转写在最后。

条件也可以内联在对话行中，用 `[if predicate]` 和 `[/if]` 包裹。
Nathan: 我 [if already_done] 又 [/if] 做过这件事了。


对于简单的二选一条件，你可以这样写：
Nathan: 你有 {{num_apples}} 个 [if num_apples == 1] 苹果 [else] 苹果 [/if]，不错！


随机行和随机跳转行也可以带条件。随机行的条件写在 `%` 后面、内容前面的方括号里：
% => some_label
%2 => some_other_label
% [if SomeGlobal.some_condition] => another_label


### Match

为了简化一长串 if/elif/elif/elif，你可以使用 `match` 行：
match SomeGlobal.some_property
when 1
Nathan: 是 1。
when > 5
Nathan: 大于 5（但不是 1）。
else
Nathan: 是其他值。


### While

你也可以用 `while` 开头写条件块。这些块会在条件为真时循环执行。
while SomeGlobal.some_property < 10
Nathan: 属性仍然小于 10 —— 当前是 {{SomeGlobal.some_property}}。
do SomeGlobal.some_property += 1
Nathan: 现在我们可以继续了。


## 状态修改

你可以用 `set` 或 `do` 行来修改状态。
if SomeGlobal.has_met_nathan == false
do SomeGlobal.animate ("Nathan", "Wave")
Nathan: 你好，我是 Nathan。
set SomeGlobal.has_met_nathan = true
Nathan: 我能为你做什么？
给我讲讲这个对话编辑器


在上面的例子中，对话管理器会期望有一个名为 `SomeGlobal` 的全局对象，实现签名为 `func animate(string, string) -> void` 的方法。

状态修改也可以内联使用。内联修改会在文字逐字显示到对应位置时执行。
Nathan: 我不确定我们见过 [do wave ()] 我是 Nathan。
Nathan: 我还能内联发送信号 [do SomeGlobal.some_signal.emit ()]。


如果内联修改的实现里用了 `await`，对话打字会暂停直到它完成。如果不想等待，可以在 `do` 后面加 `!`，例如 `[do! something()]`。

### 额外游戏状态

在[请求一行对话](API.md#func-get_next_dialogue_lineresource-resource-key-string--0-extra_game_states-array-----dictionary)时，你可以传入一个节点/对象/字典数组作为 `extra_game_states` 参数，系统也会在这些对象里查找可调用的修改方法。
类必须实例化，即使内容是静态的。系统会按传入顺序遍历对象，查找匹配的属性。示例：
func pirate():
print("yarrr")
class GameStateClass:
var pirate_name = "phil"
func hello():
print("ahoy")
func _ready() -> void:
要用 GameStateClass.new ()，不是 GameStateClass！
DialogueManager.show_example_dialogue_balloon(load("res://main.dialogue"), "start", [self, { "game" = GameStateClass.new() }])


在对话里，你可以直接调用传入对象的方法（例如 `self.pirate()` 可以写成 `do pirate()`），或通过成员/字典键调用（例如 `do game.hello()`）：
~ start
do game.hello()
do pirate()
set game.pirate_name = "delilah"
do debug(game.pirate_name)
=> END


额外游戏状态是对话系统外部存在的对象、节点或字典的引用。对它们属性的修改会在对话结束后保留。
与局部变量的主要区别：额外游戏状态可以直接按名称访问（如 `level`），而用 `set locals.variable_name = value` 创建的变量必须始终带 `locals.` 前缀（如 `locals.counter`）。

### 信号

发送信号的方式和 GDScript 类似 —— 对信号调用 `emit`。

例如，如果 `SomeGlobal` 有一个带单个字符串参数的信号 `some_signal`，你可以在对话里这样发送：
do SomeGlobal.some_signal.emit("some argument")


### 空值合并

有些情况下你需要访问可能为空的对象的属性。这时可以使用空值合并：
if some_node_reference?.name == "SomeNode"
Nathan: 注意到 ?. 语法了吗？


如果 `some_node_reference` 是 null，整个比较左边会是 null，因此不等于 "SomeNode"，条件不成立。
如果这里不使用空值合并，而 `some_node_reference` 为 null，游戏会崩溃。

### 状态简写

如果你想把 `SomeGlobal.some_property` 简写为 `some_property`，有两种方式：

1. 如果所有对话都用同一个状态，可以在[设置](./Settings.md)里配置全局状态简写。
2. 或者，如果你想每个对话文件用不同简写，可以在文件顶部加一行 `using SomeGlobal`（对应你使用的自动加载对象）。

## 特殊变量 / 状态修改

有几个内置的特殊状态修改可以使用：

- `do wait(float)` —— 等待指定秒数（内联使用时无效）。
- `do debug(...)` —— 把内容打印到输出窗口。

还有一个特殊属性 `self`，可以在对话里表示当前正在运行的 `DialogueResource`。