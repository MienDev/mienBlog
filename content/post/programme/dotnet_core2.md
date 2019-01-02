---
title: "汇总：.NET Core 2.0 及 asp.net core 2.0 正式发布"
date: 2017-08-15T17:53:24+08:00
description: ""
thumbnail: ""
include_toc: true
categories:
  - "mirco_services"
  - "programme"
tags:
  - "programme"
  - ".net core"
---

> 2017年8月14日，.NET Core 及 asp.net core 同时发布 2.0 正式版。这是 .net 在开源之后发布的第二个大版本，也代表着 .net core 特性趋于稳定，可以用于生产环境（虽然很多人已经将 1.1 之后的版本用于生产环境了）。
>
> 本次一同发布的包括： NetStandard 2， .NET Core, asp.net core, Visual Studio 2017 Version 15.3, Entity Framework Core 2.0， WCF 更新。
>
> 本文综合总结了本次更新的主要内容。

<!--more-->

## 一、.NET Standard 2.0

> [.NET Standard](http://link.zhihu.com/?target=https%3A//docs.microsoft.com/en-us/dotnet/standard/net-standard) 是指 .NET 生态下的跨平台统一接口标准，具体实现包括 .NET Framework , .NET Core, Mono, Xamarin 等。

- .NET Standard 2.0 支持近 **3.2万**个API, 是 .NET Standard 1.x (1.3万) 的两倍还要多。
- 引入兼容模式，允许引用原先面向 .NET Framework 的依赖（但不支持 WPF）， 能兼容 目前NuGet市场中 70% 的包。
- 多平台支持，包括： .NET Framework 4.6.1 , .NET Core 2.0 , Mono 5.4 , Xamarin 。

![img](https://pic2.zhimg.com/80/v2-6444b59681a244ee15aa331c7bdc5d91_hd.jpg)



## 二、.NET Core

- 速度更快（[具体参见](http://link.zhihu.com/?target=https%3A//blogs.msdn.microsoft.com/dotnet/2017/06/07/performance-improvements-in-net-core/)）
- 更多的向接口迁移到 NETStandard2。
- 支持更多系统（多个发行版） SUSE， Debian， Fedora， MacOS，Docker 等。
- Visual Studio for Mac 支持 .net core开发

![img](https://pic3.zhimg.com/80/v2-222df9ebe7b58d7be079ed14585b3f2e_hd.jpg)

详细内容[参见.NET Core 2.0.0 Release Notes](http://link.zhihu.com/?target=https%3A//github.com/dotnet/core/blob/master/release-notes/2.0/2.0.0.md)。



## 三、[asp.net](http://link.zhihu.com/?target=http%3A//asp.net/) core

**3.1 Breaking Changes | 不兼容更新：**

- Authentication 鉴权方式改变 （[Auth 2.0 Changes / Migration](http://link.zhihu.com/?target=https%3A//github.com/aspnet/Announcements/issues/262)）
- 统一 Cookie设置的接口 （[Unifying API for configuring cookie settings](http://link.zhihu.com/?target=https%3A//github.com/aspnet/Announcements/issues/257)）
- EF Core 2.0中的 DbContext 发现机制改变 （[design-time DbContext discovery changes](http://link.zhihu.com/?target=https%3A//github.com/aspnet/Announcements/issues/258)）
- 本地化扩展 中的某些资源管理类需要传入 ILogger （[Localization: Some ResourceManager classes now require ILogger in constructor](http://link.zhihu.com/?target=https%3A//github.com/aspnet/Announcements/issues/260)）

> 关于Authentication 的改变，统称为 Auth 2.0。主要是 **IAuthenticationManager** (即，httpContext.Authentication)**被废弃**，改为 **IAuthenticationService** 。

在使用时，

```csharp
// 1.使用新的扩展
using Microsoft.AspNetCore.Authentication;

// 2.移出 .Authentication 即可
context.Authentication.AuthenticateAsync 改为 context.AuthenticateAsync
context.Authentication.ChallengeAsync 改为 context.ChallengeAsync
```

在配置时，

原来是在Configure()中的 UseXyzAuthentication() 方式，Auth 2.0中变更为在ConfigureService()中 AddXyz()。

```csharp
public void ConfigureServices(IServiceCollection services) {
    services.AddIdentity<ApplicationUser, IdentityRole>().AddEntityFrameworkStores();

    // If you want to tweak identity cookies, they no longer are part of identityOptions
    services.ConfigureApplicationCookie(o => o.LoginPath = new PathString("/login");
    services.AddAuthentication()
                .AddFacebook(o =>
                {
                    o.AppId = Configuration["facebook:appid"];
                    o.AppSecret = Configuration["facebook:appsecret"];
                });
}

public void Configure(IApplicationBuilder app, ILoggerFactory loggerfactory) {
    app.UseAuthentication();
}
```

原来配制时需引入了多个 Authentication 中间件，如 UseIdentity(), UseFacebookAuthentication(), **在Auth 2.0中只有唯一的中间件，所有schema都在ConfigureServices 时注册。**即在Configure中只需要使用 UseAuthentication() 即可，不再需要UseIdentity() (使用这个时，实际在背后重复调用了4次 UseCookie() )。



> 关于 统一 Cookie设置的接口，是指在对 Session, Cookie authentication, Antiforgery, MVC 等进行配置时，写法上的变化。

```js
{
            // 原来的，废弃
            options.CookieName = "AuthCookie";
            options.CookieDomain = "contoso.com";
            options.CookiePath = "/";
            options.CookieHttpOnly = true;
            options.CookieSameSite = SameSiteMode.Lax;
            options.CookieSecure = CookieSecurePolicy.Always;

            // 新的接口
            options.Cookie.Name = "AuthCookie";
            options.Cookie.Domain = "contoso.com";
            options.Cookie.Path = "/";
            options.Cookie.HttpOnly = true;
            options.Cookie.SameSite = SameSiteMode.Lax;
            options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
}
```



> 其中关于 EF Core 2.0中的 DbContext 发现，在1.0中 是通过 Startup.ConfigureServices 来进行发现的，2.0中 是通过 静态的 IWebHost 来发现。需要改写下 Program.cs 。

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        var host = BuildWebHost(args);

        host.Run();
    }

    // Tools will use this to get application services
    public static IWebHost BuildWebHost(string[] args) =>
        new WebHostBuilder()
            .UseKestrel()
            .UseContentRoot(Directory.GetCurrentDirectory())
            .UseIISIntegration()
            .UseStartup<Startup>()
            .Build();
}
```



**3.2 重要升级更新：**

- 提供了一种以页面为中心的开发模式，**“Razor Pages” 。**

**Razor Pages，是一种页面优先的结构，可更聚焦于用户界面，并使用PageModel来简化服务端的编程** （之前的MVC可以理解为是以Controller 中的 Action 为中心）。

在使用上，在默认的MVC配制已经支持Razor Pages， 只需要在页面顶部中添加 **@page** 指令即可以表明这是一个独立的Razor页面。 在页面中可以使用 HtmlHelpers, TagHelpers, HttpContext.User等其它内容。

如果需要针对页面写一些方法，可以使用 **@functions** { /*code goes here*/} 来完成定义。



```csharp
@page

@functions {

  public string FormatDate(DateTime theTime) {
    return theTime.ToString("d");
  }

}

<html>
	<body>
		<h2>The server-local time now is:</h2>

		<p>@FormatDate(DateTime.Now)</p>
	</body>
</html>
```



当需要更复杂场景时的支持时，可以通过**PageModel** 来完成。

```csharp
// 文件名: newRazorPage.cshtml
@page
@model MyFirstRazorPage.Pages.NowModel

<html>
	<body>
		<h2>This page was last updated:</h2>

		<p>@Model.LastModified</p>
	</body>
</html>
```



```csharp
// 文件名： newRazorPage.cshtml.cs
namespace MyFirstRazorPage.Pages
{
  public class NowModel : PageModel
  {

    private IFileProvider _FileProvider;

    public NowModel(PhysicalFileProvider fileProvider)
    {
      _FileProvider = fileProvider;
      LastModified = _FileProvider.GetFileInfo("Pages/Now.cshtml").LastModified.LocalDateTime;
    }
    public DateTime LastModified { get; set; }

    public void OnGet() { }

  }
}
```



**3.3 其它更新内容：**

- 更新项目模板以及 SPA 模板

> 通过 **JavaScript Service** 实现对 NodeJS的支持，你可以在 [asp.net core](http://link.zhihu.com/?target=http%3A//asp.net/) 的项目中创建SPA项目内置模板包括 Angular, React.js 等（Visual Studio 及 dotnet-cli 都已经更新项目模板）。

- Monitor and Profile with No Code Changes and Application Insights
- 在 Razor 中支持 C# 7.1
- 简化自服务(Application Host)配制
- Razor views : 在项目编译发布时预先编译。
- aspnet/Identity: Roles Is Optional at the Microsoft.Extensions.Identity layer
- Microsoft.Data.Sqlite : 改进小数的支持（Sqlite 没有小数类型）
- aspnet/SignalR: 降级为 1.0.0-alpha1-*



## 四、EntityFramework Core

EF core 2.0 包含以下的改进特性：

- 改进的 LINQ 转译（一些之前在client-side进行的查询，现在转为SQL来查询）
- Like 查询操作符（ where **EF.Functions.Like(c.Name, "a%")** ）
- 自有实体及**分表**
- **全局查询过滤器**
- DbContext 连接池
- 参数化直接拼接SQL 的查询

![img](https://pic3.zhimg.com/80/v2-a079da6c7c14784f4f75fce7e0a8176a_hd.jpg)



## 五、从1.X向2.0迁移

[Migrating from ASP.NET Core 1.x to ASP.NET Core 2.0](http://link.zhihu.com/?target=https%3A//docs.microsoft.com/en-us/aspnet/core/migration/1x-to-2x/)



## 六、如何评价 .net core 2.0 的发布

看这里 ： [知乎问答：怎么看待.net core 2.0发布？](https://zhihu.com/question/63934091/answer/214923292)