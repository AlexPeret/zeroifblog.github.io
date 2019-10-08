# Introduction

This article describes how to compile a WebSharper client-server application on .NET Core 3.0 running on Linux.

# Environment
- openSuse (Linux) Leap 15.1 running on Oracle VM VirtualBox.
- .NET Core 3.0 (this is the only version installed)
- VS Code
- WebSharper templates' version: 4.5.19.332

# Instructions to build the application
## Installing WebSharper templates
1. I've created a directory to place my projects at ~/programs/websharper
2. at a shell (bash), run the following **dotnet CLI command**

`> dotnet new -i WebSharper.Templates`

3. create a sample client-server application using the WebSharper template
```
> cd ~/programs/websharper
> dotnet new websharper-web -lang f# -n ClientServerSample1
```

3.1. by printing the installed templates shows
```
> dotnet new -u
  WebSharper.Templates
    Details:
      NuGetPackageId: WebSharper.Templates
      Version: 4.5.19.332
      Author: IntelliFactory
    Templates:
      WebSharper 4 Client-Server Application (websharper-web) C#
      WebSharper 4 Client-Server Application (websharper-web) F#
      WebSharper 4 Library (websharper-lib) C#
      WebSharper 4 Library (websharper-lib) F#
      WebSharper 4 Html Site (websharper-html) C#
      WebSharper 4 Html Site (websharper-html) F#
      WebSharper 4 Single Page Application (websharper-spa) C#
      WebSharper 4 Single Page Application (websharper-spa) F#
    Uninstall Command:
      dotnet new -u WebSharper.Templates
```

4. run VS Code and open the '~/programs/websharper/ClientServerSample1' folder [Ctrl+K Ctrl+O]
4.1. open a *Terminal* (View > Terminal) and remove the 'Microsoft.AspNetCore.All' package

`> dotnet remove package Microsoft.AspNetCore.All`

**Note**: this package is autoload on .NET Core 3.0 when 'Microsoft.NET.Sdk.Web' is used

4.2. open *Explorer* tab (Ctrl+Shift+E) and edit the ClientServerSample1.fsproj file
replace the following line

`    <TargetFramework>netcoreapp2.0</TargetFramework>`

by

`    <TargetFramework>netcoreapp3.0</TargetFramework>`


4.3. save the file.

5. edit the wsfsc *runtimeconfig* to reference .NET Core 3.0.
Edit the following file (I'm using the Vim editor here, but you can use your prefered editor): 

`vi ~/.nuget/packages/websharper.fsharp/4.5.19.349/tools/netstandard2.0/wsfsc.runtimeconfig.json`

```
  "runtimeOptions": {
    "tfm": "netcoreapp2.0",
    "framework": {
      "name": "Microsoft.NETCore.App",
      "version": "2.0.0"
    }
  }
}
```

change the "version" attribute to '3.0.0'
```
  "runtimeOptions": {
    "tfm": "netcoreapp2.0",
    "framework": {
      "name": "Microsoft.NETCore.App",
      "version": "3.0.0"
    }
  }
}
```

**Note**: this change was required as the task was exiting with code 150, during build

6. back to VS Code's Terminal, run
```
> dotnet build
> dotnet run
```

7. load the page in your prefered browser: http://localhost:5000/

**Note**: this might failed due the following exception

```
fail: Microsoft.AspNetCore.Server.Kestrel[13]
      Connection id "0HLQBUP9N4GNE", Request id "0HLQBUP9N4GNE:00000001": An unhandled exception was thrown by the application.
System.InvalidOperationException: Synchronous operations are disallowed. Call WriteAsync or set AllowSynchronousIO to true instead.
```

## Quick fixing the code to run the application
1. open the Startup.fs file and open the Kestrel namespace

`open Microsoft.AspNetCore.Server.Kestrel.Core`

2. change the *ConfigureServices* method to look like the following:
```
    member this.ConfigureServices(services: IServiceCollection) =
        services.Configure<KestrelServerOptions>(fun (options:KestrelServerOptions) ->
            options.AllowSynchronousIO <- true
        )
        |> ignore

        services.AddSitelet(Site.Main)
            .AddAuthentication("WebSharper")
            .AddCookie("WebSharper", fun options -> ())
        |> ignore
```

3. back to the *Terminal*, call build and run commands again. The application might work now.

## Useful resources
https://docs.microsoft.com/en-us/aspnet/core/migration/22-to-30?view=aspnetcore-3.0&tabs=visual-studio
https://natemcmaster.com/blog/2019/01/09/netcore-primitives-3/
https://stackoverflow.com/questions/47735133/asp-net-core-synchronous-operations-are-disallowed-call-writeasync-or-set-all
https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-3.0

 
