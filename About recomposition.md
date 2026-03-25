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

