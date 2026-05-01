## Use Navigation2 in compose

### 1. Set up
```kotlin
plugins {
  // Kotlin serialization plugin for type safe routes and navigation arguments
  kotlin("plugin.serialization") version "2.0.21"
}

dependencies {
  val nav_version = "2.9.7"

  // Jetpack Compose integration
  implementation("androidx.navigation:navigation-compose:$nav_version")

  // Views/Fragments integration
  implementation("androidx.navigation:navigation-fragment:$nav_version")
  implementation("androidx.navigation:navigation-ui:$nav_version")

  // Feature module support for Fragments
  implementation("androidx.navigation:navigation-dynamic-features-fragment:$nav_version")

  // Testing Navigation
  androidTestImplementation("androidx.navigation:navigation-testing:$nav_version")

  // JSON serialization library, works with the Kotlin serialization plugin
  implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")
}
```

## 2. Navigation controller
```val navController = rememberNavController()```
> [!IMPORTANT]
> - You should create the NavController high in your composable hierarchy. It needs to be high enough that all the composables that need to reference it can do so.
> - **State hoisting**

Views

If you are using the Views UI framework, you can retrieve your NavController using one of the following methods depending on the context:

**Kotlin:**

- Fragment.findNavController()
- View.findNavController()
- Activity.findNavController(viewId: Int)

**Java:**

- NavHostFragment.findNavController(Fragment)
- Navigation.findNavController(Activity, @IdRes int viewId)
- Navigation.findNavController(View)

Typically, you first get a NavHostFragment, and then retrieve the NavController from the fragment. 
```kotlin
val navHostFragment =
    supportFragmentManager.findFragmentById(R.id.nav_host_fragment) as NavHostFragment
val navController = navHostFragment.navController
```
## 3. Design your navigation graph

### Destination types
- Hosted > Main and detail screens > Fills the entire navigation host. 

- Dialog > Alerts, selections, forms > Presents overlay UI components. This UI is not tied to the location of the navigation host or its size. Previous destinations are visible underneath the destination.
- Activity > Serve as an exit point to the navigation graph that starts a new Android activity that is managed separately from the Navigation component. In modern Android development, an app consists of a single activity. Activity destinations are therefore best used when interacting with third party activities or as part of the migration process. > Represents unique screens or features within the app.

### Compose
In Compose, use a serializable object or class to define a route. A route describes how to get to a destination, and contains all the information that the destination requires. Once you have defined your routes, use the NavHost composable to create your navigation graph. 
```kotlin
@Serializable
object Profile
@Serializable
object FriendsList

val navController = rememberNavController()

NavHost(navController = navController, startDestination = Profile) {
    composable<Profile> { ProfileScreen( /* ... */ ) }
    composable<FriendsList> { FriendsListScreen( /* ... */ ) }
    // Add more destinations similarly.
}
```
1. A **serializable** object represents each of the two routes, Profile and FriendsList.
2. The call to the NavHost composable passes a NavController and a route for the start destination.
3. The lambda passed to the NavHost ultimately calls NavController.createGraph() and returns a NavGraph.
4. Each route is supplied as a type argument to NavGraphBuilder.composable<T>() which adds the destination to the resulting NavGraph.
5. The lambda passed to composable is what the NavHost displays for that destination.

> - ✅ route = screen "Unique ID" + "define args"
> - Route is defined with @Serializable tag. Use data class for route with args, use object for route without args.
```kotlin
@Serializable
object Home
```
```kotlin
@Serializable
data class Detail(val id: String)
```
> ❗route 不是“名字”，而是 “可以被序列化的一段导航数据”


```
it's more like:
/home
/detail?id=123
```

### What is NavHost?
> 🖥️ 一个“导航容器” + 一张“路由表”
```
NavHost(navController, startDestination = Home) {
    composable<Home> { HomeScreen() }
    composable<Detail> { DetailScreen() }
}
```
👉 这里干了两件事：

