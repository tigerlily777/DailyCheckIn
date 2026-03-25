About recomposition

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
