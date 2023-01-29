---
title: 使用 ASP.Net Core Identity 和 JWT 实现认证与授权
date: 2023-01-28 13:47:22
tags:
    - Clean Architecture
    - ASP.Net Core Identity
    - JWT
---

利用 Clean Architecture 模板和 ASP.Net Core Identity 创建一个使用 JWT 验证的 Web API 项目<!-- more -->

---

## 创建数据表

1. 安装依赖包

    在 `Infrastructure` 项目中添加以下依赖

    ```shell
    dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
    dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
    ```

2. 新建 `User` 类，继承于 `IdentityUser`，使用 Guid 类型作为主键

    ```csharp
    public class User : IdentityUser<Guid>
    {
    }
    ```

3. 让 `ApplicationDbContext` 继承 `IdentityUserContext<User, Guid>`

    ```csharp
    public class ApplicationDbContext : IdentityUserContext<User, Guid>, IApplicationDbContext
    {
    }
    ```

4. 在 `OnModelCreating` 中，可自定义 Identity 所用到的表名，同时要显示地加上主键

    ```csharp
    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);
        
        builder.Entity<User>().ToTable("Users");
        builder.Entity<IdentityRole<Guid>>().ToTable("Roles").HasKey(x => x.Id);
        builder.Entity<IdentityUserRole<Guid>>().ToTable("UserRoles").HasKey(x => new { x.UserId, x.RoleId });
        builder.Entity<IdentityUserClaim<Guid>>().ToTable("UserClaims").HasKey(x => x.Id);
        builder.Entity<IdentityUserLogin<Guid>>().ToTable("UserLogins").HasKey(x => x.UserId);
        builder.Entity<IdentityUserToken<Guid>>().ToTable("UserTokens").HasKey(x => x.UserId);
    }
    ```

5. 在 `src/Infrastructure/ConfigureServices.cs` 中配置 Identity Service，使用自定义的用户类 `User`，和内置的角色类 `IdentityRole`，并使用 Guid 类型作为主键

    ```csharp
    services.AddIdentity<User, IdentityRole<Guid>>(options =>
        {
            options.Password.RequiredLength = 6;
            options.Password.RequireDigit = false;
            options.Password.RequireNonAlphanumeric = false;
            options.Password.RequireUppercase = false;
        })
        .AddEntityFrameworkStores<ApplicationDbContext>();
    ```

6. 在 `Program.cs` 中启用验证与授权

    ```csharp
    app.UseAuthentication();
    app.UseAuthorization();
    ```

7. 执行数据库迁移

    ```bash
    dotnet ef migrations add Init -s ..\Api\Api.csproj -o .\Persistence\Migrations\

    dotnet ef database update
    ```

最终生成以下数据表

- Roles
- UserClaims
- UserLogins
- UserRoles
- UserTokens
- Users

## 创建验证授权服务

### 配置 JWT 验证服务

- 在 `appsettings.json` 中配置 JWT 

    ```json
    "JWT": {
        "ValidAudience": "CleanIdentity",
        "ValidIssuer": "CleanIdentity"
      },
    ```

- 在 `src/Infrastructure/ConfigureServices.cs` 中启用 JWT 验证

    ```csharp
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
                ValidAudience = configuration["JWT:ValidAudience"],
                ValidIssuer = configuration["JWT:ValidIssuer"]
            };
        });
    ```

- 让 Swagger 可以使用 JWT

    ```csharp
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

### 实现 IIdentityService

1. 定义接口 `src/Application/Common/Interfaces/IIdentityService.cs` 

    ```csharp
    public interface IIdentityService
    {
        Task<(Result Result, string UserId)> CreateUserAsync(string userName, string password);

        Task<Result> CreateRoleAsync(string roleName, CancellationToken cancellationToken);

        Task<bool> CheckPassword(string userName, string password);
        
        Task<bool> IsInRoleAsync(string userId, string role);

        // ...
    }
    ```

2. 实现 `src/Infrastructure/Identity/IdentityService.cs`, 在这里调用 `UserManager` 和 `RoleManager` 的方法

    ```csharp
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

    ```csharp
    services.AddTransient<IIdentityService, IdentityService>();
    ```

### 实现 ITokenService

1. 定义接口 `src/Application/Common/Interfaces/ITokenService.cs` 

    ```csharp
    public interface ITokenService
    {
        Task<string> CreateToken(string userName);

        // ...
    }
    ```

2. 实现 `src/Api/Services/TokenService.cs`, 从这里生成 JWT

    ```csharp
    public class TokenService : ITokenService
    {
        private readonly IHttpContextAccessor _httpContextAccessor;
        private readonly IConfiguration _configuration;
        private readonly IDateTime _dateTime;
        private readonly UserManager<User> _userManager;

        public TokenService(
            IHttpContextAccessor httpContextAccessor,
            UserManager<User> userManager,
            IDateTime dateTime,
            IConfiguration configuration)
        {
            _httpContextAccessor = httpContextAccessor;
            _userManager = userManager;
            _dateTime = dateTime;
            _configuration = configuration;
        }

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
                issuer: _configuration["JWT:ValidIssuer"],
                audience: _configuration["JWT:ValidAudience"],
                expires: _dateTime.Now.AddDays(7),
                claims: authClaims
            );

            return new JwtSecurityTokenHandler().WriteToken(token);
        }
    }
    ```

3. 在 `src/Api/ConfigureServices.cs` 中注入 `ITokenService`

    ```csharp
    services.AddScoped<ITokenService, TokenService>();
    ```

## 实现接口

### 应用层(Application)

- RegistryCommand.cs

    ```csharp
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

    ```csharp
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

### 接口层(Api)

然后在 Api 项目中调用它们

- AuthenticateController.cs

    ```csharp
    namespace Api.Controllers;

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

---

资源

- [Clean Architecture Solution Template](https://github.com/jasontaylordev/CleanArchitecture)

- [Overview of ASP.NET Core authentication](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/)