1. 📍 定义有哪些 screen（route → UI）
2. 🖥️ 决定“当前显示哪个”

🧠 类比（非常重要）

👉 NavHost 就像：📺 A TV（screen）

👉 composable(route)：📀 different channels

👉 navController：🎮 controller

### ❓❓Question: one NavHost or Multiple NavHost?

✅ Usually we have multiple screens in one NavHost. It's common.
```
App
 └── NavHost
      ├── Home
      ├── Detail
      ├── Profile
```
Sometimes we need multiple NavHost esp for different bottom navigation items.
```
App
 ├── Tab A（一个 NavHost）
 │     ├── A1
 │     └── A2
 ├── Tab B（一个 NavHost）
 │     ├── B1
 │     └── B2
```
👉 why multiple NavHost？👉 Every Tab has its own back stack.

✅ When do we need only 1 navHost?

👉 所有 screen 在“同一条导航链”里 🧠 什么叫“同一个流程”？用户是“一路走过去的”

## 提问：

但是实际app有可能情况非常复杂。比如app首页有个quick link section，用户可以选择把app上几个最常用的feature 放到这个quick link的menu中，那么每个feature实际上就有不止一个入口。再或者我的app首页有个小铃铛可以查看app 的notification history，每一个notification可能还带一个超链接可以直接进入某个feature。同一个feature的notification 可能进入这个feature的list page，也可能进入这个feature的details page，这种情况如何设计？ 本来这个feature的list和details可以写到同一个navhost里，但是现在由于各自都有外部入口，他们的关系变得非常复杂。我还是觉得实际使用中，很难判断什么时候把哪些screen放进一个host里

## 回答：
🧠 记住原则

你现在卡住的原因是：

> ❌ 你在用“入口”来决定 NavHost
> ✅ 但正确方式是：用“流程（flow）”来决定 NavHost
> 👉 一句话总结：🔥 入口可以有无数个，但 flow 只有几个

🎯 一、你这个复杂场景，其实可以这样拆

你说的情况：

* 首页 quick link → 进入某个 feature
* notification → 进入 list 或 detail
* 正常路径 → list → detail

👉 表面上：```入口很多（Home / Notification / Quick Link）```

👉 但本质上：```Flow 只有一个：Feature Flow：List → Detail```

🧠 核心理解（非常重要）

> ❗Navigation design doesn't care about where do you come from, but where are you going to.

🧩 二、正确设计方式（关键来了）

👉 所有这些入口：

* quick link
* notification
* 首页点击

👉 最终都应该做一件事：

👉 统一调用 navController.navigate(route)

🧠 三、你担心的点其实是这个

你在想：

❓“既然入口这么多，那是不是 navigation 结构也要变复杂？”

👉 答案是：

❌ 不需要

✅ 入口复杂 ≠ navigation 复杂

🧩 四、真正要设计的是什么？

👉 不是 NavHost

👉 而是：

> 🔥 Route 设计 + 参数设计

⸻

🧪 比如你这个场景

Detail 页面：
```kotlin
@Serializable
data class Detail(
    val id: String,
    val from: String? = null // 可选：来源
)
```
🥇 正常路径```navController.navigate(Detail(id = "123"))```

🥈 notification 点进去```navController.navigate(Detail(id = "123", from = "notification"))```

🥉 quick link 点进去 ```navController.navigate(List)```

🧠 五、那 NavHost 怎么设计？

👉 你这个例子，答案很明确：✅ 一个 NavHost 就够了

```
NavHost
 ├── Home
 ├── Feature List
 ├── Feature Detail
 ├── Notification List
```

🔥 六、真正复杂的点在哪？（你已经感觉到了）

👉 不是 Navigation 本身

👉 而是：

> ⚠️ “返回逻辑” + “状态恢复”

🧪 举个你会遇到的坑

用户从 notification 直接进 Detail

👉 这时候按 back：

❓ 回哪里？

* Home？
* Notification？
* 直接退出？
👉 这才是你真正要设计的东西

⸻

