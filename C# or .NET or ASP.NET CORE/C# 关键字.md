## C#关键字

### 1. 访问修饰符的作用

访问修饰符是用于指定成员或类型的声明可访问性的关键字。 

### 四种常见的访问修饰符

- public（公共的）
- protected（受保护的）
- internal（内部的）
- private（私有的）

### 访问修饰符的六种组合及其可访问性级别

- public 访问不受限制
- protected 访问限于包含类或派生自包含类的类型
- internal 访问限于当前程序集
- private 访问限于包含类
- protected internal 访问限于当前程序集或派生自包含类的类型访问
- private protected 访问限于包含类或当前程序集中包含类的派生类的类型访问

### 2. readonly与const的区别

- readonly关键字（运行时常量）:字段可以在声明或构造函数中初始化，常作为运行时常量使用。
- const关键字（编译时常量）：字段只能在该字段的声明时初始化，常作为编译时常量使用过。

### 3. **virtual的作用**

virtual关键字用于修改方法、属性、索引器或事件声明，并使它们可以在派生类中被重写（使用override关键字对虚方法重写）。 如下是虚方法声明和重写虚方法的示例：

```c#
using System;

class BaseClass
{
    // 基类虚方法
    public virtual void Method1()
    {
        Console.WriteLine("Base - Method1");
    }

    public virtual void Method2()
    {
        Console.WriteLine("Base - Method2");
    }
}

class DerivedClass : BaseClass
{
    // 重写（override）基类虚方法
    public override void Method1()
    {
        Console.WriteLine("Derived - Method1");
    }

    // 使用 new 隐藏基类方法
    public new void Method2()
    {
        Console.WriteLine("Derived - Method2");
    }
}
```

### 4. **override的作用**

扩展或修改继承的方法、属性、索引器或事件的抽象或虚拟实现需要 override 修饰符。

### 5. **static的作用**

> 使用 static 修饰符可声明属于类型本身而不是属于特定对象的静态成员。 static 修饰符可用于声明 static 类。 在类、接口和结构中，可以将 static 修饰符添加到字段、方法、属性、运算符、事件和构造函数。 static 修饰符不能用于索引器或终结器。

### 静态类与非静态类的区别

1. 静态类无法实例化（换句话说，无法使用 new 运算符创建类类型的变量。 由于不存在任何实例变量，因此可以使用类名本身访问静态类的成员）。
2. 静态构造函数只调用一次，在程序所驻留的应用程序域的生存期内，静态类会保留在内存中（即使用Static修饰的类，应用一旦启用静态类就会保留在内存中）。
3. 静态类只包含静态成员。
4. 不能包含实例构造函数。
5. 静态类会进行密封，因此不能继承。 它们不能继承自任何类（除了 Object）。 静态类不能包含实例构造函数。 但是，它们可以包含静态构造函数。

### **静态成员和非静态成员的区别？**

> 成员主要指的是：字段、方法、属性、运算符、事件和构造函数等。

1. 静态成员用static修饰符，非静态成员不需要。
2. 静态成员属于类所有，非静态成员属于类的实例化对象所有。
3. 静态方法里不能使用非静态成员，非静态方法可以使用静态成员。 
4. 每创建一个类的实例，都会在内存中为非静态成员新分配一块新的存储。
5. 静态成员无论类创建多少个实例，在内存中只占同一块区域。

### **静态方法的使用场合**

1. 静态方法最适合工具类中方法的定义。
2. 静态变量适合全局变量的定义。

### **静态方法和非静态方法区别（优/缺点）**

**优点：**

1. 属于类级别的，不需要创建对象就可以直接使用。
2. 全局唯一，内存中唯一，静态变量可以唯一标识某些状态。
3. 在类加载时候初始化，常驻在内存中，调用快捷方便。

**缺点：**

1. 静态方法不能调用非静态的方法和变量。
2. （非静态方法可以任意的调用静态方法/变量）不可以使用 this 引用 static 方法或属性访问器。

### 6. sealed 关键字作用

sealed 关键字用于修饰类、方法或属性，表示该类或成员不可被继承或重写。如果一个类被声明为 sealed，其他类不能继承该类；如果一个方法或属性被声明为 sealed，其他类不能重写该方法或属性。

### 7. this 关键字作用

this 关键字表示当前对象的引用，可以用于访问当前对象的成员。它可以用来区分局部变量和实例变量、在构造函数中调用其他构造函数、传递当前对象给其他方法等。

### 8. base 关键字作用

base 关键字表示基类的引用，可以用于访问基类的成员。它可以用来在子类中调用基类的构造函数、调用基类的方法或属性等。

### 9. using 关键字的作用

- using指令为命名空间创建别名，或导入在其他命名空间中定义的类型
- using 语句定义一个范围，在此范围的末尾将释放对象资源，实现了IDisposiable的类在using中创建，using结束后会自定调用该对象的Dispose方法，释放资源。

### 10. sizeof 关键字作用

