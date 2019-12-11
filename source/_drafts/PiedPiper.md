---
title: piedpiper
date: 2019-12-10 11:27:07
categroies: 
tags:
---

```csharp
/// <summary>
/// 部门树
/// </summary>
/// <param name="tenantId"></param>
/// <returns></returns>
/// <remarks>token为管理员时租户为null，根据参数查询；token为用户时，限制按照其租户id查询</remarks>
public async Task<List<DepartmentDto>> GetDetpTree(int? tenantId)
{
    int tid = AbpSession.TenantId ?? tenantId.Value;
    string corpid = _tenantCache.GetOrNull(tid).TenancyName;
    var deptEntity = await _deptManager.GetByTenancyName(corpid);
    return new List<DepartmentDto> { ObjectMapper.Map<DepartmentDto>(deptEntity[0]) };
}

/// <summary>
/// 根据租户和部门获取人员详情
/// </summary>
/// <param name="input"></param>
/// <returns></returns>
public async Task<PagedResultDto<UserOutDto>> PostEmpInfo(UserInDto input)
{
    using (CurrentUnitOfWork.SetTenantId(input.TenantId))
    {
        var entities = await UserManager.GetUsersByDept(input.DepartmentId, input.NameSearch);
        var queryList = entities.Skip(input.SkipCount).Take(input.MaxResultCount);
        return new PagedResultDto<UserOutDto>(entities.Count, ObjectMapper.Map<List<UserOutDto>>(queryList));
    }
}
```

---

租户仓储

```cs
//Find a user by email for current tenant
var user = _userManager.FindByEmail("sampleuser@aspnetboilerplate.com");
```

```cs
//Switch to tenant 42
CurrentUnitOfWork.SetFilterParameter(AbpDataFilters.MayHaveTenant, AbpDataFilters.Parameters.TenantId, 42);

//Find a user by email for tenant 42
var user = _userManager.FindByEmail("sampleuser@aspnetboilerplate.com");
```

```cs
//Disabling MayHaveTenant filter, so we can reach all users
using (CurrentUnitOfWork.DisableFilter(AbpDataFilters.MayHaveTenant))
{
    //Now, we can search for a user name in all tenants
    var users = _userManager.Users.Where(u => u.UserName == "sampleuser").ToList();

    //Or we can add TenantId filter if we want to search for a specific tenant
    var user = _userManager.Users.FirstOrDefault(u => u.TenantId == 42 && u.UserName == "sampleuser");
}
```
