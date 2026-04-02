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
You should create the NavController high in your composable hierarchy. It needs to be high enough that all the composables that need to reference it can do so.
**State hoisting**

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
