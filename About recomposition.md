## About recomposition

Usecase: list - listitem 
假如我的app有main page，是一个list，每个list item点一下会navigate到对应的details screen, details screen上有一个按钮，点了可以弹出一个bottomsheet。
那我从list上，点击一个item进入details，当我在details page 点back button又回来list page的时候，recomposation发生了吗？

🧠 Recomposition 是什么（用生活类比）

想象你的 UI 是一张“智能白板”。
	•	你第一次画 → 叫 Composition（初次绘制）
	•	之后你改了某一小块 → 系统不会擦掉整块重画
👉 只会局部重画那一小块

👉 这个“局部重画”，就是 Recomposition

⸻

✅ 用一句人话总结

👉 Recomposition = 因为 state 变化，导致某些 Composable 被重新执行，从而更新 UI

不是“页面刷新”，不是“重新加载”，而是：

✨ “哪儿变了，改哪儿”

🔥 回到你这个具体场景（非常好！）

你的流程是：

List Page → 点击 item → Details Page
Details Page → 点 back → 回 List Page

你问：

👉 回到 list page 的时候，有没有 recomposition？

⸻

🎯 正确答案：分情况！（重点来了）

我们一层一层拆👇

⸻

🧩 情况一：List 页面还“活着”（常见）

如果你用的是：
	•	Navigation Compose（默认行为）
	•	或者 Fragment + ComposeView

那么：

👉 List 页面其实 没有被销毁

它只是：

💤 被放到后台（类似“最小化”）

⸻

那返回时发生什么？

👉 不会重新 Composition
👉 也不一定发生 Recomposition

✔ 如果 state 没变
👉 什么都不会发生（UI直接复用）

✔ 如果 state 变了（比如数据刷新）
👉 才会发生 Recomposition

⸻

✅ 所以你的问题答案是：

👉 “回到 list page ≠ 一定发生 recomposition”

⸻

🧩 情况二：List 页面被销毁了

比如你：
	•	popBackStack 彻底清掉
	•	或者 Activity 被重建（旋转 / 内存回收）

那么：

👉 再回来时是：

❗重新 Composition（不是 Recomposition）

⸻

🎯 再看你 Details 页面那个 BottomSheet

你说：

点击按钮弹出 bottom sheet

这里就很经典了👇

```kotlin
var showSheet by remember { mutableStateOf(false) }

if (showSheet) {
    ModalBottomSheet(...)
}
```
👉 当你点击按钮：
```
showSheet: false → true
```

💥 发生什么？

👉 Recomposition 触发
👉 if (showSheet) 这块重新执行
👉 BottomSheet 出现


🧠 一个非常重要的判断标准

❓有没有“state 改变”？

	•	有 👉 一定有 Recomposition
	•	没有 👉 不会发生 Recomposition

💡 用你这个例子做最终总结

👉 List → Details → Back：
	•	❌ 不是自动 recomposition
	•	❌ 不是页面刷新
	•	✅ 是 Navigation 切换 + 可能复用 UI

👉 BottomSheet：
	•	✅ 是标准 Recomposition 场景（state 改变）

⸻

🚀 给你一个进阶理解（你已经快到这一步了）

Recomposition 本质上就是：

🔁 Composable 函数被重新调用（但只针对需要的部分）

不是整棵 UI 树，而是：

👉 “精确打击”

⸻

## About state 

👉 state = 能影响 UI 显示的数据

只要这个数据一变，UI 就可能要变。

⸻

🎯 但重点来了：不是所有数据都是 state

你问得非常关键：

state 是不是 ViewModel 里的 mutableState？

👉 答案是：

❌ 不仅仅是
✅ 所有被 Compose“观察”的数据，才是 state

🧩 我们分三层讲清楚（非常重要）

🥇 第一层：Compose “看得见”的 state（核心）

只有这些，才会触发 recomposition：

1️⃣ mutableStateOf
```
var text by remember { mutableStateOf("") }
```
👉 改 text
💥 一定 recomposition

2️⃣ StateFlow / Flow（通过 collect）
```
val uiState by viewModel.uiState.collectAsState()
```
👉 Flow emit 新值
💥 recomposition

