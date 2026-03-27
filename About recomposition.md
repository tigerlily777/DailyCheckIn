## About recomposition

### Usecase: list - listitem 
If my app's home page is a list, each list item takes user to cooresponding details page. There's a button on details page, which will show a bottom sheet on clicking.
When I click any list item to nav to details screen, and nav back by clicking back button on details screen, does recomposation happen?


🧠 What is Recomposition? 

Imaging the UI is an intelligent white board.
- first drawing -> first time compose
- update some details -> the board won't draw the whole picture again, instead, only a small piece with details will be drew.


👉 "Only draw some parts of the picture" --> is called Recomposition

⸻

✅ In one word:

👉 Recomposition = UI Due to the **change of state**, some Composable are running again, and the UI is getting updated.

不是“页面刷新”，不是“重新加载”，而是：

✨ Only re-draw the part that needs updating

🔥 Back to the usecase above

List Page → click item → Details Page
Details Page → click back → back to List Page

❓Question：

👉 When user back to the list page，does recomposition happen？

🎯 It depends!


⸻

🧩 Scenario 1：List is still **alive**

If you're using
- Navigation Compose
- Or Fragment + ComposeView

Then:

👉 List screen is not getting destoyed. It's just running at the background, like "minimised"

⸻

Then what happend when navigating back?

✔ If state has **NOT** changed
👉 Nothing will happen. Just reuse the UI

✔ If state **HAS** changed（data refreshing）
👉 Recomposition will happen

⸻

✅ So the answer is：

👉 “back to the list page ≠ recomposition”

⸻

🧩 Scenario 2：List page gets destroyed

e.g：
- popBackStack 彻底清掉
- Or Activity is recreated（rotate screen / memory recycled）

Then：

👉 When coming back：❗trigger another Composition（NOT Recomposition）

⸻

🎯 How about clicking a button to show bottomsheet on details screen

```kotlin
var showSheet by remember { mutableStateOf(false) }

if (showSheet) {
    ModalBottomSheet(...)
}
```
👉 When clicking a button ```showSheet: false → true```

💥 What happened?

👉 Recomposition triggered
👉 if (showSheet) block has been executed again
👉 BottomSheet shows


🧠 An important signal

❓Any state change?

- Yes 👉 Recomposition ✅ 
- NO 👉 ❌ Recomposition

💡 In summury

👉 List → Details → Back：
	•	❌ Not a recomposition
	•	❌ Not a page refresh
	•	✅ Navigation 切换 + 可能复用 UI

👉 BottomSheet：
	•	✅ A typical scenario of Recomposition -> state changes

🚀 **Recomposition > 🔁 Composable functions are running again（only the necessary parts**

不是整棵 UI 树，而是：👉 “精确打击”

⸻

## About state 

👉 state = 能影响 UI 显示的数据 (只要这个数据一变，UI 就可能要变)

⸻

🎯 但重点来了：不是所有数据都是 state

✅ 所有被 Compose“观察”的数据，才是 state

🧩 What does it mean?

🥇 1 ：Compose “看得见”的 state（核心）

只有这些，才会触发 recomposition：

1️⃣ mutableStateOf ```var text by remember { mutableStateOf("") }```
👉 改 text
💥 一定 recomposition

2️⃣ StateFlow / Flow（via collect）```val uiState by viewModel.uiState.collectAsState()```
👉 Flow emit 新值
💥 recomposition

3️⃣ LiveData ```val data by viewModel.liveData.observeAsState()```
👉 数据变化
💥 recomposition

✅ 只有“被 collect / observe 的数据”，才算 Compose 的 state

⸻

🧠 2 ：How about the normal vars？ ```var count = 0```

❌ recomposition
👉 因为 Compose 根本“不知道它变了”

⸻

👉 这就像：
	•	state = “被摄像头盯着的人”
	•	普通变量 = “躲在角落的人”

⸻

🔥 回答你核心问题

❓“state 是谁的 state？”

👉 不是 ViewModel 的
👉 也不是 Compose 的

👉 是“UI 正在使用的那份数据”

👇
```
@Composable
fun DetailScreen(viewModel: VM) {
    val uiState by viewModel.uiState.collectAsState()

    Text(uiState.name)
}
```
👉 这里的 state 是：```uiState```, 不是 ViewModel 本身
而是：✨ UI 正在“读”的这份数据

🎯 How does recomposition get triggerred？

👉 📡 Compose 会“订阅”你读取的 state

