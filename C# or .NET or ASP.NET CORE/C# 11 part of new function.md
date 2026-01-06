##  一、字符串内插中的换行符

字符串内插的 `{` 和 `}` 字符内的文本现在可以跨多个行。 `{` 和 `}` 标记之间的文本分析为 C#。 允许任何合法 C#（包括换行符）。 使用此功能可以更轻松地读取使用较长 C# 表达式的字符串内插，例如模式匹配 `switch` 表达式或 LINQ 查询。

可以在语言参考的[字符串内插](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/tokens/interpolated)文章中了解有关换行符功能的详细信息。

### 1. 使用 `$` 的字符串内插

`$` 特殊字符将字符串文本标识为内插字符串 。 内插字符串是可能包含内插表达式的字符串文本 。 将内插字符串解析为结果字符串时，带有内插表达式的项会替换为表达式结果的字符串表示形式。

字符串内插为格式化字符串提供了一种可读性和便捷性更高的方式。 它比[字符串复合格式设置](https://learn.microsoft.com/zh-cn/dotnet/standard/base-types/composite-formatting)更容易阅读。 比较一下下面的示例，它使用了这两种功能产生相同的输出：

```c#
string name = "Mark";
var date = DateTime.Now;

// 复合格式:
Console.WriteLine("Hello, {0}! Today is {1}, it's {2:HH:mm} now.", name, date.DayOfWeek, date);
// 内插:
Console.WriteLine($"Hello, {name}! Today is {date.DayOfWeek}, it's {date:HH:mm} now.");
// Both calls produce the same output that is similar to:
// Hello, Mark! Today is Wednesday, it's 19:40 now.
```

从 C# 10 开始，可以使用字符串内插来初始化常量字符串。 用于占位符的所有表达式都必须是常量字符串。 换言之，每个内插表达式都必须是一个字符串，并且必须是编译时常量。

从 C# 11 开始，内插表达式可以包含换行符。 `{` 和 `}` 之间的文本必须是有效的 C#，这样它可以包含换行符，从而提高可读性。 下面的示例展示了换行符如何提高涉及模式匹配的表达式的可读性：

```c#
string message = $"The usage policy for {safetyScore} is {
    safetyScore switch
    {
        > 90 => "Unlimited usage",
        > 80 => "General usage, with daily safety check",
        > 70 => "Issues must be addressed within 1 week",
        > 50 => "Issues must be addressed within 1 day",
        _ => "Issues must be addressed before continued use",
    }
    }";
```

此外，从 C# 11 开始，可以对格式字符串使原始字符串字面量: 

```C#
int X = 2;
int Y = 3;

var pointMessage = $"""The point "{X}, {Y}" is {Math.Sqrt(X * X + Y * Y)} from the origin""";

Console.WriteLine(pointMessage);
// output:  The point "2, 3" is 3.605551275463989 from the origin.
```

### 2. 特殊字符

要在内插字符串生成的文本中包含大括号 "{" 或 "}"，请使用两个大括号，即 "{{" 或 "}}"。 有关详细信息，请参阅转义大括号。

因为冒号（“:”）在内插表达式项中具有特殊含义，为了在内插表达式中使用条件运算符，请将表达式放在括号内。

以下示例演示了如何在结果字符串中包括大括号。 它还演示了如何使用条件运算符：

```C#
string name = "Horace";
int age = 34;
Console.WriteLine($"He asked, \"Is your name {name}?\", but didn't wait for a reply :-{{");
Console.WriteLine($"{name} is {age} year{(age == 1 ? "" : "s")} old.");
// Expected output is:
// He asked, "Is your name Horace?", but didn't wait for a reply :-{
// Horace is 34 years old.
```

内插逐字字符串以 `$` 字符开头，后跟 `@` 字符。 可以按任意顺序使用 `$` 和 `@` 标记：`$@"..."` 和 `@$"..."` 均为有效的内插逐字字符串。

---



## 二、自动默认结构

C# 11 编译器可以确保在执行构造函数的过程中，将 `struct` 类型的所有字段初始化为其默认值。 此更改意味着，任何未由构造函数初始化的字段或自动属性都由编译器自动初始化。 现在，构造函数未明确分配所有字段的结构可以进行编译，并且未显式初始化的任何字段都设置为其默认值。 

## 三、必需的成员

可以将 [`required` 修饰符](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/required)添加到属性和字段，以强制构造函数和调用方初始化这些值。 可以将 [System.Diagnostics.CodeAnalysis.SetsRequiredMembersAttribute](https://learn.microsoft.com/zh-cn/dotnet/api/system.diagnostics.codeanalysis.setsrequiredmembersattribute) 添加到构造函数，以通知编译器构造函数将初始化所有必需的成员。

## 关于原始字符串文本

原始字符串字面量是字符串字面量的一种新格式。 原始字符串字面量可以包含任意文本，包括空格、新行、嵌入引号和其他特殊字符，无需转义序列。 原始字符串字面量以至少**三个双引号 (""")** 字符开头。 它以相同数量的双引号字符结尾。 通常，原始字符串字面量在单个行上使用三个双引号来开始字符串，在另一行上用三个双引号来结束字符串。 左引号之后、右引号之前的换行符不包括在最终内容中：

```c#
string longMessage = """
    This is a long message.
    It has several lines.
        Some are indented
                more than others.
    Some should start at the first column.
    Some have "quoted text" in them.
    """;
```

右双引号左侧的任何空格都将从字符串字面量中删除。 原始字符串字面量可以与字符串内插结合使用，以在输出文本中包含大括号。 多个 `$` 字符表示有多少个连续的大括号开始和结束内插：

```C#
int X = 1;
int Y = 2;
var location = $$"""
   You are at {{{X}}, {{Y}}
   """;
// output: "You are at {1, 2}"
```

上述指定两个大括号开始和结束内插，第三个重复的左大括号和右大括号包括在输出字符串中。

