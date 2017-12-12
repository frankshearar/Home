Status: **Reviewing**

### Issue
This specification is the NuGet side of a new dotnet cli experience, tentatively called "Global Tools". 
Related specifications are here:
- [.NET Core SDK - Global Tools](https://github.com/dotnet/designs-microsoft/blob/a9ef5b3217e776b4155266826a598db682488613/proposals/global-tools.md) 

- [Global tools implementation design](
https://github.com/dotnet/designs-microsoft/blob/implementation-global-tools/proposals/implementation_global_tools.md)

Each of these design specs are still evolving. 

[NuGet Github Collector issue #6200](https://github.com/NuGet/Home/issues/6200)

### Motivation 
NuGet currently helps provide tools per dotnet cli 1.x requirements through the [DotnetCLIToolReference](https://github.com/NuGet/Home/wiki/DotnetCliToolReference-restore) feature. 
The reasoning for adding a new feature is described in the above linked specs. 
We believe most tool publishers will move to global tools.

### Solution

We will introduce a new "PackageType" and a well-defined format how to create such tools. 

NuGet will provide APIs for the CLI to be able to install mentioned global tools to a pre-determined location. 
To do this, they will create a temporary project that will indicate that it's a global tool restore project, and will contain a global tool reference as a "PackageReference". 
Then the CLI will call NuGet restore with these parameters, which NuGet needs to respect and interpret correctly. 
NuGet will block adding tool packages into standard package reference project. 
Additionally, only 1 global tool package reference per fake project is allowed.

#### Creating a global tool - Pack Experience
Since pack is very extensible, the pack experience doesn't need additional work from the NuGet side. 
From NuGet side, we will require authors to mark their packages with a PackageType metadata, as described in [here](https://docs.microsoft.com/en-us/nuget/schema/msbuild-targets#pack-target). 

**Problem** - What should the package type be?

**Proposal** - The package type should simply be named **CommandLineTool**. Because the concepts of global and locals tools are discussed, I think this name minimizes confusion.

In addition, packages with this package type, can **only** have 1 package type!
We need to add validation on pack side that there can be no packages with more than 1 package type that contains the package type **CommandLineTool**.

##### Open Questions
- Rob - we should be enforcing this 1 package type today. Update: We are not enforcing it on pack. We only do when we explicitly install a package. Created [Task 6298](https://github.com/NuGet/Home/issues/6298) as a result. 
- Double check nuget.org for custom package types named **CommandLineTool** (Rob - however, we invent them...not third parties...need to discuss policy) Above issue discussion applies to this as well. 

#### Installing a global tool
Dotnet CLI will create a **temporary** project and provide all details regarding restore there, including 
- Install directory (where to install and extract nupkg and dependencies)
- Target Framework (used by restore to select assets)
- Runtime Identifier (used by restore to select assets)
- BaseIntermediateOutputPath (where assets file goes)
In addition, this project should contain a restore project style property, named **CommandLineToolReference**. (another option would be to do PR, but then have another property). 
I am proposing that because we already have a DotnetCLiTool restore style [type](https://github.com/NuGet/NuGet.Client/blob/dev/src/NuGet.Core/NuGet.ProjectModel/ProjectStyle.cs#L26). 
An example temporary project would be:

```
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <RestoreProjectStyle>CommandLineToolReference</RestoreProjectStyle>
    <TargetFramework>netcoreapp2.1</TargetFramework>
     <!-- c:\users\username\.nuget was returned by an API that they called on us to find machinewide place for tools
     The packageId is provided by the CLI team, and the package version if passed. 
The CLI will handle cases where the version is not passed by creating a dummy folder name and then renaming the folder when they get the package version from NuGet.
     -->
    <RestorePackagesPath>C:\Users\username\.nuget\tools\packageId\packageVersion/RestorePackagesPath>
    <RestoreSolutionDirectory>C:\Users\username\code\Library</RestoreSolutionDirectory>
    <DisableImplicitFrameworkReferences>true</DisableImplicitFrameworkReferences>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="my.tool" Version="1.2" />
  </ItemGroup>
</Project>
```

[Task 6260](https://github.com/NuGet/Home/issues/6260) to return the machine wide tools folder.

NuGet will also provide a way to walk the settings based on the CLI working directory. 
Current workaround is RestoreSolutionDirectory. [Task 6199](https://github.com/NuGet/Home/issues/6199)

The install directory will be a V3 style directory.

The assets file will need to be extended to include the tools folder when packages are marked with the **CommandLineTool** PackageType. [Task 6197](https://github.com/NuGet/Home/issues/6197)

CLI will read the assets once restore is done to find the correct tool path. 
NuGet will add the tools assets to the assets file, and then CLI will use the respective APIs to get that asset path. 

If restore fails, the CLI will be able to get the restore status from the assets file/restore exit code. 

Example:
```
C:\Users\username\.dotnet\tools\my.tool\1.2\project.assets.json
```

CLI will handle the restore failure cases, and partial file left on disk. They can use the [VersionPathResolver](https://github.com/NuGet/NuGet.Client/blob/dev/src/NuGet.Core/NuGet.Packaging/VersionFolderPathResolver.cs) to figure that out. 


##### Error cases
NuGet restore will not succeed in the following scenarios. [Task 6198](https://github.com/NuGet/Home/issues/6198)

| Scenario | Status | 
| ------------- |:-------------:|
| More than 1 reference in ToolReference RestoreProjectStyle Project     | Restore fails | 
| Tool reference in a non-ToolReference project style     | Restore fails with an incompatibility error   |  
| Non Tool reference in a ToolReference project style | Restore fails with an incompatibility error      |    


##### Visual Studio Experience
Users **must** not author projects like this and load them in VS. 
Above mentioned errors will happen for incorrect hybrid projects. 

##### Open Questions
- NuGet will persist a cache file by default in the same directory as the assets file. CLI can consider cleaning this up. 
- Clarify the experience once implemented, if someone tries to load a proper ToolReference project. Visual Studio will currently treat it as a package reference project since the restore project style property is not nominated. 
- My initial idea is for nuget.exe to not "treat" projects like this as restorable. So you will only be able to restore this type of projects through the restore command. 


#### Non-Goals
Currently there is no plans to block users from being able to use DotnetCLIToolReference. 

