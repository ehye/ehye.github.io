---
title: Serilog for ASP.NET Core 
date: 2023-02-14 09:34:28
categories: Note
tags:
    - Serilog
---

Using Serilog logging for ASP.NET Core.<!--more-->

---

# Logging during startup

Use `Serilog.Extensions.Hosting` for logging framework's internal operations.

```cmd
dotnet add package Serilog.Extensions.Hosting
```

Use `CreateBootstrapLogger()` for `appsettings.json` configuration and dependency injection logging

```cs Program.cs https://github.com/serilog/serilog-aspnetcore#instructions
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .WriteTo.Console()
    .CreateBootstrapLogger();

try
{
    Log.Information("Starting web application");

    var builder = WebApplication.CreateBuilder(args);

    var app = builder.Build();
    app.UseHsts();
    app.UseStaticFiles();
    app.UseRouting();
    app.MapControllers();

    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application terminated unexpectedly");
}
finally
{
    Log.CloseAndFlush();
}
```

# Add UseSerilog()

Add packages

```cmd
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
```

Add `UseSerilog()` to the host builder

```cs 
const string OutputTemplate = "[{Timestamp:HH:mm:ss} {Level:u3}] [{SourceContext}] {Message:lj}{NewLine}{Exception}";

builder.UseSerilog((_, _, cfg) => cfg
    .MinimumLevel.Debug()
    .MinimumLevel.Override("<project_name_space>", LogEventLevel.Warning)
    .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
    .MinimumLevel.Override("Microsoft.Hosting.Lifetime", LogEventLevel.Information)
    .MinimumLevel.Override("System", LogEventLevel.Information)
    .WriteTo.Console(outputTemplate: OutputTemplate)
    .WriteToFile(builderConfiguration.GetValue<string>("LogFolder") ?? string.Empty));
```

Use `{SourceContext}` to log the namespace.

## Write to different log files by level

To create log by different logEvent, implement methond `WriteToFile()`

```cs
private static LoggerConfiguration WriteToFile(this LoggerConfiguration loggerConfiguration, string logFolder)
{
    var logEventLevels = new[]
    {
        LogEventLevel.Debug,
        LogEventLevel.Information,
        LogEventLevel.Warning,
        LogEventLevel.Error
    };

    foreach (var logEventLevel in logEventLevels)
    {
        loggerConfiguration.WriteTo.Logger(configuration =>
        {
            configuration.Filter.ByIncludingOnly(logEvent => logEvent.Level == logEventLevel)
                .WriteTo.File(path: $"{logFolder}\\{logEventLevel}-.log",
                    outputTemplate: OutputTemplate,
                    rollingInterval: RollingInterval.Day,
                    fileSizeLimitBytes: 1 * 1024 * 1024, // 1MB
                    retainedFileCountLimit: 2,
                    rollOnFileSizeLimit: true,
                    shared: true,
                    flushToDiskInterval: TimeSpan.FromSeconds(1));
        });
    }

    return loggerConfiguration;
}
```

When setting `rollingInterval`, the log files will append current date to the file name. e.g: `Information-20230101.log`

# Request logging

Use `UseSerilogRequestLogging()` for request logging

```cs
app.UseSerilogRequestLogging(options =>
{
    // // Emit debug-level events instead of the defaults
    options.GetLevel = (_, _, _) => LogEventLevel.Debug;
    
    // Attach additional properties to the request completion event
    options.EnrichDiagnosticContext = async (diagnosticContext, httpContext) =>
    {
        diagnosticContext.Set("RequestHost", httpContext.Request.Host.Value);
        diagnosticContext.Set("RequestScheme", httpContext.Request.Scheme);
        diagnosticContext.Set("QueryString", httpContext.Request.QueryString);
    
        var requestBody = await GetRequestBody(httpContext.Request);
        diagnosticContext.Set("RequestBody", requestBody);
    };
    
    options.MessageTemplate = 
        "{RequestMethod} {RequestBody}{RequestPath}{QueryString} responded {StatusCode} in {Elapsed:0.0000} ms";
});
```

Add `UseSerilogRequestLogging()` in Startup.cs before any handlers whose activities should be logged, by this way can also reduce unnesseray log

```cs
private static async Task<string> GetRequestBody(HttpRequest httpRequest)
{
    string requestBody = string.Empty;

    if (httpRequest.Path.ToString().ContainsAny(nameof(AuthenticateController.Login).Slugify(),
            nameof(AuthenticateController.Registry).Slugify()))
    {
        requestBody = "[Redacted] Contains Sensitive Information. ";
    }
    else
    {
        using var reader = new HttpRequestStreamReader(httpRequest.Body, Encoding.UTF8);
        var payload = await reader.ReadToEndAsync();
        if (!string.IsNullOrEmpty(payload))
        {
            var json = JsonSerializer.Deserialize<object>(payload);
            requestBody = $"{JsonSerializer.Serialize(json)} ";
        }
    }

    return requestBody;
}
```

---

[Serilog.AspNetCore](https://github.com/serilog/serilog-aspnetcore)
[Serilog : Log to different files - Stack Overflow](https://stackoverflow.com/questions/38481227/serilog-log-to-different-files)