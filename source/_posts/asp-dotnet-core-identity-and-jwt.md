---
title: 使用 ASP.Net Core Identity 和 JWT 实现认证与授权
date: 2023-01-28 13:47:22
tags:
    - Clean Architecture
    - ASP.Net Core Identity
    - JWT
---

利用 Clean Architecture 模板搭配 ASP.Net Core Identity 创建一个使用 JWT 验证的 Web API 项目<!-- more -->

---

[项目地址](https://github.com/ehye/CleanIdentity)

## 使用 Clean Architecture

1. 安装模板 [Clean Architecture Solution Template](https://github.com/jasontaylordev/CleanArchitecture)

    ```cmd
    dotnet new install Clean.Architecture.Solution.Template

    dotnet new ca-sln
    ```

2. 删除配置文件中前端相关的配置信息

    - src/WebUI/WebUI.csproj
    - src/WebUI/appsettings.json
    - src/WebUI/Properties/launchSettings.json

3. 删除前端文件

    删除接口项目不要需要的前端文件夹 `ClientApp` `Pages` `wwroot`

    ```bash
    WebUI/
    ├── ClientApp *
    ├── Controllers
    ├── Filters
    ├── Pages *
    ├── Properties
    ├── Services
    └── wwwroot *
    ```

4. 删除依赖

    - Microsoft.AspNetCore.ApiAuthorization.IdentityServer
    - Microsoft.AspNetCore.Identity.UI
    - Microsoft.AspNetCore.SpaProxy
    - NSwag.MSBuild

5. 更新、增加依赖

    在 `Infrastructure` 项目中添加 `Microsoft.AspNetCore.Identity.EntityFrameworkCore` 和 `Microsoft.AspNetCore.Authentication.JwtBearer`

    ```cmd
    cd \src\Infrastructure
    dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
    dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
    ```

    在 `WebUI` 项目中添加 `Microsoft.EntityFrameworkCore.Design`

    ```cmd
    \src\WebUI
    dotnet add package Microsoft.EntityFrameworkCore.Design
    ```

## 生成 Identity 数据表

1. 新建 `User` 类，继承于 `IdentityUser`，使用 Guid 类型作为主键

    ```cs
    public class User : IdentityUser<Guid>
    {
        // Custom fields goes here
    }
    ```

2. 让 `ApplicationDbContext` 继承 `IdentityDbContext<TUser,TRole,TKey>`

    ```cs src/Infrastructure/Persistence/ApplicationDbContext.cs
    public class ApplicationDbContext : IdentityDbContext<User, IdentityRole<Guid>, Guid>, IApplicationDbContext
    {
    }
    ```

3. 在 `OnModelCreating()` 中，可自定义 Identity 所用到的表名

    ```cs src/Infrastructure/Persistence/ApplicationDbContext.cs
    protected override void OnModelCreating(ModelBuilder builder)
    {
        builder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly());

        base.OnModelCreating(builder);
        
        builder.Entity<ApplicationUser>().ToTable("Users");
        builder.Entity<IdentityRole>().ToTable("Roles");
        builder.Entity<IdentityRoleClaim<string>>().ToTable("RoleClaim");
        builder.Entity<IdentityUserRole<string>>().ToTable("UserRoles");
        builder.Entity<IdentityUserClaim<string>>().ToTable("UserClaims");
        builder.Entity<IdentityUserLogin<string>>().ToTable("UserLogins");
        builder.Entity<IdentityUserToken<string>>().ToTable("UserTokens");
    }
    ```

4. 在 `src/Infrastructure/ConfigureServices.cs` 中注入 Identity Service

    使用自定义的用户类 `User`，和默认的角色类 `IdentityRole`，并使用 `Guid` 类型作为主键；
    
    可以使用 options 参数来配置 Identity

    `AddEntityFrameworkStores()` 用于生成 Identity 使用的表

    ```cs src/Infrastructure/ConfigureServices.cs
    services.AddIdentity<User, IdentityRole<Guid>>(options =>
        {
            options.Password.RequiredLength = 4;
            options.Password.RequireDigit = false;
            options.Password.RequireNonAlphanumeric = false;
            options.Password.RequireUppercase = false;

            options.User.AllowedUserNameCharacters = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";

            options.Lockout.MaxFailedAccessAttempts = 7;
        })
        .AddEntityFrameworkStores<ApplicationDbContext>();
    ```

5. 生成数据库迁移

    语法为 `dotnet ef migrations add Init -s <ApiProjectFile> -c <DbContextClassName> -o <MigrationsFolder> --verbose`

    ```cmd
    cd src\Infrastructure\

    dotnet ef migrations add Init -s ..\WebUI\ -o .\Persistence\Migrations\ -v
    ```

    {% note info %} 

    可以使用 `dotnet ef migrations remove -s <ApiProjectFile> -c <DbContextClassName>` 移除迁移记录

    ```cmd
    cd src\Infrastructure\

    dotnet ef migrations remove -s ..\WebUI\ -c ApplicationDbContext --v
    ```
    {% endnote %}

执行 `ef database update ` 生成以下数据表

- RoleClaims
- Roles
- UserClaims
- UserLogins
- UserRoles
- UserTokens
- Users

## 创建验证授权服务

### 生成 JWT

1. 定义接口 `ITokenService.cs` 

    ```cs src/Application/Common/Interfaces/ITokenService.cs
    public interface ITokenService
    {
        Task<string> CreateToken(string userName);

        // ...
    }
    ```

2. 实现 `TokenService.cs`, 从这里生成 JWT

    ```cs src/WebUI/Services/TokenService.cs
    public async Task<string> CreateToken(string userName)
    {
        var authClaims = new List<Claim>
        {
            new Claim(ClaimTypes.Name, userName), 
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
        };

        var user = await _userManager.FindByNameAsync(userName);

        if (user is null)
        {
            throw new InvalidOperationException();
        }
        
        authClaims.Add(new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()));

        var roles = await _userManager.GetRolesAsync(user);

        authClaims.AddRange(roles.Select(role => new Claim(ClaimTypes.Role, role)));

        var token = new JwtSecurityToken(
            issuer: _configuration["Jwt:Issuer"],
            audience: _configuration["Jwt:Audience"],
            expires: _dateTime.Now.AddDays(7),
            claims: authClaims,
            signingCredentials: new SigningCredentials(
                new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_configuration["Jwt:Key"])),
                SecurityAlgorithms.HmacSha256Signature)
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
    ```

3. 在 `ConfigureServices.cs` 中注入 `ITokenService`

    ```cs src/WebUI/ConfigureServices.cs
    services.AddScoped<ITokenService, TokenService>();
    ```

### 验证 JWT

- 在 `appsettings.json` 中配置 JWT 

    ```json
    "Jwt": {
        "Audience": "Valid.Audience",
        "Issuer": "Valid.Issuer",
        "Key": "8yMDA4LzA2L2lkZW50aXR5L2NsYWltcy"
    },
    ```

- 在 `ConfigureServices.cs` 中启用 JWT 验证

    ```cs src/Infrastructure/ConfigureServices.cs
    services.AddAuthentication(options =>
        {
            options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
            options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
            options.DefaultScheme = JwtBearerDefaults.AuthenticationScheme;
        })
        .AddJwtBearer(options =>
        {
            options.SaveToken = true;
            options.RequireHttpsMetadata = false;
            options.TokenValidationParameters = new TokenValidationParameters()
            {
                ValidateIssuer = true,
                ValidateAudience = true,
                ValidAudience = configuration["JWT:Audience"],
                ValidIssuer = configuration["JWT:Issuer"]
            };
        });
    ```

- 让 Swagger 可以使用 JWT

    ```cs src/Infrastructure/ConfigureServices.cs
    services.AddOpenApiDocument(configure =>
    {
        configure.AddSecurity("JWT", Enumerable.Empty<string>(),
            new OpenApiSecurityScheme
            {
                Type = OpenApiSecuritySchemeType.ApiKey,
                Name = "Authorization",
                In = OpenApiSecurityApiKeyLocation.Header,
                Description = "Type into the textbox: Bearer {your JWT token}."
            });

        configure.OperationProcessors.Add(new AspNetCoreOperationSecurityScopeProcessor("JWT"));
    });
    ```

## 身份管理服务

实现 `IIdentityService`

1. 定义接口 `IIdentityService.cs` 

    ```cs src/Application/Common/Interfaces/IIdentityService
    public interface 
    {
        Task<(Result Result, string UserId)> CreateUserAsync(string userName, string password);

        Task<Result> CreateRoleAsync(string roleName, CancellationToken cancellationToken);

        Task<bool> CheckPassword(string userName, string password);
        
        Task<bool> IsInRoleAsync(string userId, string role);

        // ...
    }
    ```

2. 实现 `IdentityService.cs`, 在这里调用 `UserManager` 和 `RoleManager` 的方法

    ```cs src/Infrastructure/Identity/IdentityService.cs
    namespace Infrastructure.Identity;

    public class IdentityService : IIdentityService
    {
        private readonly UserManager<User> _userManager;
        private readonly RoleManager<IdentityRole<Guid>> _roleManager;
        private readonly IUserClaimsPrincipalFactory<User> _userClaimsPrincipalFactory;
        private readonly IAuthorizationService _authorizationService;

        public IdentityService(
            UserManager<User> userManager,
            RoleManager<IdentityRole<Guid>> roleManager,
            IUserClaimsPrincipalFactory<User> userClaimsPrincipalFactory,
            IAuthorizationService authorizationService)
        {
            _userManager = userManager;
            _roleManager = roleManager;
            _userClaimsPrincipalFactory = userClaimsPrincipalFactory;
            _authorizationService = authorizationService;
        }

        public async Task<(Result Result, string UserId)> CreateUserAsync(string userName, string password)
        {
            var user = new User { UserName = userName, Email = userName, };

            var result = await _userManager.CreateAsync(user, password);

            return (result.ToApplicationResult(), user.Id.ToString());
        }

        public async Task<Result> CreateRoleAsync(string roleName, CancellationToken cancellationToken)
        {
            var result = await _roleManager.CreateAsync(new IdentityRole<Guid>(roleName));

            return result.ToApplicationResult();
        }

        public async Task<bool> IsInRoleAsync(string userId, string role)
        {
            var user = _userManager.Users.SingleOrDefault(u => u.Id == Guid.Parse(userId));

            return user != null && await _userManager.IsInRoleAsync(user, role);
        }

        public async Task<bool> CheckPassword(string userName, string password)
        {
            var user = await _userManager.FindByNameAsync(userName);
            var result = user != null && await _userManager.CheckPasswordAsync(user, password);
            return result;
        }
    }
    ```

3. 在 `src/Infrastructure/ConfigureServices.cs` 中注入 `IdentityService`

    ```cs
    services.AddTransient<IIdentityService, IdentityService>();
    ```

4. 在 `Program.cs` 中启用验证与授权

    {% codeblock lang:cs mark:3,4 %}
    app.UseRouting();

    app.UseAuthentication();
    app.UseAuthorization();

    app.MapControllers();

    app.Run();
    {% endcodeblock %}

## 实现接口

### 应用层(Application)

实现用户注册和登录

- RegistryCommand.cs

    ```cs RegistryCommand.cs
    namespace Application.Authentication.Commands;

    public record RegistryCommand : IRequest<Result>
    {
        public string UserName { get; init; }

        public string Password { get; init; }
    }

    public class RegistryCommandHandler : IRequestHandler<RegistryCommand, Result>
    {
        private readonly IIdentityService _identityService;

        public RegistryCommandHandler(IIdentityService identityService)
        {
            _identityService = identityService;
        }

        public async Task<Result> Handle(RegistryCommand request, CancellationToken cancellationToken)
        {
            var r = await _identityService.CreateUserAsync(request.UserName, request.Password);

            return r.Result;
        }
    }
    ```

- LoginCommand.cs

    ```cs RegistryCommand.cs
    namespace Application.Authentication.Commands;

    public record LoginCommand : IRequest<string>
    {
        public string UserName { get; init; }

        public string Password { get; init; }
    }

    public class LoginHandler : IRequestHandler<LoginCommand, string>
    {
        private readonly IIdentityService _identityService;
        private readonly ITokenService _tokenService;

        public LoginHandler(IIdentityService identityService, ITokenService tokenService)
        {
            _identityService = identityService;
            _tokenService = tokenService;
        }

        public async Task<string> Handle(LoginCommand request, CancellationToken cancellationToken)
        {
            var checkPassword = await _identityService.CheckPassword(request.UserName, request.Password);
            if (checkPassword)
            {
                return await _tokenService.CreateToken(request.UserName);
            }

            return string.Empty;
        }
    }
    ```

### 接口层(WebUI)

然后在 WebUI 项目中使用它们

```cs AuthenticateController.cs
namespace WebUI.Controllers;

[AllowAnonymous]
public class AuthenticateController : ApiControllerBase
{
    [HttpPost]
    public async Task<IActionResult> Login([FromBody] LoginCommand command)
    {
        var token = await Mediator.Send(command);

        return string.IsNullOrEmpty(token) ? Unauthorized() : Ok(token);
    }

    [HttpPost]
    public async Task<Result> Registry([FromBody] RegistryCommand command)
    {
        return await Mediator.Send(command);
    }
}
```

## 访问接口

- 登录获得 Token

    ```http Request
    POST /api/authenticate/login HTTP/1.1
    Host: localhost:5001
    accept: application/octet-stream
    Content-Type: application/json
    Content-Length: 77

    {
      "username": "administrator@localhost",
      "password": "Administrator1!"
    }
    ```

- 访问需要验证的 API

    ```http Request
    GET /api/TodoLists HTTP/1.1
    Host: localhost:5001
    Authorization: Bearer xxxxxx
    ```

---

资源

- [Clean Architecture Solution Template](https://github.com/jasontaylordev/CleanArchitecture)

- [Overview of ASP.NET Core authentication](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/)

- [Change the primary key type - Identity model customization in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/customize-identity-model?view=aspnetcore-7.0#change-the-primary-key-type)

