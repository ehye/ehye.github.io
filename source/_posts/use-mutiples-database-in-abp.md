---
title: 在ABP中连接到多个数据库
date: 2019-12-10 11:27:07
categroies: 
tags:
    - ASP.NET Core
    - ABP
---

此套方法适用于有ABP表格的数据库<!-- more -->

---

1.添加连接字符串

打开`appsettings.json`添加第二个连接字符串

```json
"ConnectionStrings": {
    "Default": "server=***;database=***;user=***;password=***;",
    "SecondDb": "server=***;database=***;user=***;password=***;"
}
```

打开Core项目的`Consts`文件，添加对连接字符串的引用

```csharp
public const string SecondDbConnectionStringName = "SecondDb";
```

2.添加实体

和往常一样在`Core`项目中添加实体，实体中的字段可以只添加所需要的

Product.cs

```csharp
public class Product : Entity<int> {
    public string Name { get; set; }
}
```

3.添加DbContext

在`EntityFrameworkCore`项目中添加以下文件

SecondDbContext.cs

```csharp
public class SecondDbContext : AbpZeroDbContext<Tenant, Role, User, SecondDbContext>
{
    public virtual DbSet<Product> Product { get; set; }

    public SecondDbContext(DbContextOptions<SecondDbContext> options)
        : base(options)
    {
    }
}
```

SecondDbContextConfigurer.cs

```csharp
public static void Configure(DbContextOptionsBuilder<SecondDbContext> builder, string connectionString)
{
    builder.UseMySql(connectionString);
}

public static void Configure(DbContextOptionsBuilder<SecondDbContext> builder, DbConnection connection)
{
    builder.UseMySql(connection);
}
```

SecondDbContextFactory.cs

```csharp
public class SecondDbContextFactory: IDesignTimeDbContextFactory<SecondDbContext>
{
    public SecondDbContext CreateDbContext(string[] args)
    {
        var builder = new DbContextOptionsBuilder<SecondDbContext>();
        var configuration = AppConfigurations.Get(WebContentDirectoryFinder.CalculateContentRootFolder());

        SecondDbContextConfigurer.Configure(builder, configuration.GetConnectionString(Consts.SecondDbConnectionStringName));

        return new SecondDbContext(builder.Options);
    }
}
```

4.添加连接字符串解析服务

还是在`EntityFrameworkCore`项目中添加`MyConnectionStringResolver.cs`文件

MyConnectionStringResolver.cs

```csharp
public class MyConnectionStringResolver : DefaultConnectionStringResolver
{
    public MyConnectionStringResolver(IAbpStartupConfiguration configuration) 
        : base(configuration)
    {
    }

    public override string GetNameOrConnectionString(ConnectionStringResolveArgs args)
    {
        if (args["DbContextConcreteType"] as Type == typeof(SecondDbContext))
        {
            var configuration = AppConfigurations.Get(WebContentDirectoryFinder.CalculateContentRootFolder());
            return configuration.GetConnectionString(MultipleDbContextEfCoreDemoConsts.SecondDbConnectionStringName);
        }

        return base.GetNameOrConnectionString(args);
    }
}
```

然后在`EntityFrameworkModule.cs`更新`PreInitialize()`方法

```csharp
public class MultipleDbContextEfCoreDemoEntityFrameworkCoreModule : AbpModule
{
    public override void PreInitialize()
    {
        Configuration.ReplaceService<IConnectionStringResolver, MyConnectionStringResolver>();

        // Configure first DbContext
        Configuration.Modules.AbpEfCore().AddDbContext<FirstDbContext>(options =>
        {
            if (options.ExistingConnection != null)
            {
                FirstDbContextConfigurer.Configure(options.DbContextOptions, options.ExistingConnection);
            }
            else
            {
                FirstDbContextConfigurer.Configure(options.DbContextOptions, options.ConnectionString);
            }
        });

        // Configure second DbContext
        Configuration.Modules.AbpEfCore().AddDbContext<SecondDbContext>(options =>
        {
            if (options.ExistingConnection != null)
            {
                SecondDbContextConfigurer.Configure(options.DbContextOptions, options.ExistingConnection);
            }
            else
            {
                SecondDbContextConfigurer.Configure(options.DbContextOptions, options.ConnectionString);
            }
        });
    }
}
```
---

Ref

[How to use multiples databases in ABP Core Zero?](https://stackoverflow.com/questions/49243891/how-to-use-multiples-databases-in-abp-core-zero)