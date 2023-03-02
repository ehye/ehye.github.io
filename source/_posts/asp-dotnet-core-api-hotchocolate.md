---
title: GraphQL query in .NET Core
date: 2023-03-02 16:40:27
categories: Note
tags:
    - GraphQL
    - HotChocolate
---

使用 HotChocolate 在 ASP.NET Core Web API 项目中提供 GraphQL 查询服务<!-- more -->

---

# 添加依赖

.NET 平台上的 GraphQL 服务有 GraphQL.net 和 HotChocolate

> HotChocolate 提供了更多的特性和扩展，比如过滤、排序、分页、订阅、批量操作等，并且可以自动将 .NET 类型转换为 GraphQL 类型
> 如果想要更多的控制和灵活性，可以选择 GraphQL.net；如果想要更多的便利和功能，则可以选择 HotChocolate

配置 HotChocolate 会更简单一些

```bash
dotnet add package HotChocolate.Data
dotnet add package HotChocolate.AspNetCore
dotnet add package HotChocolate.Abstractions
```

# 实现查询

- 在 Persistence 层创建 `TodoListQuery.cs`，在这里实现需要的查询

```cs
public class TodoListQuery
{
    [UsePaging(IncludeTotalCount = true, DefaultPageSize = 10)]
    public IQueryable<TodoList> GetTodoLists([Service] ApplicationDbContext dbContext)
        => dbContext.TodoLists.AsQueryable();
    
    public IQueryable<TodoList> GetTodoList(int id, [Service] ApplicationDbContext dbContext)
        => dbContext.TodoLists.Include(x => x.Items).Where(x => x.Id == id);

    // ...
}
```

Use `IncludeTotalCount = true` to show the total count of the output result.

Use `int id` as input param.

- 若使用 MediatR，则在 API 层传递 command，并实现相应的 handler

```cs
public class TodoListQuery
{
    [UsePaging(IncludeTotalCount = true, DefaultPageSize = 10)]
    [UseFiltering]
    public async Task<List<TodoListDto>> GetTodoLists([Service] IMediator mediator)
        => await mediator.Send(new GetTodosQueryGraph(null));
    
    [UsePaging(IncludeTotalCount = true, DefaultPageSize = 10)]
    public async Task<List<TodoListDto>> GetTodoLists(GetTodosQueryGraph query, [Service] IMediator mediator)
        => await mediator.Send(query);

    // ...
}
```

```cs
public record GetTodosQueryGraph(string? Title) : IRequest<List<TodoListDto>>;

public class GetTodosQueryGraphHandler : IRequestHandler<GetTodosQueryGraph, List<TodoListDto>>
{
    private readonly IApplicationDbContext _context;
    private readonly IMapper _mapper;

    public GetTodosQueryGraphHandler(IApplicationDbContext context, IMapper mapper)
    {
        _context = context;
        _mapper = mapper;
    }

    public async Task<List<TodoListDto>> Handle(GetTodosQueryGraph request, CancellationToken cancellationToken)
    {
        return await _context.TodoLists
            .AsNoTracking()
            .Where(x => x.Title.Contains(request.Title!))
            .ProjectTo<TodoListDto>(_mapper.ConfigurationProvider)
            .OrderBy(t => t.Title)
            .ToListAsync(cancellationToken);
    }
}
```

# 注入配置

- 注入 GraphQL 服务配置

```cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddGraphQLServer()
    .AddQueryType<TodoListQuery>()
    .AddFiltering();
```

仅在开发环境下启用 Banana Cake Pop GraphQL IDE

```cs 
app.MapGraphQL().WithOptions(new GraphQLServerOptions()
{
    Tool = { Enable = builder.Environment.IsDevelopment() }
});

app.Run();
```

# 测试查询

访问 `/graphql` 进行测试

- request

  ```graphql
  query TodoLists {
    todoLists(first: 3, where: {title: {contains: "occaecat"}}){
      totalCount
      nodes{
        id
        title
      }
      pageInfo{
        hasPreviousPage
        startCursor
        endCursor
        hasNextPage
      }
    }  
  }
  ```

- repsonse

  ```json
  {
    "data": {
      "todoLists": {
        "totalCount": 18,
        "nodes": [
          {
            "id": 3,
            "title": "occaecat m"
          },
          {
            "id": 4,
            "title": "occaecat rm"
          },
          {
            "id": 5,
            "title": "occaecat; rm"
          }
        ],
        "pageInfo": {
          "hasPreviousPage": false,
          "startCursor": "MA==",
          "endCursor": "Mg==",
          "hasNextPage": true
        }
      }
    }
  }
  ```

---

[Getting started with GraphQL and HotChocolate | Microsoft Learn](https://learn.microsoft.com/en-us/shows/on-net/getting-started-with-hotchocolate)

[HotChocolate - An Introduction to GraphQL for ASP.NET Core](https://www.jetbrains.com/dotnet/guide/tutorials/dotnet-days-online-2020/hotchocolate-an-introduction-to-graphql-for-aspnet-core)

[Introduction - Hot Chocolate - ChilliCream GraphQL Platform](https://chillicream.com/docs/hotchocolate/v13)
