部门树（左数右表）

appservice

```csharp
/// <summary>
/// 部门树
/// </summary>
/// <returns></returns>
public async Task<List<DepTreeOutDto>> GetDetpTreeWithEmp(string corpid, int tenantid)
{
    using (CurrentUnitOfWork.SetTenantId(tenantid))
    {
        var deptEntity = await _deptManager.GetDeptById(corpid);
        var deptDto = ObjectMapper.Map<DepTreeOutDto>(deptEntity);
        return new List<DepTreeOutDto>() { deptDto };
    }
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

manager

```csharp
public async Task<Department> GetDeptById(string corpid)
{
    var allDept = await _deptRepository.GetAll()
        .Where(d => d.CorpId == corpid).ToListAsync();
    return allDept.FirstOrDefault();
}

/// <summary>
/// 根据部门取人员
/// （用于部门树的人员列表字段）
/// </summary>
/// <param name="deptid"></param>
/// <param name="namesearch"></param>
/// <returns></returns>
public async Task<List<User>> GetUsersByDept(long deptid, string namesearch)
{
    return await UserRepository.GetAll()
        .Where(u => u.DepartmentId == deptid)
        .WhereIf(!string.IsNullOrEmpty(namesearch), u => u.Name.Contains(namesearch))
        .ToListAsync();
}
```