⸻

🧠 Imaging：

When you write ```Text(uiState.name)```
系统偷偷做了这件事：“我盯着 uiState.name，一旦它变，我就重新执行这段 UI”

🔥 关键问题来了：

❓我怎么“控制 state 变没变”？

👉 本质是：**你控制“有没有发出新值”**

⸻

🧩 Another example

❌ 错误写法（不会触发 recomposition）
```
data class UiState(val count: Int)

uiState.count += 1
```
👉 为什么不行？

因为：**👉 对象没变，只是里面的值变了**

Compose 看的是：这个“盒子”有没有换,不是里面的东西
✅ 正确写法
_uiState.value = uiState.copy(count = uiState.count + 1)
👉 这次：**💥 新对象 → state 变化 → recomposition**
🧠 一个非常关键的直觉（你要建立这个）

👉 Compose 判断的是：**❗“引用有没有变”（是不是新对象）**

⸻

🧩 再用一个生活类比（保证你记住）

State is a parcel：📦 

- If you change the thing inside of the box 👉 Compose：🙈 doesn't know
- If you change another parcel box 👉 Compose：👀 YES I can tell it changed! Updating UI!

⸻

🎯 Back to the original usecase

List → Details → Back

👉 recomposition depends on：✔ whether the state of list has changed!

比如：val list by viewModel.list.collectAsState()

情况 1：数据没变

👉 不 recomposition

情况 2：你在 Details 改了数据 viewModel.updateItem(...)
👉 Flow emit 新值
💥 List recomposition

3. 最后一层理解

👉 **Compose 本质是：🧠 “读取 state → 记录依赖 → state 变 → 重新执行”**

## The lifecycle of viewmodel 
🧠 1️⃣ ViewModel - when alive/die?

✅ ViewModel lifecycle ≠ Composable lifecycle
✅ It binds to ViewModelStoreOwner

⸻

🎯 Who's ViewModelStoreOwner？

3 common ones：
	•	Activity
	•	Fragment
	•	**Navigation BackStackEntry** 

⸻

🧩 In Navigation Compose 
```
composable("list") {
    val vm: ListViewModel = viewModel()
}
```
👉 The ViewModel belongs to：

🧱 The current navigation entry（list page）

⸻

🔥 那什么时候会被销毁？

❗当这个 screen 从 back stack 被移除时

⸻

🧪 举几个你能感知的例子

✔ 正常来回跳 List → Details → Back
👉 List 还在 back stack 里
👉 ViewModel ✅ 还活着

❌ 被 pop 掉 navController.popBackStack("list", inclusive = true)
👉 List 被干掉
👉 ViewModel 💀 destroyed

❌ Kill the app（系统回收）
👉 ViewModel 💀 destroyed


🧠 一个超好记的类比
你可以把：
	•	Navigation back stack = 🏨 酒店房间
	•	ViewModel = 🧳 你放在房间里的行李

👉 房间还在 → 行李还在
👉 房间被退掉 → 行李没了

❗2️⃣ 你问的“scope 写错”是什么意思？

这个是很多人踩坑的地方，你问得非常好。
🎯 正确方式（推荐）val vm: ListViewModel = viewModel()

👉 默认 scope：当前 screen（NavBackStackEntry）

⸻

⚠️ 错误 / 容易踩坑的方式

❶ 每个 screen 都拿自己的 VM（但你以为是共享的）
// ListScreen
val vm: MyVM = viewModel()

// DetailScreen
val vm: MyVM = viewModel()

👉 ❗这是两个不同的 ViewModel！

👉 结果：
	•	List 请求了一次 API
	•	Detail 又请求一次

💥 重复请求

⸻

❷ 应该共享但没共享（真正的“scope 写错”）

比如你其实想：

👉 List 和 Detail 共享数据

但你没这样写：
```
val parentEntry = remember(navController) {
    navController.getBackStackEntry("list")
}
val vm: MyVM = viewModel(parentEntry)
```
👉 结果就是：

❗ViewModel scope 选错了 → 生命周期不对 → 行为异常

🔥 3️⃣ **init vs LaunchedEffect**

你观察得非常细：

init -> in viewmodel
LaunchedEffect -> in Composable 

👉 ✔ 完全正确
🎯 Best approach

```
class ListViewModel : ViewModel() {
    init {
        loadData()
    }
}
```
👉 Why?

✅ bind data lifecycle to ViewModel 

