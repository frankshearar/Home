This document contains a list of all warnings and errors that may occur during restore.

# Errors

## Non-specific errors and warnings

### NU1000

#### Issue
Generic error from NuGet.

### NU1500

#### Issue
Generic warning from NuGet.

## Invalid inputs

### NU1001

#### Issue
The project does not contain one or more frameworks.

#### Common causes
The project does not contain a `TargetFramework` or `TargetFrameworks` property.

#### Example
```
The project projA does not specify any target frameworks in c:\tmp\projA.csproj.
```

### NU1002

#### Issue
Invalid combination of inputs along with a CLEAR keyword.

#### Common causes
CLEAR may not be combined with other inputs.

#### Example
```
'CLEAR' cannot be used in conjunction with other values.
```

### NU1003

#### Issue
`PackageTargetFallback` and `AssetTargetFallback` provide different behavior for selecting assets and can not be used together.

#### Common causes
Both `PackageTargetFallback` and `AssetTargetFallback` exist in the project.

#### Example
```
PackageTargetFallback and AssetTargetFallback cannot be used together. Remove PackageTargetFallback(deprecated) references from the project environment.
```

## Missing packages and projects

### NU1100

#### Issue
A dependency group not be resolved. This is a generic issue for types that are not packages or projects.

#### Common causes
The project contains a dependency on an item that does not exist.

#### Example
```
Unable to resolve System.Missing for net45
```

### NU1101

#### Issue
The package id cannot be found on any sources.

#### Common causes
The correct package source is missing or the package id has a typo.

#### Example
```
Unable to find package System.Missing. No packages exist with this id in source(s): dotnet-core, dotnet-roslyn, NuGet.org
```
### NU1102

#### Issue
The package id is found but a version within the specified dependency range cannot be found on any of the sources.

#### Common causes
The correct package source is missing or the dependency range is incorrect. The range might be specified by a package and not the user.

The user may need to switch to an available version if this package is referenced by the project directly.

#### Example
```
Unable to find package NuGet.Versioning with version (>= 9.0.1)
  - Found 30 version(s) in NuGet.org [ Nearest version: 4.0.0 ]
  - Found 10 version(s) in dotnet-buildtools [ Nearest version: 4.0.0-rc-2129 ]
  - Found 9 version(s) in NuGetVolatile [ Nearest version: 3.0.0-beta-00032 ]
  - Found 0 version(s) in dotnet-core
  - Found 0 version(s) in dotnet-roslyn
```

### NU1103

#### Issue
No stable versions were found in the dependency range. Pre-release versions were found but are not allowed.

#### Common causes
The project specified a stable version for the dependency range. Users need to change this to include pre-release versions.

#### Example
```
Unable to find a stable package NuGet.Versioning with version (>= 3.0.0)
  - Found 10 version(s) in dotnet-buildtools [ Nearest version: 4.0.0-rc-2129 ]
  - Found 9 version(s) in NuGetVolatile [ Nearest version: 3.0.0-beta-00032 ]
  - Found 0 version(s) in dotnet-core
  - Found 0 version(s) in dotnet-roslyn
```

### NU1104

#### Issue
A ProjectReference points to a file that does not exist.

#### Common causes
The project file is missing from disk or the reference is incorrect.

#### Example

```
Project reference does not exist 'c:\a.csproj'. Check that the project reference is valid and that the project file exists.
```

### NU1105

#### Issue
The project file exists but no restore information was provided for it.

#### Common causes
In Visual Studio this could mean that the project is unloaded. From the command line this could mean that the file is corrupt or that it does not contain the custom after imports target needed for restore to read the project.

#### Example

```
Unable to read project information for 'c:\a.csproj'. The project file may be invalid or missing targets required for restore.
```

### NU1106

#### Issue
Dependency constraints cannot be resolved.

#### Common causes
Packages contain dependency on exact versions of a package instead of open ended ranges.

