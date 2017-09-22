Status: **Incubation**

### Intro
Msbuild SDK Resolver need an API From NuGet which can download nuget package to specified path by passing package Id and version.

The goal of this API is to provider a high level interface (supports cross-platform) to download nuget package directly and return the package path.

### Public APIs
1. It will be a static utility method in project NuGet.Commands

```csharp
    public static string DownloadPackage(string packageId, string version, string targetFramework, string root, string configFile)
```


#### Parameters
1. Package Id Type: String
2. Package version (can be floating version) Type: String
3. Target Framework Type: String or NuGetFramework
4. NuGet Setting Root Path Type: String
5. NuGet Config File Path Type: String

#### Option Parameters (TBD)
1. NoCache
2. Prerelease
3. DependencyBehavior


#### Returns
Package Path for the package which is passed as parameter

### Design
1. **How to discover Package source, Package Credential?**

   This API will use NuGet.Config to get Package Source information
* User pass NuGet.Config file to the API, then it will use this specified NuGet.Config
* If No NuGet.Config passed and Root path is null or empty, it will load NuGet.Config from Appdata and ProgramData
* If No NuGet.Config passed and Root path is not null, it will load every Nuget.Config from Root Path to Driver root and also load NuGet.Config from Appdata and ProgramData.

2. **How to specify where the package is downloaded to?** (**TBD**)

* Choice 1: user can specify the globalPackages Folder in NuGet.Config, it user this path as installed Path.
* Choice 2: user can pass the path to this API.

3. **How to consume this API?**
* Reference NuGet.Commands or NuGet.Commands.Xplat package in their project.
* NuGet provides a msbuild task to call this API. (need a msbuild task spec for this)

### Open Questions:

1. Does NuGet need to support downloading multiple packages in one call?
2. Does NuGet need to provide a way to find all dependency packages after the top package downloaded?




