这个文档中会列举不推荐的coding approach

## Navigation

1 - 
```
// ❌ 不推荐的做法
@Composable
fun TransferScreen(navController: NavController) {
    Button(onClick = { navController.navigate(SuccessRoute) }) { ... }
}
```

为什么不能这样写？

测试难：如果你想写个 UI 测试来验证转账按钮，你还得模拟一个复杂的 navController。

逻辑耦合：如果以后转账成功后不是直接跳页面，而是要先弹个窗或者发个埋点，你得去修改 UI 组件内部的代码。

官方推荐的做法（也是 Overview 的精华）：
使用 Lambda 表达式。UI 只负责发出“我被点了”的信号，具体的导航逻辑写在 NavHost 那一层。

```kotlin
// ✅ 推荐做法：状态提升
@Composable
fun TransferScreen(onTransferClick: () -> Unit) {
    Button(onClick = onTransferClick) { ... }
}

// 在 NavHost 中统筹
composable<TransferRoute> {
    TransferScreen(onTransferClick = { navController.navigate(SuccessRoute) })
}
```
