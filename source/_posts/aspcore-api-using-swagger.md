---
title: 如何在ASP.NET Core(ABP)项目中生成自定义的Swagger文档
date: 2019-08-29 15:08:42
categories: Coding
tags:
    - aspnetboilerplate(ABP)
    - ASP.NET Core
    - Swagger
---

后端开发人员提供一个书写良好的Swagger文档页面可有效提高与前端对接的效率<!-- more -->

---

# 书写XML注释

> MSDOC上一个较为完整的例子

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
public ActionResult<TodoItem> Create(TodoItem item)
{
    _context.TodoItems.Add(item);
    _context.SaveChanges();

    return CreatedAtRoute("GetTodo", new { id = item.Id }, item);
}
```

![swagger summary](https://github.com/aspnet/AspNetCore.Docs/raw/master/aspnetcore/tutorials/web-api-help-pages-using-swagger/samples/3.0/TodoApi.Swashbuckle/sample_images/xml-comments-extended.png)

# 生成XML

1. 在【解决方案管理器】右击**Application**项目，选择**属性**
2. 在【生成】页面，勾选**XML文档文件**
3. 在【取消显示警告】一栏中增加**1591;**

# 配置Swagger参数(4.0.0)

1. 打开*Web.Host.Startup.cs*,在*ConfigureServices*方法中增加下行

    ```csharp
    options.IncludeXmlComments(Path.Combine(AppContext.BaseDirectory, "<XML File Name>"));
    ```

2. 在*Configure*方法中自定义你的Swagger页面样式

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

---

参考

[Swagger UI Integration](https://aspnetboilerplate.com/Pages/Documents/Swagger-UI-Integration)

[Get started with Swashbuckle and ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-swashbuckle?view=aspnetcore-2.2&tabs=visual-studio#customize-and-extend)