#### Example
```
Unable to satisfy conflicting requests for {id}: {conflict path} Framework: {target graph}
```

### NU1107 (Previously NU1607)

#### Issue
Unable to resolve dependency constraints between packages.

#### Common causes
Packages with dependency constraints on exact versions do not allow other packages to increase the version if needed.

#### Example
```
Version conflict detected for NuGet.Versioning. Reference the package directly from the project to resolve this issue.
  NuGet.Packaging 3.5.0 -> NuGet.Versioning (= 3.5.0)
  NuGet.Configuration 4.0.0 -> NuGet.Versioning (= 4.0.0)
```

### NU1108 (Previously NU1606)

#### Issue
A circular dependency was detected.

#### Common causes
A package is authored incorrectly.

#### Example
```
Cycle detected: A -> B -> A
```

## Compatibility

### NU1201

#### Issue
A dependency project does not contain a framework compatible with the current project.

#### Common causes
The project's target framework is a higher version than the consuming project.

#### Example
```
Project ServerWeb is not compatible with netstandard1.3 (.NETStandard,Version=v1.3). Project ServerWeb supports:
  - netstandard1.6 (.NETStandard,Version=v1.6)
  - netcoreapp1.0 (.NETCoreApp,Version=v1.0)
```

### NU1202

#### Issue
A dependency package does not contain any assets compatible with the project.

#### Common causes
The package does not support the project's target framework.

#### Example
```
Package System.ComponentModel.EventBasedAsync 4.0.11 is not compatible with netstandard1.3 (.NETStandard,Version=v1.3). Package System.ComponentModel.EventBasedAsync 4.0.11 supports:
  - monoandroid10 (MonoAndroid,Version=v1.0)
  - monotouch10 (MonoTouch,Version=v1.0)
  - net45 (.NETFramework,Version=v4.5)
  - netcore50 (.NETCore,Version=v5.0)
  - netstandard1.0 (.NETStandard,Version=v1.0)
  - netstandard1.3 (.NETStandard,Version=v1.3)
  - portable-net45+win8+wp8+wpa81 (.NETPortable,Version=v0.0,Profile=Profile259)
  - win8 (Windows,Version=v8.0)
  - wp8 (WindowsPhone,Version=v8.0)
  - wpa81 (WindowsPhoneApp,Version=v8.1)
  - xamarinios10 (Xamarin.iOS,Version=v1.0)
  - xamarinmac20 (Xamarin.Mac,Version=v2.0)
  - xamarintvos10 (Xamarin.TVOS,Version=v1.0)
  - xamarinwatchos10 (Xamarin.WatchOS,Version=v1.0)
```

### NU1203

#### Issue
The package does not support the project's RuntimeIdentifier.

#### Common causes
The package does not support the current RuntimeIdentifier. Change the RuntimeIdentifiers used in the project if needed.

#### Example
```
System.Example 1.0.0 provides a compile-time reference assembly for a.dll on net461, but there is no compatible run-time assembly.
```

### NU1401

#### Issue
The package requires features or frameworks not currently supported by the installed version of NuGet.

#### Common causes
Upgrade NuGet to fix the issue.

#### Example
```
The 'NuGet.Versioning' package requires NuGet client version '5.0.0' or above, but the current NuGet version is '4.3.0'. To upgrade NuGet, please go to http://docs.nuget.org/consume/installing-nuget.
```

# Warnings

## Invalid inputs

### NU1501

#### Issue
The project restore is attempting to operate on was not found.

#### Common causes
The project is missing.

#### Example
```
The folder 'c:\projects\a' does not contain a project to restore.
```

### NU1502

#### Issue
`RuntimeSupports` contains an invalid profile.

#### Common causes
The supports profile was not found in a runtime.json file from the current dependency packages.

#### Example
```
Unknown Compatibility Profile: aaa
```

### NU1503

