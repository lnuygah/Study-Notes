# 一、ASP.NET Core启动流程

## 1. 引言

对于ASP.NET Core应用程序来说，我们要记住非常重要的一点是：其本质上是一个独立的控制台应用，它并不是必需在[IIS](https://so.csdn.net/so/search?q=IIS&spm=1001.2101.3001.7020)内部托管且并不需要IIS来启动运行（而这正是ASP.NET Core跨平台的基石）。ASP.NET Core应用程序拥有一个内置的**Self-Hosted（自托管）**的**Web Server（Web服务器）**，用来处理外部请求。

![ASP.NET Core总体启动流程](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8yNzk5NzY3LTVlY2RmYzUyYzI4OGI2NmEucG5n?x-oss-process=image/format,png)

> 我们知道ASP.NET Core应用程序的启动主要包含三个步骤：

1. CreateDefaultBuilder()：创建IWebHostBuilder
2. Build()：IWebHostBuilder负责创建IWebHost
3. Run()：启动IWebHost

**所以，ASP.NET Core应用的启动本质上是启动作为宿主的WebHost对象。**
其主要涉及到两个关键对象`IWebHostBuilder`和`IWebHost`，它们的内部实现是ASP.NET Core应用的核心所在。下面我们就结合源码并梳理调用堆栈来一探究竟！



---



## 2. 宿主构造器：IWebHostBuilder

在启动`IWebHost`宿主之前，我们需要完成对`IWebHost`的创建和配置。而这一项工作需要借助`IWebHostBuilder`对象来完成的，ASP.NET Core中提供了默认实现`WebHostBuilder`。而`WebHostBuilder`是由WebHost的同名工具类（Microsoft.AspNetCore命名空间下）中的`CreateDefaultBuilder`方法创建的。

![CreateDefaultBuilder()调用堆栈](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8yNzk5NzY3LWNjNzU1NzBhMzc1ZTNlMDUucG5n?x-oss-process=image/format,png)

从上图中我们可以看出`CreateDefaultBuilder()`方法主要干了六件大事：

1. UseKestrel：使用Kestrel作为Web server。
2. UseContentRoot：指定Web host使用的content root（内容根目录），比如Views。默认为当前应用程序根目录。
3. ConfigureAppConfiguration：设置当前应用程序配置。主要是读取 appsettinggs.json 配置文件、开发环境中配置的UserSecrets、添加环境变量和命令行参数 。
4. ConfigureLogging：读取配置文件中的Logging节点，配置日志系统。
5. UseIISIntegration：使用IISIntegration 中间件。
6. UseDefaultServiceProvider：设置默认的依赖注入容器。

创建完毕`WebHostBuilder`后，通过调用`UseStartup()`来指定启动类，来为后续服务的注册及中间件的注册提供入口。



## 3. 宿主：IWebHost

在ASP.Net Core中定义了`IWebHost`用来表示Web应用的宿主，并提供了一个默认实现`WebHost`。宿主的创建是通过调用`IWebHostBuilder`的`Build()`方法来完成的。那该方法主要做了哪些事情呢，我们来看下面这张【ASP.NET Core启动流程调用堆栈】中的黄色边框部分

![ASP.NET Core启动流程调用堆栈](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8yNzk5NzY3LTk1NmEwNjE0MzcxMDMwNzkucG5n?x-oss-process=image/format,png)

其核心主要在于WebHost的创建，又可以划分为三个部分：

1. **构建依赖注入容器，初始通用服务的注册**：BuildCommonService();
2. 实例化WebHost：var host = new WebHost(...);
3. 初始化WebHost，也就是构建由中间件组成的请求处理管道：host.Initialize();



#### 3.1. 注册初始通用服务

`BuildCommonService`方法主要做了两件事：

1. 查找`HostingStartupAttribute`特性以应用其他程序集中的启动配置
2. 注册通用服务
3. 若配置了启动程序集，则发现并以`IStartup`类型注入到IOC容器中



#### 3.2. 创建IWebHost

```dotnet
public IWebHost Build()
{
    //省略部分代码
 
    var host = new WebHost(
        applicationServices,
        hostingServiceProvider,
        _options,
        _config,
        hostingStartupErrors);
    }
    
    host.Initialize();
 
    return host;
}
```



#### 3.3. 构建请求处理管道

请求管道的构建，主要是中间件之间的衔接处理。

> 而请求处理管道的构建，又包含三个主要部分：
>
> 1. 注册Startup中绑定的服务；
> 2. 配置IServer；
> 3. 构建管道

请求管道的构建主要是借助于`IApplicationBuilder`，相关类图如下：

![请求管道的构建](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8yNzk5NzY3LWI0YmNlZDhlNDk2NTlhY2QucG5n?x-oss-process=image/format,png)



## 4. 启动WebHost

WebHost的启动主要分为两步：

1. 再次确认请求管道正确创建
2. 启动Server以监听请求
3. 启动 HostedService



