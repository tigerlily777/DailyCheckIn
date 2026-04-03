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
- Hosted > Main and detail screens > Fills the entire navigation host. That is, the size of a hosted destination is the same as the size of the navigation host and previous destinations are not visible.
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
1. A ==serializable==. object represents each of the two routes, Profile and FriendsList. ==very important words==.
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
❓Question: one NavHost or Multiple NavHost?
✅ Usually we have multiple screens in one NavHost. It's common.
App
 └── NavHost
      ├── Home
      ├── Detail
      ├── Profile

Sometimes we need multiple NavHost esp for different bottom navigation items.
App
 ├── Tab A（一个 NavHost）
 │     ├── A1
 │     └── A2
 ├── Tab B（一个 NavHost）
 │     ├── B1
 │     └── B2
