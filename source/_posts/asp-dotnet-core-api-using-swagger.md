---
title: 如何在ASP.NET Core(ABP)项目中生成自定义的Swagger文档
date: 2019-08-29 15:08:42
categories: Note
tags:
    - aspnetboilerplate(ABP)
    - ASP.NET Core
    - Swagger
---

后端开发人员提供一个书写良好的Swagger文档页面可有效提高与前端对接的效率<!-- more -->

---

## 安装Nuget包

对**core**项目安装`Swashbuckle.AspNetCore`

{% asset_img install-nuget-swagger.png %}

然后在`Startup.cs`中的`ConfigureServices`方法增加配置

```csharp
services.AddMvc();

services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });
});
```

以及`Configure`方法中

```csharp
app.UseSwagger();

app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API V1");
});
```

## 书写XML注释

> 微软官方文档上一个较为完整的例子

```csharp
/// <summary>
/// Creates a TodoItem.
/// </summary>
/// <remarks>
/// Sample request:
///
///     POST /Todo
///     {
///        "id": 1,
///        "name": "Item1",
///        "isComplete": true
///     }
///
/// </remarks>
/// <param name="item"></param>
/// <returns>A newly created TodoItem</returns>
/// <response code="201">Returns the newly created item</response>
/// <response code="400">If the item is null</response>
[HttpPost]
[ProducesResponseType(201)]
[ProducesResponseType(400)]
// [Obsolete]
public ActionResult<TodoItem> Create(TodoItem item)
{
    _context.TodoItems.Add(item);
    _context.SaveChanges();

    return CreatedAtRoute("GetTodo", new { id = item.Id }, item);
}
```

{% asset_img xml-comments-extended.png %}

## 在swagger文档上显示注释

1. 在【解决方案管理器】右击**Application**项目，选择**属性**
2. 在【生成】页面，勾选**XML文档文件**
3. 在【取消显示警告】一栏中增加**1591;**

## 配置Swagger参数(4.0.0)

1. 打开`Startup.cs`,在`ConfigureServices`方法中增加下行

    ```csharp
    options.IncludeXmlComments(Path.Combine(AppContext.BaseDirectory, "<XML File Path>"));
    ```

2. 在*Configure*方法中自定义Swagger页面样式

    ```csharp
    options.ShowExtensions();
    options.EnableValidator();
    // 将选择的接口地址连接到URL，方便与他人定位
    options.EnableDeepLinking();
    // 启用过滤器（大小写敏感）
    options.EnableFilter();
    // 展开级别
    options.DocExpansion(Swashbuckle.AspNetCore.SwaggerUI.DocExpansion.None);
    // 右侧显示方法名
    options.DisplayOperationId();

    /* 启用下面代码，生成适合用于对接的Swagger页面 */

    // 默认显示model
    options.DefaultModelRendering(Swashbuckle.AspNetCore.SwaggerUI.ModelRendering.Model);
    // 模型展开到2级
    options.DefaultModelExpandDepth(2);
    // 隐藏model列表
    options.DefaultModelsExpandDepth(-1);
    // 隐藏 [Try it]
    options.SupportedSubmitMethods();
    ```

## 增加折叠块注释

新增一个过滤器

```csharp
public class TagDescriptionsDocumentFilter : IDocumentFilter
{
    public void Apply(SwaggerDocument swaggerDoc, DocumentFilterContext context)
    {
        swaggerDoc.Tags = new[]
        {
            new Tag { Name = "Admin", Description = "管理员" },
            new Tag { Name = "User", Description = "用户" }
        };
    }
}
```

然后在`Startup.cs > ConfigureServices > AddSwaggerGen`里添加

```csharp
options.DocumentFilter<TagDescriptionsDocumentFilter>();
```

---

参考

[Swagger UI Integration - ASP.NET Boilerplate](https://aspnetboilerplate.com/Pages/Documents/Swagger-UI-Integration)

[Get started with Swashbuckle and ASP.NET Core | Microsoft Docs](https://docs.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-swashbuckle?view=aspnetcore-2.2&tabs=visual-studio#customize-and-extend)

[Swashbuckle.AspNetCore On GitHub](https://github.com/domaindrivendev/Swashbuckle.AspNetCore)