🧠 解决方式（核心思路）

👉 不是拆 NavHost，而是：

方式 1：控制 back stack

```kotlin
navController.navigate(Detail(id)) {
    popUpTo(Home)
}
```
方式 2：人为构造路径

Home → List → Detail 即使用户是从 notification 来的 👉 让用户体验一致

🧩 七、一个超级重要的设计原则（你一定要记住）

> 🔥 Navigation graph = UI 结构
> 而不是入口结构

👉 入口只是：👉 “触发 navigation 的事件”

* deep link
* button click
* notification

🧠 八、一些总结（实战用）

⸻

🎯 什么时候放一个 NavHost？

👉 满足：

* screen 之间有前后关系（flow）
* 用户可以在它们之间来回

⸻

🎯 什么时候拆 NavHost？

👉 满足：

* 完全独立的导航世界（Tab）
* 不共享 back stack

⸻

🎯 入口很多怎么办？

👉 不影响 NavHost

👉 只需要：

✅ 统一 route
✅ 设计好参数

### Nav graph

🧠 一句话先定住

> 🔥 Navigation graph = 一组有组织的 route（带结构的导航地图）

🧩 一、你现在看到的 NavHost，其实已经是 graph 了

```
NavHost(navController, startDestination = Home) {
    composable<Home> { HomeScreen() }
    composable<Detail> { DetailScreen() }
}
```
👉 这里其实就是一个：

> ✅ 最简单的 navigation graph

👉 所以：

> ❗NavHost = graph 的“载体”
> ❗graph = NavHost 里面那一坨定义

🧠 二、那为什么还要单独提 graph？（关键点）

因为：

👉 当你的 app 变复杂时，你需要“分组”和“分层”

👉 这时候就会用到：

## 🔥 Nested Navigation Graph（嵌套图）

🧩 三、graph 的真正作用（重点来了）

🎯 作用 1：分模块（最常用）

比如你的 app：
```
App
 ├── Auth（登录流程）
 └── Main（主功能）
```

👉 你可以这样写：
 ```
NavHost(...) {
    navigation(startDestination = Login, route = "auth") {
        composable<Login> { ... }
        composable<Register> { ... }
    }

    navigation(startDestination = Home, route = "main") {
        composable<Home> { ... }
        composable<Detail> { ... }
    }
}
```

👉 这里：

* "auth" = 一个 graph
* "main" = 一个 graph

💡 本质：

> 👉 graph = route 的“分组容器”

⸻

🧠 作用 2：定义一个“流程入口”

👉 每个 graph 都有：```startDestination```

👉 意味着：> 👉 你可以“跳到一个流程”，而不是一个 screen

🧪 举个非常重要的例子

navController.navigate("auth")

👉 实际效果：进入 auth graph → 自动跳到 Login

👉 这就是：

💥 graph 可以作为 navigation 的目标

🧠 作用 3：控制 back stack（高级但重要）

👉 graph 可以作为一个整体被 pop

```navController.popBackStack("auth", inclusive = true)```

👉 意味着：

👉 一次性退出整个流程

🧪 典型场景
Login → Register → ForgotPassword

👉 登录成功：
```popUpTo("auth") { inclusive = true }```
👉 整个登录流程清空

🧠 作用 4：解耦（架构级）

👉 大项目里：

* 每个 feature 一个 graph
* 每个 graph 在不同 module

⸻

👉 最终主 NavHost 只是：

```kotlin
include(authGraph)
include(mainGraph)
```

👉 这就是：

> 💥 模块化 navigation

🧩 四、graph 和 route 的关系（你一定要搞清）

👉 我帮你画一个结构👇

```
NavHost
 ├── Graph: auth
 │     ├── Login
 │     └── Register
 │
 └── Graph: main
       ├── Home
       └── Detail
```

🧠 五、你现在的问题本质上是这个

> ❗入口不影响 graph
> ❗graph 只关心“功能结构”

🧪 回到你之前那个复杂场景