⚠️ Not so good
```
LaunchedEffect(Unit) {
    vm.loadData()
}
```
👉 WHY?

⸻

💣 这个坑到底在哪？（重点）

👉 Composable 不是只执行一次！

它可能会：
	•	重新进入 Composition
	•	被销毁再创建
	•	navigation 切回来时重新执行
🧪 举个真实会发生的情况

场景：List → Details → Back
如果 ListScreen 被重新 compose：

👉 LaunchedEffect(Unit) 可能会再执行一次

💥 → 再 call API
❗更隐蔽的 bug

1️⃣ 屏幕旋转

👉 Activity 重建
👉 Composable 重新执行
👉 LaunchedEffect 再跑

💥 再请求一次 API
2️⃣ 参数变化
LaunchedEffect(userId)
👉 userId 变了
💥 再请求

⸻

🎯 所以核心原则是：

❗UI 不负责决定“什么时候加载数据”

👉 UI 只负责：

🖼️ 展示数据

⸻

👉 数据什么时候加载，由谁决定？
✅ ViewModel

🧠 再用一个类比（你会一下记住）
	•	ViewModel = 🍳 厨师
	•	Composable = 🍽️ 服务员

⸻

👉 正确方式：

🍳 厨师决定什么时候做菜
🍽️ 服务员只是端上来

⸻

👉 错误方式（你现在的）：

🍽️ 服务员说：“我觉得你该做菜了！”

💥 每来一个客人（recomposition）就喊一次

## Tests

🎯 规则（很重要）

每一题你回答：
1️⃣ 会不会发生 recomposition？
2️⃣ 哪一部分会？
3️⃣ 为什么？

🧪 第 1 题（热身）
```
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }

    Column {
        Text("Count: $count")
        Button(onClick = { count++ }) {
            Text("Add")
        }
    }
}
```
👉 点击按钮一次

🧪 第 2 题
```
@Composable
fun Counter() {
    var count = 0

    Column {
        Text("Count: $count")
        Button(onClick = { count++ }) {
            Text("Add")
        }
    }
}
```
👉 点击按钮一次

⸻

🧪 第 3 题（关键理解：谁在读 state）
```
@Composable
fun Parent() {
    var count by remember { mutableStateOf(0) }

    Column {
        Button(onClick = { count++ }) {
            Text("Add")
        }
        Child()
    }
}

@Composable
fun Child() {
    Text("Hello")
}
```
👉 点击按钮一次

⸻

🧪 第 4 题
```
@Composable
fun Parent() {
    var count by remember { mutableStateOf(0) }

    Column {
        Button(onClick = { count++ }) {
            Text("Add")
        }
        Child(count)
    }
}

@Composable
fun Child(count: Int) {
    Text("Count: $count")
}
```
👉 点击按钮一次

⸻

🧪 第 5 题（面试高频坑🔥）
```
data class UiState(val count: Int)

@Composable
fun Screen() {
    var state by remember { mutableStateOf(UiState(0)) }

    Column {
        Text("Count: ${state.count}")
        Button(onClick = {
            state.count + 1
        }) {
            Text("Add")
        }
    }
}
```
👉 点击按钮一次

### Answers

第 1 题 ✅
	•	你的回答完全正确
	•	💡 补充一点：只有 mutableStateOf 才会被 Compose 观察；remember + mutableStateOf 才能保证 state 在 recomposition 之间存活

⸻

第 2 题 ✅
	•	完全正确
	•	💡 核心就是：普通变量不会被 Compose 观察
	•	所以点击按钮，count++ 改了值，但 Composable 并不知道 → 不会重新执行

⸻

第 3 题 ✅
	•	对，你抓住了重点：只有 Parent 观察了 count
	•	Child 没有任何依赖，所以不会被重执行
	•	💡 这就是 Compose 的精确打击：只重绘用到 state 的部分

⸻

第 4 题 ✅
	•	完全正确
	•	补充：Child 也会被 recomposition，因为它是 Child(count)，参数 count 变了 → Child Composable 也被执行
	•	👉 这就是参数传递也会触发 recomposition 的典型场景

⸻

第 5 题 ✅
	•	正确！这就是“内部对象变了，但引用没变”的坑
	•	💡 补充：如果你想让这个 recomposition 发生，需要写成：
	state = state.copy(count = state.count + 1)
	•	这样 state 是新对象，Compose 才会感知到 → recomposition 发生



## Advanced
🎯 规则

我会给你一个完整的 screen，你回答：

