# Generic type

## 1 - Use generic type to create the class that can be used repeatly + Use data class + Use enum class
-1.  假设您正在为某项在线测验编写一款应用，测验问题通常有多种类型，例如填空或判断正误。下面我们来定义三种不同类型的问题。

填空题：答案是由 String 表示的字词。
判断正误：答案由 Boolean 表示。
数学题：答案是数值。简单算术题的答案由 Int 表示。
此外，示例中的测试问题还包含难度等级，string表示："easy"、"medium" or "hard"。

有3种题型
```
class FillInTheBlankQuestion(
    val questionText: String,
    val answer: String,
    val difficulty: String
)

class TrueOrFalseQuestion(
    val questionText: String,
    val answer: Boolean,
    val difficulty: String
)

class NumericQuestion(
    val questionText: String,
    val answer: Int,
    val difficulty: String
)
```
这样使用有很多repeat 
After using generic type

```
fun main() {
    val question1 = Question<String>("Quoth the raven ___", "nevermore", Difficulty.MEDIUM)
    val question2 = Question<Boolean>("The sky is green. True or false", false, Difficulty.EASY)
    val question3 = Question<Int>("How many days are there between full moons?", 28, Difficulty.HARD)
    println(question1.toString())
}

class Question<T>(
    val questionText: String,
    val answer: T,
    val difficulty: Difficulty
)

enum class Difficulty {
    EASY, MEDIUM, HARD
}
```
Run -> output is something like```Question@2dda6444```

So we need to change```class Question<T>``` to ```data class```

再run，output: ```Question(questionText=Quoth the raven ___, answer=nevermore, difficulty=MEDIUM)```

data class has:

```
equals()
hashCode()：在使用某些集合类型时，您会看到此方法。
toString()
componentN()：component1()、component2() 等
copy()
```
## 2 - Use object -> for singleton

在 Kotlin 中，你不需要像 Java 那样写复杂的双重检查锁。只需把 class 换成 object，它就变成了一个单例对象。

特点：

- No constructor（因为你不能手动 new 它）。

- 第一次被访问时自动初始化。

- 全局唯一，状态共享。

```
class Quiz {
    val question1 = Question<String>("Quoth the raven ___", "nevermore", Difficulty.MEDIUM)
    val question2 = Question<Boolean>("The sky is green. True or false", false, Difficulty.EASY)
    val question3 = Question<Int>("How many days are there between full moons?", 28, Difficulty.HARD)

    companion object StudentProgress {
    var total: Int = 10
    var answered: Int = 3
    }
}

fun main() {
    println("${Quiz.answered} of ${Quiz.total} answered.")
}
```

output: ```3 of 10 answered.```

这是你在页面看到的进阶用法。如果你想让某个单例逻辑隶属于某个特定的类，就用 companion object。

作用： 让你能直接通过“类名”来访问属性，语法更简洁。

对比：

普通对象： StudentProgress.total

伴生对象： 放在 Quiz 类里后，直接写 Quiz.total（虽然它定义在内部的 StudentProgress 中）。

💡 为什么这里要讲这个？（联想你的 Compose 学习）

你之前问过“为什么要用 object 定义route”，现在答案呼之欲出了：

你的“主页”或者“转账成功页”本身是不带状态的固定目的地。

使用 object 定义路由，不仅节省了内存（不用每次跳转都建新对象），还保证了导航路径的唯一性和确定性。

## 3 - Extension

1. 什么是扩展？ (The "Magic")
扩展允许你在不修改原始类代码的情况下，给它增加新的功能（属性或方法）。

为什么牛？ 你可以给 Kotlin 的内置类（如 String, Int）或者第三方库的类（如 NavController）添加你自己的逻辑。

Compose 例子： 你在写布局时常用的 16.dp，其实就是有人给 Int 类写了一个叫 dp 的扩展属性。

2. 扩展属性 (Extension Properties)
如果你发现某个逻辑只是为了从现有数据中转换出一个新格式，就用扩展属性。

语法要点：

必须定义为 val（因为扩展属性不能真正存储新数据，它只是一个“计算出的值”）。

必须提供 get() 方法。

代码练习重点：
```
// 为 Quiz.StudentProgress 增加一个 progressText 属性
val Quiz.StudentProgress.progressText: String
    get() = "$answered of $total answered"

fun main() {
    println(Quiz.progressText)
}
```
注意：由于扩展属性无法存储数据，因此它们必须是 get-only 的。

```
fun Quiz.StudentProgress.printProgressBar() {
    repeat(Quiz.answered) { print("▓") }
    repeat(Quiz.total - Quiz.answered) { print("▒") }
    println()
    println(Quiz.progressText)
}

fun main() {
    Quiz.printProgressBar()
}
```
output: 
```
▓▓▓▒▒▒▒▒▒▒
3 of 10 answered.
```

##  使用接口重写扩展函数

2. 接口 vs 扩展 (Extension)
这一课的核心是在对比这两者：

扩展： 像给房子“加装电梯”，你没法改房子内部结构。

接口： 像房子的“设计图纸”，从根源上规定这房子必须有电梯。

结论： 如果你有权修改源代码，且多个类有共同行为，优先用接口。

3. 实战要点：在 Quiz 中应用
文档演示了将之前的扩展功能重构成接口的过程：

定义接口：interface ProgressPrintable，规定了 progressText 和 printProgressBar()。

类实现接口：class Quiz : ProgressPrintable。

关键字 override：当你在类里实现接口定义的属性或方法时，必须加上 override，告诉编译器：“我正在履行合同”。

```
interface ProgressPrintable {
    val progressText: String  // 属性声明
    fun printProgressBar()    // 方法声明
}

class Quiz : ProgressPrintable {
    // 履行合同：提供具体的文字格式
    override val progressText: String
        get() = "$answered of $total answered"

    // 履行合同：提供具体的打印逻辑
    override fun printProgressBar() {
        // ... 之前的 repeat 逻辑 ...
    }
}
```

### 💥 error handling + 分层 + navigation decision

🧠 一、先给你一个“总原则”（非常重要）

🔥 每一层只做一件事：

* Data layer 👉 处理“请求 + 原始错误”
* Domain layer 👉 处理“业务规则 + 结果抽象”
* ViewModel 👉 处理“UI state + 导航决策”
