```
Feature
 ├── List
 └── Detail
```
👉 正确设计：

```kotlin
navigation(startDestination = List, route = "feature") {
    composable<List> { ... }
    composable<Detail> { ... }
}
```

👉 不管入口：

* quick link
* notification
* home

👉 全部：```kotlin navController.navigate(Detail(id))```

🔥 六、❌ graph 不是必须的, ✅ graph 是“当你需要分组时才用”

🎯 七、总结成一句你能记住的话

🧠 graph 是什么？

> 👉 一组有 startDestination 的 route 分组

⸻

🎯 用来干嘛？

* 分模块（Auth / Main / Feature）
* 表示一个 flow
* 控制 back stack
* 支持模块化

⸻

❗最重要一句

🔥 graph 是“流程”，route 是“页面”

## Dialog destination
为什么银行 App 离不开 Dialog 导航？
在普通的 Compose 中，我们常用 MutableStateOf(showDialog) 来控制弹窗。但在 Navigation 框架下，把弹窗定义为一个“目的地”有奇效：
```
- 二重验证 (2FA)：用户点转账，弹出一个“请输入短信验证码”的 Dialog。如果这是一个独立的 dialog 目的地，它会有自己独立的生命周期。

- 返回键处理：作为导航目的地的 Dialog，用户按手机物理返回键时，Navigation 会自动帮你关闭弹窗，而不需要你手动写 onBack 逻辑。

- Deep Link 直达：甚至可以实现“点击通知，直接弹出一个特定的风险提示对话框”。
```
文档要点简述：
在 NavHost 里，不要用 composable<T>，而是改用 dialog<T>。
```kotlin
NavHost(...) {
    composable<Home> { ... }
    // 这是一个弹窗目的地
    dialog<RiskWarning> {
        RiskWarningScreen()
    }
}
```
| 维度 | 传统 UI 组件式 (AlertDialog) | 导航目的地式 (dialog route) |
| ----------- | ----------- | ----------- |
| 定义方式 | 在 composable 内部定义 mutableStateOf(show) | 在 NavHost 里使用 dialog<Route> 定义 |
| 触发方式 | 修改布尔值状态 | 调用 navController.navigate(Route) |
| 返回键处理 | 需要手动处理 | onDismissRequest,自动处理。按返回键会自动从 BackStack 弹出 |
| URL/路由 | 没有独立路由 | 有独立路由，支持 Deep Link 直接打开 |
| 生命周期 | 依附于所属的 Screen | 拥有独立的生命周期 (LifecycleOwner) |

2. 为什么要用 NavHost 来 host 一个 dialog？
你提到以前可能是在 Fragment 里创建。现在在 NavHost 里定义 dialog<T>，其实是把 Dialog 看作一个独立的页面。

银行 App 实战场景：
想象一下“转账流程”：

Screen A: 输入金额。

Dialog B: 弹出“风险提示”，要求点击“同意”。

Screen C: 输入密码。

如果你用 dialog<RiskRoute>：

解耦：Screen A 不需要知道 RiskDialog 的存在，它只需要发出“我想去转账”的指令，导航器根据业务逻辑决定是否要 Maps 到 RiskDialog。

状态保存：即使手机横竖屏切换，Navigation 也能帮你记住当前停留在哪个 Dialog 上。

Deep Link：如果后台发个推送说“您的账户有风险，请查看”，点击后可以直接通过路由定位到这个 Dialog。

3. 什么时候该用哪种？
正如你阅读的 对话框目的地 文档中所述：

使用对话框目的地代表应用中需要自己拥有单独界面时...如果您想使用对话框显示不太复杂的提示（例如确认），不妨考虑使用 AlertDialog。

用 NavHost 的 dialog： 逻辑复杂、需要独立 ViewModel、需要通过路由跳转、或者是一个完整的业务子流（如：登录后的协议确认）。

用普通 Compose AlertDialog： 简单的二次确认（如：“确定退出登录吗？”）、加载等待框（Loading）。

