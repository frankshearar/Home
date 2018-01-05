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
The will be included in the following path in the nupkg:
```
tools\{tfm}\{rid}\*.dll
```
* tfm is all accepted frameworks
* rid is all known rids + "Any"

##### Open Questions
###### Enforce package types
- Rob - we should be enforcing this 1 package type today. Update: We are not enforcing it on pack. We only do when we explicitly install a package. Created [Task 6298](https://github.com/NuGet/Home/issues/6298) as a result. 
###### Package Type Name
- What should the package type be?
Options: **CommandLineTool**, **GlobalTool**, **Tool**, **DotnetTool**
Are these packages command line only? They can pull up a UI etc. 
Is global tools the correct branding considering there are local tools as well.
Tool is too generic, and per .NET principle, don’t waste a great name. 
DotnetTool is a misnomer, because it implies dotnet(portable), but eventually the dotnet CLI will/might support .NET full framework. 
Need to work with the CLI team to understand the branding. 
###### NuGet.org handling of tool packages
NuGet.org will need to make a few changes based on the new package type.
- NuGet.org should reject packages that include the global package tool and any other package type [Gallery Task 5182](https://github.com/NuGet/NuGetGallery/issues/5182) 
- Packages with this package type should display a “dotnet install packageId packageVersion” tab only, rather than the standard tabs. [Gallery Task 5182](https://github.com/NuGet/NuGetGallery/issues/5182) 
- Search by packageType should be consider in the package applicability search work. [Task 5725](https://github.com/NuGet/Home/issues/5725)


#### Installing a global tool
Dotnet CLI will create a **temporary** project and provide all details regarding restore there, including 
- Install directory (where to install and extract nupkg and dependencies)
c:\users\username\.nuget was returned by an API that they called on us to find machinewide place for tools
     The packageId is provided by the CLI team, and the package version if passed. 
The CLI will handle cases where the version is not passed by creating a dummy folder name and then renaming the folder when they get the package version from NuGet.
- BaseIntermediateOutputPath (where assets file goes)
We need to qualify the [scenario](#local-tool-and-install-location) for local tools
- Target Framework (used by restore to select assets)
- Runtime Identifier (used by restore to select assets)
- RestoreRootConfigDirectory(the CLI working directory, from where the nuget.config graph will be walked)
- RestoreFallbackFolders (for global tools, this needs to be "clear" in order to make the tools self-containing)
- RestoreAdditionalProjectSources, RestoreAdditionalProjectFallbackFolders, RestoreAdditionalProjectFallbackFoldersExcludes are values brought in through the SDK. Same as above, for global tools this compromises the principle of the tools needing to be self-containing. 

In addition, this project should contain a restore project style property, named **DotnetToolReference**. (another option would be to do PR, but then have another property). 
I am proposing that because we already have a DotnetCLiTool restore style [type](https://github.com/NuGet/NuGet.Client/blob/dev/src/NuGet.Core/NuGet.ProjectModel/ProjectStyle.cs#L26). 
An example temporary project would be:

```
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <RestoreProjectStyle>DotnetToolReference</RestoreProjectStyle>
    <TargetFramework>netcoreapp2.1</TargetFramework>
    <RestorePackagesPath>C:\Users\username\.nuget\tools\packageId\packageVersion</RestorePackagesPath>
    <BaseIntermediateOutputPath>C:\Users\username\.nuget\toools\packageId\versionVersion</BaseIntermediateOutputPath>
    <RestoreRootConfigDirectory>C:\Users\username\code\Library</RestoreRootConfigDirectory>
    <DisableImplicitFrameworkReferences>true</DisableImplicitFrameworkReferences>
    <RestoreFallbackFolders>clear</RestoreFallbackFolders>
    <RestoreAdditionalProjectSources></RestoreAdditionalProjectSources>
    <RestoreAdditionalProjectFallbackFolders></RestoreAdditionalProjectFallbackFolders>
    <RestoreAdditionalProjectFallbackFoldersExcludes></RestoreAdditionalProjectFallbackFoldersExcludes>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="my.tool" Version="1.2" />
  </ItemGroup>
</Project>
```

[Task 6260](https://github.com/NuGet/Home/issues/6260) to return the machine wide tools folder.

NuGet will also provide a way to walk the settings based on the CLI working directory. 
Update: Added RestoreRootConfigDirectory [Task 6199](https://github.com/NuGet/Home/issues/6199)

The install directory will be a V3 style directory.

The assets file will need to be extended to include the tools folder when packages are marked with the **DotnetTool** PackageType. [Task 6197](https://github.com/NuGet/Home/issues/6197)

CLI will read the assets once restore is done to find the correct tool path. 
NuGet will add the tools assets to the assets file, and then CLI will use the respective APIs to get that asset path. 

If restore fails, the CLI will be able to get the restore status from the assets file/restore exit code. 

Example:
```
C:\Users\username\.nuget\tools\my.tool\1.2\project.assets.json
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
- How do we handle the scenario where the global packages folder is at the root of a drive? Example F:\
The tools folder cannot be at the same level as this. 
Related task: [Task 6260](https://github.com/NuGet/Home/issues/6260)
- NuGet will persist a cache file by default in the same directory as the assets file. CLI can consider cleaning this up.
- NuGet version currently does not support a way to "request" the latest prerelease version.
Tracking issues, [Task 4699](https://github.com/nuget/home/issues/4699)  [Task 912](https://github.com/nuget/home/issues/912)
- Clarify the experience once implemented, if someone tries to load a proper ToolReference project. Visual Studio will currently treat it as a package reference project since the restore project style property is not nominated. 
- My initial idea is for nuget.exe to not "treat" projects like this as restorable. So you will only be able to restore this type of projects through the restore task. 

##### CLI actionables. 
###### Conform the above project outline
###### Help settle on the [package type name](#package-type-name), by considering their branding. 
###### Local Tool and install location
- How does the CLI make a decision about the RestorePackagesPath/baseintermediateOutputPath for local tools?
###### In the current implementation -g installs a tool user-wide, in npm it's user wide
- Would this cause a confusion? Is the branding correct? 
###### How will global tools be cleaned up
Does the CLI provide a remove command? 
NuGet does not need to add tools to clean this global tools folder. However, we have had occasional complaints about the size of the .nuget folder. This increases it drastically
###### Handle partial installs
CLI needs to make sure they clean when the restore fails for whatever reason. Be it an incompatibility error, or where only 1/2 packages gets installed etc. 


#### Non-Goals
Currently there is no plans to block users from being able to use DotnetCLIToolReference. 

