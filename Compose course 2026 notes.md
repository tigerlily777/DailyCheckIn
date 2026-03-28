## rengwuxian compose course 2026

### 1.3 mutableStateOf -> by vs =

```
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
```
class Example {
    var p: String by Delegate()
}
```
```
class Delegate {
   operator fun getValue(...) { ... }
   operator fun setValue(...) { ... }
}
```
Delegate class has no extends, no implement, just need to have getValue and setValue with matching args, return types, and must have **operator** key word.
```
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
