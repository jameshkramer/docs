---
title: Migrating from ASP.NET Web Forms to Blazor
description: How to approach migrating an existing ASP.NET Web Forms app to Blazor
author: twsouthwick
ms.author: tasou
ms.date: 09/11/2019
---

# Migrating from ASP.NET Web Forms to Blazor

Migrating a code base from ASP.NET Web Forms to Blazor follows many of the same patterns that are recommended for .NET Framework applications in general moving to .NET Core. The steps required are:

1. Analyze assemblies with .NET Portability Analyzer and develop a plan to move to .NET Standard
1. Convert project files to new SDK style
1. Update all dependencies to latest versions
1. Update project to run on .NET 4.7.2 or higher
1. Mitigate unsupported APIs
1. Move to .NET Standard

## Analyze assemblies with .NET Portability Analyzer

The first step in making a move to .NET Standard is to understand the holistic view of the application and its dependencies. The [.NET Portability Analyzer](https://github.com/Microsoft/dotnet-apiport) can help in this task. It will take as input all the assemblies that you are interested in porting and give a report as to how many APIs are in used that will not be available in .NET Standard.

When looking at the report and devising a plan for migration, it is important to start at the leaf nodes of the application's dependency graph. The .NET Portability Analyzer has a report it can build that will provide a graph output of the dependencies so you can identify the leaf nodes. A useful way of running the command is to request both the graph output (DGML) and the Excel output:

```
apiport.exe -f [path/to/folder] -r dgml -r excel
```

The `.dgml` output can be opened in Visual Studio to view a graph representation, while the `.xslx` file can be opened in Excel. The output in Excel is useful as the full power of filtering and analysis that Excel provides will be available.

When using this tool, it is also important to only submit user code; third-party dependencies will be addressed at a later stage.

## Convert project files to new SDK style

The next step in converting a code base to support .NET Core is to ready the project files themselves. A new, leaner style of `.csproj` and `.vbproj` is supported for .NET Core that removes much of the extraneous information that was required but proved difficult to maintain. Some of the benefits of this approach are:

1. Source code is implicitly included in the project (with easy opt-out if needed)
1. NuGet packages are referenced in the project file and their dependencies are automatically brought in without needing to explicitly list them
1. Globbing is better supported for including/excluded files
1. AssemblyInfo properties can be controlled by the project file, which is helpful for build-time setting of versions, copyright, and other assembly level attributes.

A basic `csproj` file will now look like the following code:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net46</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="11.0.1" />
  </ItemGroup>
</Project>
```

For more information on the SDK style of project, see the [documentation](https://docs.microsoft.com/en-us/dotnet/core/tools/csproj).

## Update all dependencies to latest versions

One of the most difficult parts of moving to .NET Core is ensuring that third-party dependencies will continue to work once ported. Since often the source code for these libraries is not available, you are not able to rebuild to target .NET Standard. Many dependencies, however, have been ported to support .NET Standard. By updating to the latest version, you are most likely to have a supported package.

In cases where .NET Standard is not supported, there is also the option of being able to run libraries compiled for .NET Framework on .NET Core. Starting with .NET Standard 2.0, a .NET Framework compatibility mode was introduced that allows libraries compiled against the .NET Framework to run on .NET Core. These libraries will not magically find APIs that don't exist, so there is the chance that they will fail at runtime; thus, targeted testing of the scenarios these libraries are used for is recommended. A warning will appear at build time that there were restored for .NET Framework. Although this scenario may cause some errors, it may also help unblock some porting scenarios.

For more information on supporting third-party dependencies, see the [documentation](https://docs.microsoft.com/en-us/dotnet/core/porting/third-party-deps).

## Update project to run on .NET 4.7.2 or higher

.NET Framework 4.x is produced as a machine-wide framework that is updated in-place. This has created the need for a high bar of compatibility for subsequent versions. The main way this has been accomplished is with the introduction of quirks. These quirks are defined by a combination of what framework the application was compiled against as well as the version of the runtime that is being used.

For example, in .NET Framework 4.5, the algorithm used by `List.Sort` was changed from a quick sort to an introspective sort. Both of these sorting algorithms are considered unstable (where items that are equal may not be in the same order each time), but users may have taken a dependency on the ordering of the order of these unstable elements. In order to preserve this old behavior, a quirk was added that would continue to use the quick sort in later versions if the application targeted .NET Framework 4.0 but ran on .NET 4.5 or later. However, if the application were to be retargeted to .NET Framework 4.5 or later, then it would automatically opt into the introspective sort.

The majority of these kinds of breaking changes are manifest when an application takes a dependency on an implementation detail (such as the undefined behavior of an unstable sort). To review what possible changes may impact your application, see the [documentation](https://docs.microsoft.com/en-us/dotnet/framework/migration-guide/application-compatibility) for this topic.

## Mitigate unsupported APIs

At this point, we are ready at a tooling level to make the switch to .NET Standard. However, there may still be APIs in the application that are not supported. Some of these types may be available via the [Windows Compatibility Pack](https://docs.microsoft.com/en-us/dotnet/core/porting/windows-compat-pack). This package adds support for around 20,000 APIs, including registry, WMI, Directory Services, among many more areas.

Some of the high-level areas that are not supported in .NET Core:

- AppDomains
- Remoting
- Code Access Security
- Security Transparency
- System.EnterpriseServices

For more information on areas that are not supported, see the [documentation](https://docs.microsoft.com/en-us/dotnet/core/porting/net-framework-tech-unavailable).

## Move to .NET Standard

Having moved to SDK-style project files, moved to .NET 4.7.2 or later, and mitigated unsupported APIs, the project is now ready to be switched to .NET Standard. This change is as simple as changing:

```
<TargetFramework>net472</TargetFramework>
```

to 

```
<TargetFramework>netstandard2.0</TargetFramework>
```

If a .NET Framework version of the library is still needed for some reason (such as another dependency requiring it), the library can be multi-targeted so that both will be available when needed:

```
<TargetFrameworks>net472;netstandard2.0</TargetFrameworks>
```

Notice that here `TargetFrameworks` is used rather than `TargetFramework`. This is what is meant as multi-targeting and can be helpful while porting a large solution over to .NET Standard when some upstream components still need a .NET Framework build of the library.

Once the target framework has been updated, the project has been updated to .NET Standard. This process will need to be continued for each project up the dependency tree as discovered by the .NET Portability Analyzer. Once all dependencies have been updated, then the application will be ready to be moved to Blazor.