1️⃣ 点击按钮 / navigate / back 时，会不会 recomposition
2️⃣ 会不会重新请求 API
3️⃣ 为什么

你尽量像之前那样写出理由，
```
data class UiState(val items: List<String> = emptyList())

class ListViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(UiState())
    val uiState: StateFlow<UiState> = _uiState

    init {
        loadData()
    }

    fun loadData() {
        viewModelScope.launch {
            delay(1000) // 模拟网络请求
            _uiState.value = UiState(listOf("A", "B", "C"))
        }
    }

    fun addItem(item: String) {
        _uiState.value = _uiState.value.copy(items = _uiState.value.items + item)
    }
}

@Composable
fun ListScreen(
    viewModel: ListViewModel = viewModel()
) {
    val uiState by viewModel.uiState.collectAsState()

    Column {
        uiState.items.forEach { item ->
            Text(item, modifier = Modifier.clickable {
                // Navigate to details
            })
        }
        Button(onClick = { viewModel.addItem("D") }) {
            Text("Add D")
        }
    }

    LaunchedEffect(Unit) {
        println("LaunchedEffect called")
    }
}
```

| Usecase 1 |  recomposition? | loadData() called | LaunchedEffect |
|---|---|---|---|
| Launch app -> First time show list screen | ✅ First time running compose, not RE but compose | ✅ | ✅ _uiState.value 从 UiState(emptyList()) → UiState(listOf("A","B","C")) |
| Click on Add D| ✅ _uiState.value 被 addItem() 改了，Compose 观察到 StateFlow emit 新值 → 依赖它的 Composable 重新执行  | ❌ viewmodel is still alive | ❌ because of unit
| Click on a list item to nav to details screen | ❌ List screen is still in backstack, and no state change | ❌ | ❌ omposable 没被销毁 → Unit 也没变 → LaunchedEffect 不执行
| Nav back to list screen | ❌ list screen is from back stack, no state change, no recompose UI | ❌ Viewmodel still alive | ❌ LaunchedEffect 只会在 Composable 首次 Composition 时执行，返回 backstack 只是恢复显示，Composable 本身并没有被销毁 → Unit 没变 → LaunchedEffect 不会再执行

⸻

✅ **KEY WORDS**
	1.	Recomposition ≠ show screen
	•	页面显示/Navigation 切换，不代表 recomposition，只有 state 改变才触发。
	2.	ViewModel 生命周期决定 loadData 执行次数
	•	init 只在第一次创建时执行。
	3.	LaunchedEffect(Unit) 只执行一次
	•	只在首次 Composition 或 key 改变时执行。
	4.	StateFlow emit 新值 → 观察它的 Composable 会 recomposition
	•	即使 List 内部元素变化，也要用新对象 (copy) 才触发 recomposition。

Question:
state 从 empty list → 有内容
	•	_uiState.value 从 UiState(emptyList()) → UiState(listOf("A","B","C"))
这为什么是 state 的变化啊，这不是list里面的value发生变化吗
Answer:
🧠 核心概念

Compose 判断 state 是否变化，不是看里面的内容，而是看 “引用有没有变”。
	•	StateFlow / mutableStateOf 观察的是 整个对象的引用
	•	只要你给它赋了一个新对象 → Compose 会感知到 → 触发 recomposition
🔍 回到你的例子_uiState.value = UiState(listOf("A", "B", "C"))
关键点：
	1.	初始值：UiState(emptyList())
	2.	新值：UiState(listOf("A","B","C"))
	✅ 两者是 不同的对象（不同引用）

⸻

❌ 不是“List 内部值变了”
	•	如果你只是修改了 原来的 list：_uiState.value.items.add("A")

		•	这时候 _uiState.value 的引用没变
	•	Compose 不会感知 → 不会 recomposition

⸻

✅ 正确的触发方式
	•	创建 新对象（或者新 list）
	•	赋值给 _uiState.value
	•	Compose 观察到引用变 → recomposition 发生

	💡 类比

把 state 想象成一个“包裹盒子”：
	•	初始包裹：空 list
	•	新包裹：包含 ABC 的 list

不管你是不是把盒子里面的东西换了，只要盒子没换，Compose 看不到变化
只有换了一个盒子 → Compose 才知道“我需要重新画 UI”
所以，	
_uiState.value = UiState(listOf("A","B","C"))
•	UiState 对象是新的
	•	✅ 这就是 state 变化 → 会触发 recomposition
