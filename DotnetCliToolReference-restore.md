### Output paths

Tool restore files will be written to the ``obj`` folder along side the ``project.assets.json`` file.

Each tool is written to a separate file to track success/failure and related restore errors and warnings individually.

Example path ``myProject\obj\myTool.dotnetclitool.json``

### Output format

#### Example
```json
{
  "formatVersion": 1,
  "success": true,
  "toolId": "a",
  "toolVersion": "1.0.0",
  "dependencyRange": "[1.0.0, )",
  "depsFiles": {
    ".NETCoreApp,Version=v1.0": "C:\\Users\\example\\globalPackages\\a\\1.0.0\\lib\\netcoreapp1.0\\a.deps.json",
    ".NETCoreApp,Version=v1.1": "C:\\Users\\example\\globalPackages\\a\\1.0.0\\lib\\netcoreapp1.1\\a.deps.json"
  },
  "packageFolders": {
    "C:\\Users\\example\\.nuget\\packages": {},
    "D:\\shared\\fallback": {}
  },
  "log": [
  ]
}
```

``dependencyRange`` is the original range requested by the project.

``toolVersion`` is the resolved tool package version that NuGet selected as the best match.

``depFiles`` contains the full path to all .deps.json files for a tool package indexed by target framework.

``packageFolders`` uses the same property name and format as project.assets.json. All package folders needed to resolve dependencies are list in order of precedence.

#### Failure example
```json
{
  "formatVersion": 1,
  "success": false,
  "toolId": "a",
  "toolVersion": "1.0.0",
  "dependencyRange": "[1.0.0, )",
  "depsFiles": {
    ".NETCoreApp,Version=v1.0": "C:\\Users\\example\\globalPackages\\a\\1.0.0\\lib\\netcoreapp1.0\\a.deps.json",
    ".NETCoreApp,Version=v1.1": "C:\\Users\\example\\globalPackages\\a\\1.0.0\\lib\\netcoreapp1.1\\a.deps.json"
  },
  "packageFolders": {
    "C:\\Users\\example\\.nuget\\packages": {},
    "D:\\shared\\fallback": {}
  },
  "log": [
    {
      "type": "error",
      "message": "Unable to resolve 'b (= 1.0.0)' for '.NETCoreApp,Version=v1.0'."
    }
  ]
}
```

In the above example restore failed. This is noted by ``success`` being set to ``false``.

Errors and warnings that were displayed during the restore are added to the output file under ``log``. 

### Package format

Tool packages must be marked with the type ``DotnetCliTool``. Restore will validate this type and fail with a related error message.

NuGet will only read deps files under ``lib/netcoreapp*`` that have a name matching the package id. Ex for packageA: ``lib/netcoreapp1.1/packageA.deps.json`` will be used. ``lib/netcoreapp1.1/a.deps.json`` will be ignored.

### API

```csharp
/// <summary>
/// Gives the full path to the tool file.
/// </summary>
DotnetCliToolPathResolver.GetFilePath(string projectOutputDirectory, string packageId)
```

``DotnetCliToolFile`` supports reading/writing tool output files similar to ``LockFile``.

```csharp
public class DotnetCliToolFile
{
    /// <summary>
    /// File json.
    /// </summary>
    public JObject Json { get; }

    /// <summary>
    /// File version.
    /// </summary>
    public int FormatVersion { get; set; } = 1;

    /// <summary>
    /// True if all packages were restored.
    /// </summary>
    public bool Success { get; set; }

    /// <summary>
    /// Tool id.
    /// </summary>
    public string ToolId { get; set; }

    /// <summary>
    /// Resolved tool version.
    /// </summary>
    public NuGetVersion ToolVersion { get; set; }

    /// <summary>
    /// Requested dependency range.
    /// </summary>
    public VersionRange DependencyRange { get; set;}

    /// <summary>
    /// Package folder and package fallback folders.
    /// These are ordered by precedence.
    /// </summary>
    public IList<string> PackageFolders { get; set; } = new List<string>();

    /// <summary>
    /// Framework -> Lib folder path
    /// </summary>
    public IDictionary<NuGetFramework, string> DepsFiles { get; set; } = new Dictionary<NuGetFramework, string>();

    /// <summary>
    /// Restore errors and warnings.
    /// </summary>
    public IList<FileLogEntry> Log { get; set; } = new List<FileLogEntry>();
}
```

### Open questions

1. The deps file stores package hashes, if an author edits a package on nuget.org and the hash no longer matches the deps from stored in a tool package, will the tool runner allow the package to be used?