3️⃣ LiveData
```
val data by viewModel.liveData.observeAsState()
```
👉 数据变化
💥 recomposition

👉 重点一句：

✅ 只有“被 collect / observe 的数据”，才算 Compose 的 state

⸻
🧠 第二层：那普通变量算不算？
```
var count = 0
```

👉 你改它：

❌ 不会触发 recomposition
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

举个你现在能秒懂的例子👇
```
@Composable
fun DetailScreen(viewModel: VM) {
    val uiState by viewModel.uiState.collectAsState()

    Text(uiState.name)
}
```
👉 这里的 state 是：
```
uiState
```
不是 ViewModel 本身
而是：

✨ UI 正在“读”的这份数据
🎯 那 recomposition 是怎么触发的？

👉 核心机制是：

📡 Compose 会“订阅”你读取的 state

⸻

🧠 想象一下：

当你写：Text(uiState.name)
系统偷偷做了这件事：“我盯着 uiState.name，一旦它变，我就重新执行这段 UI”

🔥 那你问的关键问题来了：

❓我怎么“控制 state 变没变”？

👉 本质是：

👉 你控制“有没有发出新值”

⸻

🧩 举个超级关键的例子（很多人卡死在这）

❌ 错误写法（不会触发 recomposition）
```
data class UiState(val count: Int)

uiState.count += 1
```
👉 为什么不行？

因为：

👉 对象没变，只是里面的值变了

Compose 看的是：这个“盒子”有没有换,不是里面的东西
✅ 正确写法
_uiState.value = uiState.copy(count = uiState.count + 1)
👉 这次：

💥 新对象 → state 变化 → recomposition
🧠 一个非常关键的直觉（你要建立这个）

👉 Compose 判断的是：

❗“引用有没有变”（是不是新对象）

⸻

🧩 再用一个生活类比（保证你记住）

你可以把 state 想成：

👉 📦 一个“包裹”

⸻

	•	改包裹里面的东西（不换盒子）
👉 Compose：🙈 看不见
	•	换一个新包裹
👉 Compose：👀 发现了！更新 UI！

⸻

🎯 回到你最初的那个场景

List → Details → Back

👉 会不会 recomposition，取决于：

✔ List 用的 state 有没有变

比如：val list by viewModel.list.collectAsState()

情况 1：数据没变

👉 不 recomposition

情况 2：你在 Details 改了数据 viewModel.updateItem(...)
👉 Flow emit 新值
💥 List recomposition

🚀 最后一层理解

👉 Compose 本质是：

🧠 “读取 state → 记录依赖 → state 变 → 重新执行”

## The lifecycle of viewmodel 
🧠 一、ViewModel 到底“什么时候活 / 什么时候死”？

先给你一句定海神针：

✅ ViewModel 的生命周期 ≠ Composable 生命周期
✅ 它跟的是 ViewModelStoreOwner

⸻

🎯 谁是 ViewModelStoreOwner？

常见有三个：
	•	Activity
	•	Fragment
	•	Navigation 的 BackStackEntry（最重要！）

⸻

🧩 在 Navigation Compose 里（你现在的场景）
```
composable("list") {
    val vm: ListViewModel = viewModel()
}
```
👉 这个 ViewModel 属于：

🧱 当前这个 navigation entry（list 这页）

⸻

🔥 那什么时候会被销毁？

👉 非常关键一句：

❗当这个 screen 从 back stack 被移除时

⸻

🧪 举几个你能感知的例子

✔ 正常来回跳 List → Details → Back
👉 List 还在 back stack 里
👉 ViewModel ✅ 还活着

❌ 被 pop 掉 navController.popBackStack("list", inclusive = true)
👉 List 被干掉
👉 ViewModel 💀 销毁

❌ 进程被杀（系统回收）
👉 ViewModel 💀 也没了
🧠 一个超好记的类比

你可以把：
	•	Navigation back stack = 🏨 酒店房间
	•	ViewModel = 🧳 你放在房间里的行李

👉 房间还在 → 行李还在
👉 房间被退掉 → 行李没了

❗二、你问的“scope 写错”是什么意思？

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

