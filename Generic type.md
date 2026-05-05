# Generic type

## 使用泛型创建可重复使用的类
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
class Question<T>(
    val questionText: String,
    val answer: T,
    val difficulty: String
)
```
