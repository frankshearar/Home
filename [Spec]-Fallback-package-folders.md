# Fallback package folder

Fallback package folders allow packages to be shared across users and machines to reduce disk space. These folders are treated as fallback folders for the primary global packages folder (``%USERPROFILE%\.nuget\packages``). They differ from package sources in that the package assets will be referenced directly and will not be copied into the user's packages folder.

The concept of a fallback package folder can be thought of as a GAC for nupkgs.

### Goal
The goal of the fallback packages folder is to reduce disk space. Common packages such as *NetStandard.Library* can exist on a network share and multiple users may reference the package assets directly from the share.

### Precedence
Package folders have an order of precedence. During restore and build the user's packages folder should be checked first for a package, then each fallback packages folder in order. Once a package has been found the search stops. Search is based on the id and version of the package only. Where the package was originally downloaded from and other asset information is not considered.

The user's package folder is *always* overrides fallback package folders.

### Folder format
Fallback package folders must use the NuGet v3 folder format of ``{id-lowercase}/{version-lowercase)/{package contents}``. This is the same format as the user package folder format.

The contents of the folder must contain:

1. The extracted package
1. ``{id-lowercase}.nuspec``
1. ``{id-lowercase}.{version-lowercase}.nupkg.sha512``

The folder may optionally contain the nupkg under: ``{id-lowercase}.{version-lowercase}.nupkg``

This format may be created today using ``nuget.exe add`` or ``nuget.exe init``. It may also be created by copying an existing user packages folder that has been created by ``dotnet restore``.

### Immutable
Fallback package folders are never modified by NuGet restore. New packages will be installed only to the user's packages folder.

### Defining fallback folders

Fallback folders may be defined using either an environment variable or *NuGet.Config*. If the environment variable is non-empty *NuGet.Config* will be ignored.

##### Environment variable
Environment variable: ``NUGET_FALLBACK_PACKAGES``

The value must contain the full path or paths to the fallback package folders. Multiple paths may be included by setting the value to a ``;`` delimited list.

##### NuGet.Config

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <fallbackPackageFolders>
    <add key="Shared" value="\\server\sharedPackages" />
    <add key="MachineWide" value="e:\machineWide" />
    <add key="Relative" value="..\..\global" />
  </fallbackPackageFolders>
</configuration>
```

Folders may be defined in NuGet.Config using the ``fallbackPackageFolders`` section. Unlike ``packageSources`` only ``add`` and ``clear`` are supported here, folders may not be individually removed.

Ordering is based on the location of the NuGet.Config file and the order within the file. Entries at the top of the list are used first. Config files nearest to the project are ordered before config files in a higher level parent directory. Machine wide settings are applied last and have the lowest precedence. 

### API

There are two main helpers for working with fallback folders. The first is *INuGetPathContext* which is created with *NuGetPathContext.Create(projectRoot)*, it contains all relevant folders paths. The second is *FallbackPackagePathResolver* which acts on the folder paths to find locate the package.

```cs
public interface INuGetPathContext
{
    /// <summary>
    /// User packages folder directory.
    /// </summary>
    string UserPackagesFolder { get; }

    /// <summary>
    /// Fallback packages folder locations.
    /// </summary>
    IReadOnlyList<string> FallbackPackagesFolders { get; }

    /// <summary>
    /// Http file cache.
    /// </summary>
    string HttpCache { get; }
}
```

```cs
public class FallbackPackagePathResolver
{
    /// <summary>
    /// Creates a package folder path resolver that scans multiple folders to find a package.
    /// </summary>
    /// <param name="pathContext">NuGet paths loaded from NuGet.Config settings.</param>
    public FallbackPackagePathResolver(INuGetPathContext pathContext);

    /// <summary>
    /// Returns the root directory of an installed package.
    /// </summary>
    /// <param name="packageId">Package id.</param>
    /// <param name="version">Package version.</param>
    /// <returns>Returns the path if the package exists in any of the folders. Null if the package does not exist.</returns>
    public string GetPackageDirectory(string packageId, string version);

    /// <summary>
    /// Returns the root directory of an installed package.
    /// </summary>
    /// <param name="packageId">Package id.</param>
    /// <param name="version">Package version.</param>
    /// <returns>Returns the path if the package exists in any of the folders. Null if the package does not exist.</returns>
    public string GetPackageDirectory(string packageId, NuGetVersion version);
}
```

##### Example
The example below loads NuGet.Config settings and creates a helper containing all relevant NuGet Paths using *NuGetPathContext.Create(projectPath)*. Using *FallbackPackagePathResolver* these directories are searched and the first folder found to contain the package id and version is returned.

Required packages: *NuGet.Configuration*, *NuGet.Packaging*.

```cs
using System;
using System.Linq;
using NuGet.Configuration;
using NuGet.Packaging;

namespace Example
{
    class Program
    {
        static void Main(string[] args)
        {
            // Load settings for a project and find the package folder paths.
            var nugetPaths = NuGetPathContext.Create(@"C:\src\json-ld.net");

            // User package folder
            Console.WriteLine($"User folder: {nugetPaths.UserPackagesFolder}");

            // Fallback package folders
            var fallbackFolders = nugetPaths.FallbackPackagesFolders.Any()
                ? string.Join(", ", nugetPaths.FallbackPackagesFolders)
                : "none";
            Console.WriteLine($"Fallback folders: {fallbackFolders}");

            // Create a path resolver to search the package folders.
            var pathResolver = new FallbackPackagePathResolver(nugetPaths);

            // Get the path of the first package folder containing System.Runtime.
            var packagePath = pathResolver.GetPackageDirectory("System.Runtime", "4.1.0-rc2-24027");

            // If the package was not found the path resolver will return null.
            if (packagePath == null)
            {
                Console.WriteLine("Unable to find package!");
                Environment.Exit(1);
            }
            else
            {
                // C:\Users\username\.nuget\packages\System.Runtime\4.1.0-rc2-24027
                Console.WriteLine(packagePath);
            }
        }
    }
}
```

### Error Handling
All fallback package folders specified *must* exist. If the root directory does not exist or any errors are encountered when attempting to access packages in the folder an exception will be thrown. 

This behavior avoids silent failures which could result in unexpected downloads and decreased performance. For example if a fallback package folder is hosted on a network share and the machine is temporarily offline the restore or build should fail.

Note that the user's package folder may not exist, and may never be created if all necessary packages exist in the fallback package folders.

### Tools
The ``.tools`` folder will be ignored for fallback folders. The lock files for tools may only exist at the user level.

Packages for tools may be provided by fallback folders.