- sizeof 运算符返回给定类型的变量所占用的字节数。 
- sizeof 运算符的参数必须是一个[非托管类型](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/builtin-types/unmanaged-types)的名称，或是一个[限定](https://learn.microsoft.com/zh-cn/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters#unmanaged-constraint)为非托管类型的类型参数。

> 补充：类型是 **非托管类型** （如果类型为以下任一类型）：
>
> - `sbyte`、`byte`、、`short``ushort`、`int`、`uint``long``ulong``nint``nuint``char``float``double`或 `decimal``bool`
> - 任何 [枚举](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/builtin-types/enum) 类型
> - 任何 [指针](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/unsafe-code#pointer-types) 类型
> - 一个 [元组](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/builtin-types/value-tuples) ，其成员都是非托管类型
> - 任何仅包含非托管类型的字段的用户定义 [结构](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/builtin-types/struct) 类型。
>
> 可以使用 [`unmanaged` 约束](https://learn.microsoft.com/zh-cn/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters#unmanaged-constraint) 指定类型参数是非指针、不可为 null 的非托管类型。

### 11. delegate 关键字作用

- delegate 关键字用于声明委托类型，即代表一个或多个方法的对象。
- 使用 delegate 可以实现事件和回调机制，简化方法的调用和管理。

### 12. **C# 中的 in 关键字的作用**

in 修饰符通过只读引用传递参数，避免复制大值类型的开销。编译器在参数为字面量、属性返回值或需要类型转换时自动创建临时变量。若 API 明确要求按引用传递参数，应优先使用 ref readonly 修饰符以强化调用语义。

> 使用 in 参数可优化性能，尤其适用于需要频繁传递大型结构体（如循环或关键代码中）的场景。它通过只读引用传递参数，避免复制数据的开销，同时确保方法内部不会修改参数状态。调用时显式添加 in 修饰符，明确要求按引用而非值传递，从而提升效率。

```c#
using System;

class Program
{
    static void Main(string[] args)
    {
        int x = 5;

        MultiplyByTwo(in x);

        Console.WriteLine(x); // 输出 5
    }

    static void MultiplyByTwo(in int number)
    {
        // 无法修改 in 参数的值
        // number *= 2; // 编译错误

        // 仅能读取 in 参数的值
        Console.WriteLine(number * 2); // 输出 10
    }
}
```

### 13. C# 中的 ref 关键字作用

- 参数在使用 ref 关键字进行引用传递时，必须在方法调用之前对其进行初始化。
- ref 关键字既可以在进入方法之前初始化参数的值，也可以在方法内部对参数进行修改。
- ref 参数在进入方法时保持原始值，并在方法结束后将值带回到调用处。

### 14. C# 中的 out 关键字作用

- 参数在使用 out 关键字进行引用传递时，不需要在方法调用之前进行初始化。
- out 关键字通常用于表示方法返回多个值的情况，或者用于修改方法外部的变量。
- out 参数必须在方法内部进行初始化，并确保在方法结束前完成赋值操作。方法内部没有为 out 参数赋值的情况下，方法调用将会导致编译错误。

### 15. C#中参数传递 ref 与 out 区别

- **ref 指定此参数由引用传递**：指定的参数在函数调用时必须先初始化（有进有出）。
- **out 指定此参数由引用传递**：指定的参数在进入函数时会清空参数值，因此该参数必须在调用函数内部进行初始化赋值操作（无进有出）。

**总结：**

- ref 和 out 都用于引用传递参数。
- ref 参数在方法调用前必须被初始化，而 out 参数不需要。
- ref 参数可以保留原始值并在方法内部进行修改，而 out 参数必须在方法内部进行初始化赋值。

### 16. as和is的区别

- is 只是做类型兼容判断，并不执行真正的类型转换。返回true或false，不会返回null，对象为null也会返回false。
- as运算符将表达式结果显式转换为给定的引用类型或可以为null值的类型。 如果无法进行转换，则as运算符返回 null。 

**总结：**
as模式的效率要比is模式的高，因为借助is进行类型转换的化，需要执行两次类型兼容检查。而as只需要做一次类型兼容，一次null检查，null检查要比类型兼容检查快。

### 17. null类型

null 关键字是表示不引用任何对象的空引用的文字值。 null是引用类型变量的默认值。 普通值类型不能为 null，[可为空的值类型](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/builtin-types/nullable-value-types)除外。

### 18. new关键字作用

- 运算符：创建类型的新实例。
- 修饰符：可以显式隐藏从基类继承的成员。
-  泛型约束：泛型约束定义，约束可使用的泛型类型。

### 19. return、continue、break区别

- **return**：结束整个方法，return关键字并不是专门用于跳出循环的，return的功能是结束一个方法。 一旦在循环体内执行到一个return语句，return语句将会结束该方法，循环自然也随之结束。与continue和break不同的是，return直接结束整个方法，不管这个return处于多少层循环之内。
- **continue**：结束本次循环，然后持续进行下一次循环。
- **break**：break用于完全结束一个循环，跳出循环体。不管是哪种循环，一旦在循环体中遇到break，系统将完全结束循环，开始执行循环之后的代码。