#### Issue
A dependency project does not import NuGet's restore targets. This is similar to NU1105 but here the project is skipped and ignored instead of causing all of restore to fail. In complex solutions there are often other types of projects that may not support restore.

#### Common causes
This can happen for projects that do not import common props/targets which automatically import restore. If the project does not need to be restored this can be ignored. 

#### Example
```
Skipping restore for project 'c:\a.csproj'. The project file may be invalid or missing targets required for restore. 
```

## Unexpected package versions

### NU1601

#### Issue
A direct project dependency was bumped to a higher version than the project specified.

#### Common causes
Another dependency package required a higher version and bumped the package up.

#### Example
```
Dependency specified was NuGet.Versioning (>= 3.5.0) but ended up with NuGet.Versioning 4.0.0.
```

### NU1602

#### Issue
A package dependency is missing a lower bound. This does not allow restore to find the *best match*. Each restore will float downwards trying to find a lower version that can be used. This means that restore goes online to check all sources each time instead of using the packages that already exist in the user package folder.

#### Common causes
This is usually a package authoring error.

#### Example
```
NuGet.Packaging 4.0.0 does not provide an inclusive lower bound for dependency NuGet.Versioning (> 3.5.0). An approximate best match of 3.6.0 was resolved.
```

### NU1603

#### Issue
A package dependency specified a version that could not be found. A higher version was used instead, which differs from what the package was authored against.

This means that restore did not find the *best match*. Each restore will float downwards trying to find a lower version that can be used. This means that restore goes online to check all sources each time instead of using the packages that already exist in the user package folder.

#### Common causes
The package sources do not contain the expected lower bound version. If the package expected has not been released then this may be a package authoring error.

#### Example
```
NuGet.Packaging 4.0.0 depends on NuGet.Versioning (>= 4.0.0) but 4.0.0 was not found. An approximate best match of 5.0.0 was resolved.
```

### NU1604

#### Issue
A project dependency does not define a lower bound.

This means that restore did not find the *best match*. Each restore will float downwards trying to find a lower version that can be used. This means that restore goes online to check all sources each time instead of using the packages that already exist in the user package folder.

#### Common causes
The project's *PackageReference* *Version* attribute should be updated to include a lower bound.

#### Example
```
Project dependency NuGet.Versioning (<= 9.0.0) does not contain an inclusive lower bound. Include a lower bound in the dependency version to ensure consistent restore results.
```

### NU1605

#### Issue
A dependency package specified a version constraint on a higher version of a package than restore ultimately resolved.

#### Common causes
Nearest wins when resolving packages. A nearer package in the graph may have overridden a distant package.

#### Example
```
Detected package downgrade: NuGet.Versioning from 4.0.0 to 3.5.0. Reference the package directly from the project to select a different version.
  NuGet.Packaging 3.5.0 -> NuGet.Versioning 3.5.0
  NuGet.Commands 4.0.0 -> NuGet.Configuration 4.0.0 -> NuGet.Versioning 4.0.0
```

## Resolver conflicts

### NU1608

#### Issue
A resolve package is higher than a dependency constraint allows. In some cases this is intentional and the warning can be suppressed.

#### Common causes
A package referenced directly by a project will override dependency constraints from other packages. 

#### Example
```
Detected package version outside of dependency constraint: x 1.0.0 requires y (= 1.0.0) but version y 2.0.0 was resolved.
```

## Package fallback

### NU1701

#### Issue
*PackageTargetFallback* was used to select assets from a package. This is a warning to let the user know that the assets may not be 100% compatible.

#### Common causes
The package does not support the project framework.

#### Example
```
Package 'NuGet.Versioning' was restored using 'portable-net45+win8' instead the project target framework 'netstandard1.5'. This package may not be fully compatible with your project.
```

## Feed warnings

### NU1801

#### Issue
An error occurred when reading the feed. IgnoreFailedSources was set to true, converting it to a non-fatal warning. This could contain any message and is generic.

#### Common causes
The source is invalid.