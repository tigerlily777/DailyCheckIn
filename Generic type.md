# Generic type

## 使用泛型创建可重复使用的类 + Use data class + Use enum class
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
现在运行，得到的是output是类似```Question@2dda6444```

所以需要把```class Question<T>``` 变成 data class

再run，得到 ```Question(questionText=Quoth the raven ___, answer=nevermore, difficulty=MEDIUM)```

data class has:

```
equals()
hashCode()：在使用某些集合类型时，您会看到此方法。
toString()
componentN()：component1()、component2() 等
copy()
```
## Use object -> for singleton

在 Kotlin 中，你不需要像 Java 那样写复杂的双重检查锁。只需把 class 换成 object，它就变成了一个单例对象。

特点：

- 不能有构造函数（因为你不能手动 new 它）。

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














