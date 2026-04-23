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














