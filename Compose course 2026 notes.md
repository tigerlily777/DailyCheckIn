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

### 1.19 Architect - viewModel

### 1.20 Architect - Navigation

