# Bottomsheet

![An example of a Material Design 3 modal bottom sheet.](https://developer.android.com/static/develop/ui/compose/images/layouts/material/m3-bottom-sheet.png)

If you want to implement a [bottom sheet](https://m3.material.io/components/bottom-sheets/overview), you can use the
[`ModalBottomSheet`](https://developer.android.com/reference/kotlin/androidx/compose/material3/ModalBottomSheet.composable) composable.

You can use the `content` slot, which uses a [`ColumnScope`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/ColumnScope) to layout sheet
content composables in a column:


```kotlin
ModalBottomSheet(onDismissRequest = { /* Executed when the sheet is dismissed */ }) {
    // Sheet content
}
```

<br />

## Control sheet state programmatically

To programmatically expand and collapse the sheet, use
[`SheetState`](https://developer.android.com/reference/kotlin/androidx/compose/material3/SheetState). You can use [`rememberModalBottomSheetState`](https://developer.android.com/reference/kotlin/androidx/compose/material3/rememberModalBottomSheetState.composable#rememberModalBottomSheetState(kotlin.Boolean,kotlin.Function1)) to create
an instance of `SheetState` that must be passed to `ModalBottomSheet` with
the `sheetState` parameter. `SheetState` provides access to the [`show`](https://developer.android.com/reference/kotlin/androidx/compose/material3/SheetState#show())
and [`hide`](https://developer.android.com/reference/kotlin/androidx/compose/material3/SheetState#hide()) functions, as well as properties related to the current sheet
state. These suspending functions require a `CoroutineScope` --- for example,
using [`rememberCoroutineScope`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/rememberCoroutineScope.composable) --- and you can call them in response to UI
events. Make sure to remove the `ModalBottomSheet` from composition upon hiding
the bottom sheet.


```kotlin
val sheetState = rememberModalBottomSheetState()
val scope = rememberCoroutineScope()
var showBottomSheet by remember { mutableStateOf(false) }
Scaffold(
    floatingActionButton = {
        ExtendedFloatingActionButton(
            text = { Text("Show bottom sheet") },
            icon = { Icon(Icons.Filled.Add, contentDescription = "") },
            onClick = {
                showBottomSheet = true
            }
        )
    }
) { contentPadding ->
    // Screen content

    if (showBottomSheet) {
        ModalBottomSheet(
            onDismissRequest = {
                showBottomSheet = false
            },
            sheetState = sheetState
        ) {
            // Sheet content
            Button(onClick = {
                scope.launch { sheetState.hide() }.invokeOnCompletion {
                    if (!sheetState.isVisible) {
                        showBottomSheet = false
                    }
                }
            }) {
                Text("Hide bottom sheet")
            }
        }
    }
}
```

<br />

You can partially show a [bottom sheet](https://developer.android.com/develop/ui/compose/components/bottom-sheets) and then let the user either make it
full screen or dismiss it.

To do so, pass your [`ModalBottomSheet`](https://developer.android.com/reference/kotlin/androidx/compose/material3/ModalBottomSheet.composable) an instance of [`SheetState`](https://developer.android.com/reference/kotlin/androidx/compose/material3/SheetState)
with `skipPartiallyExpanded` set to `false`.

> [!NOTE]
> **Note:** `ModalBottomSheet` is experimental. File any issues on the [issue
> tracker](https://issuetracker.google.com/issues/new?component=856989&template=1425922).

## Example

This example demonstrates how you can use the `sheetState` property of
`ModalBottomSheet` to display the sheet only partially at first:


```kotlin
@Composable
fun PartialBottomSheet() {
    var showBottomSheet by remember { mutableStateOf(false) }
    val sheetState = rememberModalBottomSheetState(
        skipPartiallyExpanded = false,
    )

    Column(
        modifier = Modifier.fillMaxWidth(),
        horizontalAlignment = Alignment.CenterHorizontally,
    ) {
        Button(
            onClick = { showBottomSheet = true }
        ) {
            Text("Display partial bottom sheet")
        }

        if (showBottomSheet) {
            ModalBottomSheet(
                modifier = Modifier.fillMaxHeight(),
                sheetState = sheetState,
                onDismissRequest = { showBottomSheet = false }
            ) {
                Text(
                    "Swipe up to open sheet. Swipe down to dismiss.",
                    modifier = Modifier.padding(16.dp)
                )
            }
        }
    }
}
```

<br />

### Key points about the code

In this example, note the following:

- `showBottomSheet` controls whether the app displays the bottom sheet.
- `sheetState` is an instance of [`SheetState`](https://developer.android.com/reference/kotlin/androidx/compose/material3/SheetState) where `skipPartiallyExpanded` is false.
- [`ModalBottomSheet`](https://developer.android.com/reference/kotlin/androidx/compose/material3/ModalBottomSheet.composable) takes a modifier that ensures it fills the screen when fully expanded.
- `ModalBottomSheet` takes `sheetState` as the value for its `sheetState` parameter.
  - As a result, the sheet only partially displays when first opened. The user can then drag or swipe it to make it full screen or dismiss it.
- The `onDismissRequest` lambda controls what happens when the user tries to dismiss the bottom sheet. In this case, it only removes the sheet.

### Results

When the user first presses the button, the sheet displays partially:
![A bottom sheet that initially only fills part of the screen. The user can swipe to fill the screen with it, or dismiss it](https://developer.android.com/static/develop/ui/compose/images/components/bottom-sheet-partial.png) **Figure 1.** Partially displayed bottom sheet.

If the user swipes up on the sheet, it fills the screen:
![A bottom sheet that the user has expanded to fill the screen.](https://developer.android.com/static/develop/ui/compose/images/components/bottom-sheet-fullscreen.png) **Figure 2.** Full-screen bottom sheet.

> [!NOTE]
> **Note:** If you set `skipPartiallyExpanded` to true, the sheet opens immediately to full screen.

## Additional resources

- [Bottom sheets](https://developer.android.com/develop/ui/compose/components/bottom-sheets)
