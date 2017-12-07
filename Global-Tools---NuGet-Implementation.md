Status: **Reviewing**

### Issue
This specification is the NuGet side of a new dotnet cli experience, tentatively called "Global Tools". 
Related specifications are here:
- [.NET Core SDK - Global Tools](https://github.com/dotnet/designs-microsoft/blob/a9ef5b3217e776b4155266826a598db682488613/proposals/global-tools.md) 

- [Global tools implementation design](
https://github.com/dotnet/designs-microsoft/blob/implementation-global-tools/proposals/implementation_global_tools.md)

Each of these design spec are still evolving. 

[NuGet Github Collector issue #6200](https://github.com/NuGet/Home/issues/6200)

### Motivation 
NuGet currently helps provide tools per dotnet cli requirements through the [DotnetCLIToolReference](https://github.com/NuGet/Home/wiki/DotnetCliToolReference-restore) feature. 
The reasoning for adding a new feature is described in the above linked specs. 

### Solution

We will introduce a new "PackageType" and a well-defined format how to create such tools. 

NuGet will provide APIs for the CLI to be able to install mentioned global tools to a pre-determined location. 
To do this, they will create a temporary project that will indicate that it's a global tool restore project, and will contain a global tool reference as a "PackageReference". 
Then the CLI will call NuGet restore with these parameters, which NuGet needs to respect and interpret correctly. 
NuGet will block adding tool packages into standard package reference project. 
Additionally, only 1 global tool package reference per fake project is allowed.

#### Creating a global tool - Pack Experience
Since pack is very extensible, the pack experience, is almost completely in the hands of the CLI team. 
From NuGet side, we want authors to mark their packages with a PackageType metadata, as described in [here](https://docs.microsoft.com/en-us/nuget/schema/msbuild-targets#pack-target). 

**Problem** - What should the package type be?
**Proposal** - The package type should simply be named **Tool**. Because the concepts of global and locals tools are discussed, I think this name minimizes confusion.
In addition, packages with this package type, can **only** have 1 package type!

##### Open Questions
- Double check nuget.org for custom package types named **Tool**
- Do we want to add extra validation on pack side to warn against creating packages with 2 packages if 1 of those package types is **Tool**

#### Installing a global tool
Dotnet CLI will create a **temporary** project and provide all details regarding restore there, including 
- Install directory
- Target Framework
- Runtime Identifier
In addition, this project should contain a restore project style property, named **ToolReference**. 
I am proposing that because we already have a DotnetCLiTool restore style [type](https://github.com/NuGet/NuGet.Client/blob/dev/src/NuGet.Core/NuGet.ProjectModel/ProjectStyle.cs#L26). 
An example temporary project would be:

```
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <RestoreProjectStyle>ToolReference</RestoreProjectStyle>
    <TargetFramework>netcoreapp2.1</TargetFramework>
    <RestorePackagesPath>C:\Users\username\.dotnet\tools\/RestorePackagesPath>
    <RestoreSolutionDirectory>C:\Users\username\code\Library</RestoreSolutionDirectory>
    <DisableImplicitFrameworkReferences>true</DisableImplicitFrameworkReferences>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="my.tool" Version="1.2" />
  </ItemGroup>
</Project>
```

The install directory will be a V3 style directory, where the project.assets.json file will be in the root of the package folder. 

The assets file will need to be extended to include the tools folder when packages are marked with the **Tools** PackageType. CLI will read the assets once restore is done to find the correct tool path. 
NuGet will add the tools assets to the assets file, and then CLI will use the respective APIs to get that asset path. 
If restore fails, the CLI will be able to get the restore status from the assets file/restore exit code. 

Example:
```
C:\Users\username\.dotnet\tools\my.tool\1.2\project.assets.json
```

CLI will control the uninstallation (really deleting the folder), and making sure that there's only a single version of each tool. They can use the [VersionPathResolver](https://github.com/NuGet/NuGet.Client/blob/dev/src/NuGet.Core/NuGet.Packaging/VersionFolderPathResolver.cs) to figure that out. 


##### Error cases
NuGet restore will not succeed in the following scenarios. 

| Scenario | Status | 
| ------------- |:-------------:|
| More than 1 reference in ToolReference RestoreProjectStyle Project     | Restore fails | 
| Tool reference in a non-ToolReference project style     | Restore fails with an incompatibility error   |  
| Non Tool reference in a ToolReference project style | Restore fails with an incompatibility error      |    


##### Visual Studio Experience
Users **must** not author projects like this and load them in VS. 
Above mentioned errors will happen for incorrect hybrid projects. 

##### Open Questions
- NuGet will persist a cache file by default in the same directory as the assets file. Does this cause any problems? Potentially CLI should remove if so. 
- How is the tools restore directory provided, and what is this default directory? Should CLI be the one that provides the directory? If NuGet provides, is it part of a config, and I think this compromises the long-term maintainability of the tools. 
- Discuss the location of the the tools, William is proposing something like ***C:\Users\username\.dotnet\tools\toolName\toolVersion\toolName\toolVersion\***
- Clarify the experience once implemented, if someone tries to load a proper ToolReference project. 

#### Non-Goals
Currently there is no plans to block users from being able to use DotnetCLIToolReference. 