🔥 三、你问的 init vs LaunchedEffect（核心 battle）

你观察得非常细：

init 是构造后执行
LaunchedEffect 是 Composable 里执行

👉 ✔ 完全正确
🎯 推荐模式（非常重要）
```
class ListViewModel : ViewModel() {
    init {
        loadData()
    }
}
```
👉 为什么推荐？

因为：

✅ 数据生命周期交给 ViewModel 管

⚠️ 你现在的写法（常见坑）
```
LaunchedEffect(Unit) {
    vm.loadData()
}
```
👉 看起来没问题，但有坑👇

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
🔥 场景：
	1.	App 启动 → ListScreen 首次显示
	2.	点击按钮 Add D
	3.	点击其中一个 item → navigate 到 DetailsScreen（假设是普通 Nav）
	4.	Back 回到 ListScreen

⸻

💡 问题：

1️⃣ 每个步骤，会不会发生 recomposition？
2️⃣ 每个步骤，会不会调用 loadData()？
3️⃣ 每个步骤，LaunchedEffect 会不会执行？
4️⃣ 原因是什么？

场景 1：App 启动 → ListScreen 首次显示
	•	Recomposition ✅ 会发生
	•	解释：Composable 首次执行，UI 组件需要根据初始 state 渲染，所以第一次其实也是 Composition，不是 recomposition，但这个阶段可以理解为 UI 执行了。
	•	loadData() ✅ 会调用
	•	解释：ViewModel 刚创建，init { loadData() } 执行。
	•	state 从 empty list → 有内容
	•	你说“并不是 state 改变，其实不完全对”：
	•	_uiState.value 从 UiState(emptyList()) → UiState(listOf("A","B","C"))
	•	💥 这确实是 state 变化，会触发 recomposition（或者说首次 render 后第二次 render）。
	•	LaunchedEffect(Unit) ✅ 会执行
	•	解释：首次 Composition 执行，会触发一次。

⸻

场景 2：点击按钮 Add D
	•	Recomposition ✅ 会发生
	•	解释：_uiState.value 被 addItem() 改了，Compose 观察到 StateFlow emit 新值 → 依赖它的 Composable 重新执行。
	•	loadData() ❌ 不会调用
	•	解释：ViewModel 没被销毁，init 只执行过一次。
	•	LaunchedEffect(Unit) ❌ 不会执行
	•	解释：Unit 没变 → LaunchedEffect 只执行一次。

👍 完全正确。

⸻

场景 3：点击某个 item → navigate 到 DetailsScreen
	•	Recomposition ❌ 不会发生
	•	解释：ListScreen 虽然仍然在 backstack，但没有 state 改变 → 不需要 recomposition。
	•	loadData() ❌ 不会调用
	•	解释：ViewModel 已经存在，不会再 init。
	•	LaunchedEffect(Unit) ❌ 不会执行
	•	解释：Composable 没被销毁 → Unit 也没变 → LaunchedEffect 不执行。

👍 正确。

⸻

场景 4：Back 回到 ListScreen
	•	Recomposition ❌ 不会发生
	•	解释：ListScreen 从 backstack 回来，本身的 state 没变，UI 不需要重新执行。
	•	loadData() ❌ 不会调用
	•	解释：ViewModel 还在 → init 不会再次执行。
	•	LaunchedEffect(Unit) ❌ 不会执行
	•	✅ 你不确定的地方：LaunchedEffect 只会在 Composable 首次 Composition 时执行，返回 backstack 只是恢复显示，Composable 本身并没有被销毁 → Unit 没变 → LaunchedEffect 不会再执行。

⸻

✅ 总结几个核心点
	1.	Recomposition ≠ 页面显示
	•	页面显示/Navigation 切换，不代表 recomposition，只有 state 改变才触发。
	2.	ViewModel 生命周期决定 loadData 执行次数
	•	init 只在第一次创建时执行。
	3.	LaunchedEffect(Unit) 只执行一次
	•	只在首次 Composition 或 key 改变时执行。
	4.	StateFlow emit 新值 → 观察它的 Composable 会 recomposition
	•	即使 List 内部元素变化，也要用新对象 (copy) 才触发 recomposition。

