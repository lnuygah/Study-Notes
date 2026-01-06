ASP.NET Core中的过滤器（Filter）是一种在请求处理管道中的特定阶段之前或之后运行代码的机制。过滤器主要分为以下几种类型：

1. **授权过滤器（Authorization Filters）**：
   - 第一个被执行的过滤器，用于验证请求的合法性。
   - 如果请求不合法，后续的过滤器将不会执行。

2. **资源过滤器（Resource Filters）**（部分资料中也称为“动作过滤器”的一种前置或扩展类型，但在一些ASP.NET Core版本中明确区分）：
   - 在授权过滤器之后执行，主要用于实现缓存和截断功能。
   - 提供了`OnResourceExecuting`和`OnResourceExecuted`方法。

3. **动作过滤器（Action Filters）**：
   - 在调用动作方法之前和之后执行。
   - 可用于修改请求参数、调用动作方法前后的日志记录等。

4. **结果过滤器（Result Filters）**：
   - 在动作方法执行完毕并准备返回结果之前和之后执行。
   - 可用于修改响应结果。

5. **异常过滤器（Exception Filters）**：
   - 用于处理在请求处理管道中抛出的异常。
   - 提供了处理异常的机制，并允许开发者自定义异常处理逻辑。

下面是一个简单的例子，演示如何创建一个自定义的动作过滤器，并将其应用到ASP.NET Core MVC的控制器或动作方法上：

```csharp
// 自定义动作过滤器类
public class CustomActionFilter : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        // 在动作方法执行之前执行的逻辑
        Console.WriteLine("Action is executing.");
        // 可以添加自定义逻辑，如验证请求参数、记录日志等
    }

    public override void OnActionExecuted(ActionExecutedContext context)
    {
        // 在动作方法执行之后执行的逻辑
        Console.WriteLine("Action has executed.");
        // 可以添加自定义逻辑，如修改响应结果、记录日志等
    }
}

// 应用自定义过滤器到控制器或动作方法上
[CustomActionFilter]
public class HomeController : Controller
{
    public IActionResult Index()
    {
        // 动作方法的逻辑
        return View();
    }
}
```

在ASP.NET Core MVC中，过滤器（Filters）之间存在明确的执行顺序。这些过滤器按照特定的顺序在MVC请求管道中执行，以便开发者可以在不同的执行阶段介入处理。以下是ASP.NET Core MVC中过滤器的执行顺序及其相关说明：

1. **授权过滤器（Authorization Filter）**：

   * **执行时机**：这是最先执行的过滤器，用于验证当前请求的合法性，即用户是否有权限访问某个资源。
   * **特点**：授权过滤器只有一个执行前的方法（OnAuthorization），没有执行后的方法。如果请求未通过授权，过滤器会执行“管道短路”操作，即直接拒绝请求，后续的过滤器将不会执行。

2. **资源过滤器（Resource Filter）**：

   * **执行时机**：在授权过滤器之后、模型绑定之前执行。
   * **作用**：资源过滤器在实现缓存或截断过滤器管道方面尤为重要。它可以在请求处理之前和之后执行代码，从而有机会修改数据源或实现缓存策略。
   * **方法**：资源过滤器包含OnResourceExecuting（执行前）和OnResourceExecuted（执行后）两个方法。

3. **操作过滤器（Action Filter）**：

   * **执行时机**：在控制器操作方法执行前后执行。
   * **作用**：操作过滤器可以用于修改传入操作方法的参数，或改变操作方法返回的结果。它提供了一种在请求处理管道中对操作方法进行预处理和后处理的机制。
   * **方法**：操作过滤器包含OnActionExecuting（执行前）和OnActionExecuted（执行后）两个方法。

4. **异常过滤器（Exception Filter）**：

   * **执行时机**：当MVC操作方法运行过程中发生异常时执行。如果操作方法没有发生异常，则异常过滤器不会执行。
   * **作用**：异常过滤器用于捕获并处理操作方法中抛出的异常，从而提供全局的异常处理策略。
   * **方法**：异常过滤器只包含一个OnException（异常发生时）方法。

5. **结果过滤器（Result Filter）**：

   * **执行时机**：在MVC操作方法成功执行后、视图结果执行前执行。
   * **作用**：结果过滤器可以用于修改操作方法的返回结果，例如添加HTTP消息头或修改Cookie等。
   * **方法**：结果过滤器包含OnResultExecuting（执行前）和OnResultExecuted（执行后）两个方法。

综上所述，ASP.NET Core MVC中的过滤器之间存在明确的执行顺序，这个顺序是：授权过滤器 → 资源过滤器 → 操作过滤器（执行前）→ 控制器操作方法 → 操作过滤器（执行后）→ 结果过滤器 → 响应请求。需要注意的是，如果某个过滤器在执行过程中执行了“管道短路”操作，那么后续的过滤器将不会执行。
