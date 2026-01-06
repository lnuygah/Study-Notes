## 零、简介

[ASP.NET Core](https://link.zhihu.com/?target=http%3A//ASP.NET) 中的权限“子系统”主要包括两个部分：**验证（Autentication）**与 **授权（Authorization）**。

在 ASP.NET Core 中它们是相对独立的，这两个概念对应的具体功能如下，

- **验证：指的是确定用户身份的过程。**
- **授权：指的是确定用户是否可以执行某个操作的过程。**

因为我们要先确定用户的身份才能确定用户是否可以执行某个操作，所以本篇先来介绍验证功能。

## 一、ASP.NET Core 验证功能的支持情况

在  [ASP.NET Core](https://link.zhihu.com/?target=http%3A//ASP.NET) 框架（既 Microsoft.AspNetCore.App 包）中，内置有下面两种验证方式：

- Microsoft.AspNetCore.Authentication.Cookies
- Microsoft.AspNetCore.Authentication.OAuth

除此之外微软也提供了多个第三方包来实现各种常用的验证方式：

- Microsoft.AspNetCore.Authentication.AzureAD.UI
- Microsoft.AspNetCore.Authentication.AzureADB2C.UI
- Microsoft.AspNetCore.Authentication.Certificate
- Microsoft.AspNetCore.Authentication.Facebook
- Microsoft.AspNetCore.Authentication.Google
- Microsoft.AspNetCore.Authentication.JwtBearer
- Microsoft.AspNetCore.Authentication.Microsoft
- Microsoft.AspNetCore.Authentication.Negotiate
- Microsoft.AspNetCore.Authentication.OpenIdConnect
- Microsoft.AspNetCore.Authentication.Twitter
- Microsoft.AspNetCore.Authentication.WsFederation

## 二、主要概念

在开发 [ASP.NET Core](https://link.zhihu.com/?target=http%3A//ASP.NET) 验证功能的过程中，需要了解下面几个“东西”：

### **1、Autentication 服务与中间件**

Autentication 服务是  [ASP.NET Core](https://link.zhihu.com/?target=http%3A//ASP.NET) 内置验证功能所需的最基础的服务。 使用 IServiceCollection.AddAuthentication 扩展方法可以方便的添加 Authentication 服务，并返回 **AuthenticationBuilder** 对象，该对象是用于具体验证方式配置的核心对象。

[AddAuthentication源码](https://link.zhihu.com/?target=https%3A//github.com/dotnet/aspnetcore/blob/52eff90fbcfca39b7eb58baad597df6a99a542b0/src/Security/Authentication/Core/src/AuthenticationServiceCollectionExtensions.cs)

[AuthenticationService源码](https://github.com/dotnet/aspnetcore/blob/52eff90fbcfca39b7eb58baad597df6a99a542b0/src/Http/Authentication.Core/src/AuthenticationService.cs#L17)

**需要说明的就是在我们调用完这个AddAuthentication的方法之后并不意味着我们的应用就拥有了可用的验证功能，该方法的作用是为验证功能提供一个地基，具体的验证方式还需要额外的配置。**

>  a. 启用验证功能的第一步就是在 Startup.ConfigureServices 方法中调用 services.AddAuthentication 方法。
>
>  ```csharp
>  // 该方法返回一个 AuthenticationBuilder 这是配置验证功能的核心类，
>  // 上面提到的Cookies和OAuth验证功能都是通过对这个类添加扩展方法来实现的。
>  services.AddAuthentication();
>  ```

[AuthenticationBuilder源码](https://link.zhihu.com/?target=https%3A//github.com/dotnet/aspnetcore/blob/52eff90fbcfca39b7eb58baad597df6a99a542b0/src/Security/Authentication/Core/src/AuthenticationBuilder.cs)

> b. 添加了 Authentication 服务之后，还需要添加 Authentication 中间件来使验证功能工作。在Startup.Configure 方法中调用 app.UseAuthentication() 方法来添加验证中间件。
>
> ```csharp
> //添加 Microsoft.AspNetCore.Authentication.AuthenticationMiddleware
> app.UseAuthentication();
> ```

[AuthAppBuilderExtensions.cs源码](https://link.zhihu.com/?target=https%3A//github.com/dotnet/aspnetcore/blob/52eff90fbcfca39b7eb58baad597df6a99a542b0/src/Security/Authentication/Core/src/AuthAppBuilderExtensions.cs)

到这里一个验证的基础搭建就差不多了。

### **2、方案 / Scheme**

在一个 [ASP.NET Core](https://link.zhihu.com/?target=http%3A//ASP.NET) 应用中你可以配置多种验证方式，而你配置的每种验证方式被称为一个**验证方案 / Scheme。**比如下面的代码配置了两种验证方式，也就是两个验证方案，每个方案都有一个名称标识被称为方案名称。

```csharp
//调用AddAuthentication时你可以指定一个默认的Scheme名称
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    //配置一个JwtBearer验证方式，并命名为 JwtBearerDefaults.AuthenticationScheme
    .AddJwtBearer(JwtBearerDefaults.AuthenticationScheme, options => Configuration.Bind("JwtSettings", options))
    //配置第二个Cookie验证方式，并命名为 CookieAuthenticationDefaults.AuthenticationScheme
    .AddCookie(CookieAuthenticationDefaults.AuthenticationScheme, options => Configuration.Bind("CookieSettings", options));
```

一个验证方案对应一个验证方式的配置，方案名称会在之后的配置和授权中被使用到。

### **3、用户凭证**

在用户通过某种方式登录到系统之后，我们需要给用户一个凭证作为验证用户的依据，这样用户在使用系统的时候只需要提供凭证即可。

在Cookie验证方式中，用户凭证被保存在Cookie中，这样服务器可以在每次用户请求时验证其Cookie中的凭证。

### **4、上下文用户对象 / ClaimsPrincipal**

验证的结果需要提供一个包含用户身份信息的对象，在[ASP.NET Core](https://link.zhihu.com/?target=http%3A//ASP.NET) 中这个对象为 **HttpContex.User**，对象类型则为 System.Security.Claims.ClaimsPrincipal。

ClaimsPrincipal 及其相关类型的关系和含义如下：

**a）继承关系** ClaimsIdentity：IIdentity ； ClaimsPrincipal：IPrincipal

**b）IIdentity**，Identity 的含义是一个可验证的身份，而接口 IIdentity 代表了一个可验证身份所包含的基本信息。

**c）ClaimsIdentity**，基于声明(Claim)的Identity，所谓基于声明就是使用声明的方式来描述 Identity 包含的信息，比如：你要为这个 Identity 添加 Name 信息你就为它添加一个声明：

```csharp
claimsIdentity.AddClaim(new Claim(ClaimTypes.Name, "user_name"));
```

**d**）**IPrincipal**，Principal 单词有主角，委托人的含义，这里它代表了当前运行代码的安全上下文中的用户，包含用户身份（Identity），角色以及其他信息。

**e）ClaimsPrincipal**，基于声明的安全上下文用户，可以包含一个或多 ClaimsIdentity。

[ClaimsPrincipal类 ](https://link.zhihu.com/?target=https%3A//docs.microsoft.com/zh-cn/dotnet/api/system.security.claims.claimsprincipal%3Fview%3Dnet-5.0)

## 三、配置（JwtBearer验证方式）

配置包括三个部分：

- 第一步，添加 Authentication 服务
- 第二步，添加并配置 JwtBearer 验证方式
- 第三步，添加 Authentication 中间件

关于服务和中间件已经在上文中介绍了。配置 Cookie 验证方式所需的对象为 services.AddAuthentication() 方法返回的 **AuthenticationBuilder** 对象，JwtBearerExtensions 为 AuthenticationBuilder 提供了一个 AddJwtBearer 扩展方法可以让我们方便的添加和配置 JwtBearer 验证方式。

[JwtBearerExtensions类官网介绍](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.extensions.dependencyinjection.jwtbearerextensions?view=aspnetcore-7.0) 在这里也不难看出，这个extensions中主要是对AddJwtBearer()方法的重载。


```csharp
//(第一步)添加Authentication服务
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
  				//(第二步)通过JwtBearerExtensions添加一个AddJwtBearer 扩展方法
           .AddJwtBearer(options =>
           {
               options.TokenValidationParameters = new TokenValidationParameters
               {
                   ValidateLifetime = false,
                   ValidateAudience = false,
                   ValidateIssuer = false,
                   ValidateIssuerSigningKey = true,
                   IssuerSigningKey =
                       new SymmetricSecurityKey(
                           Encoding.UTF8.GetBytes(new JwtSymmetricKeySetting(configuration).Value
                               .PadRight(256 / 8, '\0')))
               };
           });

//除了在services中添加authentication服务，还需要添加authentication中间件
//(第三步)在Startup.Configure方法中添加authentication中间件
app.UseAuthentication();
```

一般我们可以通过一个服务扩展类进行扩展以方便后续进一步对服务进行增强或者修改。

```csharp
public static class AuthenticationExtension
{
    public static void AddCustomAuthentication(this IServiceCollection services,
      IConfiguration configuration)
    {
       services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
           .AddJwtBearer(options =>
           {
               options.TokenValidationParameters = new TokenValidationParameters
               {
                   ValidateLifetime = false, // 检查令牌是否未过期以及发行者的签名密钥是否有效
                   ValidateAudience = false, // 验证令牌的接收者是否有权接收
                   ValidateIssuer = false, // 验证生成令牌的服务器
                   ValidateIssuerSigningKey = true, // 验证令牌的签名
                   IssuerSigningKey =
                       new SymmetricSecurityKey(
                           Encoding.UTF8.GetBytes(new JwtSymmetricKeySetting(configuration).Value
                               .PadRight(256 / 8, '\0')))
               };
           });
    }
}
```

至此，JwtBearer验证功能配置完毕，下面通过具体代码来了解整个验证的流程。

## 四、验证流程与代码（JwtBearer验证方式）


...未完待续...
