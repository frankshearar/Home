MSBuild projects should be able to dynamically provide inputs to restore such as sources, fallback folders, and the packages folder path. This would allow the SDK to modify the settings as needed and allow advanced scenarios where these settings are conditional along with PackageReferences.

## Fallback folder scenario for the SDK



## Project properties

| Property | Comments |
| -------- | -------- |
| RestoreSources | Feeds used for resolving packages. Equivalent to *--source* in *dotnet restore* |
| RestoreFallbackFolders | Fallback folders, these are used in the same way the user packages folder is used. |
| RestorePackagesPath | Path to the user packages folder. All downloaded packages are extracted here. Equivalent to *--packages* in *dotnet restore* |

## Order of precedence

Settings will be determined using the following order.

1. Properties passed on the command line
1. MSBuild properties
1. NuGet.Config settings

## Supported project types
* NETCore SDK projects with PackageReferences
* Legacy projects with PackageReferences

## Unsupported project types
* Packages.config projects
* project.json projects

## Properties not covered

* HTTP cache folder
* Adding additional sources (NuGet.Config settings + a list of sources)
* Adding additional config files (NuGet.Config settings for the directory + an additional config)

## Additional considerations
* Perf impact of reading three additional properties