### 提问

比如你说拥有独立生命周期，为什么一个dialog会需要独立生命周期呢？有什么例子吗，在什么情况下需要？dialog不是跟着fragment的生命周期走就行了？
还有返回键处理这，如果我想创建一个dialog，但是我不希望user通过device back button dismiss dialog，只能通过dialog上的button来dismiss，这种dialog route还能做到吗？
再有你举的例子里，手机横竖屏切换，dialog不会消失，这不是viewmodel管的事情吗？如果viewmode给一个状态让compose显示dialog，那横竖屏的时候dialog是不是也不会消失，这种是常见的写法吗？


1. 为什么 Dialog 需要独立的生命周期？
在传统的写法里，Dialog 确实常跟着 Fragment 走。但在 Compose Navigation 中，给 Dialog 一个独立的生命周期（LifecycleOwner）有以下几个典型的银行场景：

自动倒计时/超时退出：
假设你在显示一个“交易确认验证码”的 Dialog。这个 Dialog 需要在显示 60 秒后自动关闭并提示超时。如果它有独立生命周期，你可以在 dialog 内部通过 LaunchedEffect 监听 OnResume，直接在 Dialog 级别管理这个倒计时。

埋点监控：
银行合规通常要求记录用户在每一个弹窗停留的时间。有独立生命周期意味着你可以精准捕捉到这个 Dialog 什么时候“可见”，什么时候“消失”，而不需要在主页面里写一大堆逻辑去判断弹窗状态。

Scoped ViewModel：
这是最核心的。你可以为这个 Dialog 绑定一个专属的 ViewModel。当 Dialog 消失时，这个 ViewModel 会自动被销毁（onCleared）。

例子： 一个复杂的“理财产品风险揭示书”弹窗，里面需要请求网络数据、校验滚动位置。给它一个独立的 ViewModel，比把这些逻辑全塞进主页面的 ViewModel 要干净得多。

2. 拦截返回键：不许用户通过 Back 键关闭
在银行 App 中，有些弹窗是强制性的（比如：强制更新、检测到环境风险、支付中的 Loading）。

能不能做到？ 能。
虽然你使用了 dialog 路由，但你依然可以控制它的行为。在定义 dialog 时，你可以传入 dialogProperties：

```kotlin
dialog<RiskRoute>(
    dialogProperties = DialogProperties(
        dismissOnBackPress = false, // 禁用物理返回键关闭
        dismissOnClickOutside = false // 禁用点击弹窗外部关闭
    )
) {
    RiskScreen(
        onConfirm = { navController.popBackStack() } // 只能通过点击按钮关闭
    )
}
```

这样一来，这个 Dialog 就像一个“钉子户”，用户按返回键没反应，只能乖乖按照你 UI 上的逻辑走。

3. 横竖屏切换与状态恢复：ViewModel vs Navigation
这是一个非常经典的误区，我们来理清一下：

ViewMode 管的是“数据”：比如 isDialogShowing = true。

Navigation 管的是“路径”：比如“当前用户在哪个门牌号”。

为什么推荐用 Navigation 管？
如果你用 isDialogShowing：

逻辑分散：你的 HomeScreenViewModel 里要写满 showLoading, showRiskDialog, showSuccessDialog 各种布尔值。

堆栈管理难：如果弹窗 A 消失后要弹窗 B，你需要手动写逻辑控制布尔值的转换。

如果你用 Navigation dialog：
它是自动的。当你调用 Maps(RouteB) 时，Navigation 自动把 B 压入堆栈。

横竖屏切换时：系统会销毁并重建 Activity。Navigation 会自动根据 BackStack（返回栈）里的记录，重新把那个 Dialog 给“画”出来。

写法是否常见？：在现代 Compose 开发中，越来越常见。尤其是复杂的业务流。对于简单的“确定/取消”，用 AlertDialog 配合状态位确实更轻量；但对于有业务逻辑的弹窗，用 dialog 路由是更规范的做法。


## Type safty









