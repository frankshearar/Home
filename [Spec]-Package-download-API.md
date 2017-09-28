Status: **Incubation**

### Intro
Msbuild SDK Resolver need an API From NuGet which can download nuget package to .nuget folder by passing package Id and version.

The goal of this API is to provider a high level interface (supports cross-platform) to download nuget package directly and return the package path.

### Public APIs
1. It will be some static utility methods in project NuGet.Commands or new Project

```csharp
    // Download the package 
    public static async Task<string> DownloadPackageAsync(string packageId, 
                                         string versionRange, 
                                         string initialConfigDirectory,
                                         ILogger log);
    
    public static async Task<string> DownloadPackageAsync(string packageId,
                                                   string versionRange,
                                                   string configFile
                                                   ILogger log);

    // Download the package and all dependency packages
    public static async Task<string> DownloadPackageClosureAsync(string packageId, 
                                         string versionRange, 
                                         string initialConfigDirectory,
                                         string targetFramework
                                         ILogger log);
    
    public static async Task<string> DownloadPackageClosureAsync(string packageId,
                                                   string versionRange,
                                                   string configFile
                                                   string targetFramework
                                                   ILogger log);

    public static async Task<string> IsPackageInstalled(string packageId, string version);

```


#### Parameters
1. Package Id Type: String
2. Package Version Range  Type: String
3. Target Framework Type: String 
4. NuGet Setting Root Path Type: String
5. NuGet Config File Path Type: String

#### Option Parameters 
1. NoCache


#### Returns
Package path for the package which is passed as parameter

### Msbuild Task

#### (In) Task Parameters
1. Package Id
2. Package version Range
3. NuGet Setting Root
4. NuGet Config File Path
5. TargetFramework
6. Nocahe

#### (out) Msbuild item
1. Package installed Path for the package which is passed as parameter 

### Design
1. **How to discover Package source, Package Credential?**

   This API will use NuGet.Config to get Package Source information
* User pass NuGet.Config file to the API, then it will use this specified NuGet.Config
* If No NuGet.Config passed and Root path is null or empty, it will load NuGet.Config from Appdata and ProgramData
* If No NuGet.Config passed and Root path is not null, it will load every Nuget.Config from Root Path to Driver root and also load NuGet.Config from Appdata and ProgramData.

2. **How to consume this API?**
* Reference NuGet.Commands or a new package which contains this API in their project.
* NuGet provides a msbuild task to call this API. (need a msbuild task spec for this)

3. **Does this API ignore failed source?**

   Yes, this API ignore failed source by default. we might add parameter to override this behavior in the future.

### Open Questions:

1. Does NuGet need to support downloading multiple packages in one call?
2. Does NuGet need to provide a way to find all dependency packages after the top package downloaded?
3. Do we need a new project for this API?
4. what's the return type (string / Boolean / result type)?
5. Does this API accept VersionRange Type?
6. Should we put this API in a new dll?



