## rengwuxian compose course 2026

### 1.3 mutableStateOf -> by vs =

```kotlin
class MainActivity : ComponentActivity() {
    val name = mutableStateOf("Tiger")

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            Compose2026Theme {
                Box(modifier = Modifier.safeContentPadding()) {
                    Text(name.value)
                }
            }
        }

        lifecycleScope.launch {
            delay(3000)
            name.value = "Ning"
        }
    }
}
```

When using "=", name is observing mutableState, rather than the value inside of it. As a result, it requires to use ```name.value``` to access the value of mutableState.
When using "by", name value is reading from delegation.
```kotlin
class Example {
    var p: String by Delegate()
}
```
```kotlin
class Delegate {
   operator fun getValue(...) { ... }
   operator fun setValue(...) { ... }
}
```
Delegate class has ❌ extends, ❌ implement, just need to have getValue && setValue with matching args, return types, and must have **operator** key word.
```kotlin
...
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.setValue
...


class MainActivity : ComponentActivity() {
    val name = mutableStateOf("Tiger")
    var name2 by mutableStateOf("DJ")

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            Compose2026Theme {
                Box(modifier = Modifier.safeContentPadding()) {
                    Text(name.value)
                    Spacer(modifier = Modifier.padding(bottom = 16.dp))
                    Text(name2)
                }
            }
        }

        lifecycleScope.launch {
            delay(3000)
            name.value = "Ning"
            name2 = "SS"
        }
    }
}
```

### 1.4 remember 

compose > layout > draw

```kotlin
class MainActivity : ComponentActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            Compose2026Theme {
                Box(modifier = Modifier.safeContentPadding()) {
                    val name = mutableStateOf("Tiger")
                    Text(name.value)
                }
            }
        }

        lifecycleScope.launch {
            delay(3000)
            name.value = "Ning"
        }
    }
}
```
What if I move ```val name = mutableStateOf("Tiger")`` inside of the Box composable? 

When name value changes, it will trigger recompostion, during recomposition, val name is created again. This is why if move name declaration into composable, it will not change the name.

### 1.18 Architect - screens/pages
**activity**

single activity + multiple fragment 

activity -> Fragment -> view FragmentManager add() replace()

**Composable**
Activity > Composable > Composable

ViewModel > Jetpack

Navigation 3 > Jetpack Compose

Animation, read/write data to bundle

**State**
局部状态 - remember
外部状态 - state hoisting
screen的state - viewmodel 管理

### 1.19 Architect - viewModel
Use viewmodel to manage screen data so it will survive during configuration changes
stateflow/mutableStateflow

### 1.20 Architect - Navigation

Navigating between multiple screens
- Backstack
- Pass data
- Animation

Adding navigation 3 dependency to the app:

```toml
[versions]
...
navigation = "1.0.1"
lifecycle = "2.10.0"

[libraries]
nav3 = { group = "androidx.navigation3", name = "navigation3-ui", version.ref = "navigation" }
viewmodel-nav3 = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-navigation3", version.ref = "lifecycle" }
```

```kotlin
class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            val backStack = remember { mutableStateListOf<String>() }

            NavDisplay(backStack) { key ->
                when (key) {
                    "list" -> NavEntry(key) { ListScreen() }
                    "details" -> NavEntry(key) { DetailsScreen() }
                    else -> NavEntry(key) { }
                }
            }
        }
    }
}

@Composable
fun ListScreen(onClick: () -> Unit) {
    Box(modifier = Modifier.fillMaxSize().background(Color.Yellow).clickable { onClick() }) 
}

@Composable
fun DetailsScreen() {
    Box(modifier = Modifier.fillMaxSize().background(Color.Green))
}
```
