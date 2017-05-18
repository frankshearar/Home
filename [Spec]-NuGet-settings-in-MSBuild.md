MSBuild projects should be able to dynamically provide inputs to restore such as sources, fallback folders, and the packages folder path. This would allow the SDK to modify the settings as needed and allow advanced scenarios where these settings are conditional along with PackageReferences.

## Scenarios

### Fallback folders provided by the SDK

CLI 2.x supports fallback folders and users would benefit from using them, however CLI 1.x did not support these. There needs to be a way to define the nuget settings based on the the CLI version.

Using MSBuild properties this could be done by setting the fallback folder path in the SDK for 2.x projects. During restore the new property would allow use of the fallback folder, and 1.x projects would not see the folder or would get the folder as a source.

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

