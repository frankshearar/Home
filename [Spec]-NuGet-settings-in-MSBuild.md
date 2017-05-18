MSBuild projects should be able to dynamically provide inputs to restore such as sources, fallback folders, and the packages folder path. This would allow the SDK to modify the settings as needed and allow advanced scenarios where these settings are conditional along with PackageReferences.

## Scenarios

### Fallback folders provided by the SDK

CLI 2.x supports fallback folders and users would benefit from using them, however CLI 1.x did not support these. There needs to be a way to define the nuget settings based on the the CLI version.

Using MSBuild properties this could be done by setting the fallback folder path in the SDK for 2.x projects. During restore the new property would allow use of the fallback folder, and 1.x projects would not see the folder or would get the folder as a source.

### Packages folder

Restore uses the user packages folder as a source which can often cause packages from other sources to bleed across the machine and into projects not using those sources.

To fix this a user could dynamically set a different package folder for each set of sources they are restoring against. This would ensure that only packages from the current sources were used.

## Project properties

These project properties would be read by restore from both command line restores and from Visual Studio.

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

If a project provides a value for a property it will be used instead of NuGet.Config settings. If no value is provided then NuGet.Config will be read.

Settings will be selected independently similar to how it is done today. For example if a project sets only fallback folders then sources will still be read from NuGet.Config.

### Clear
``Clear`` is a special property value that will indicate that no value should be used and that NuGet.Config should not be read to find the value either.

Example:
```xml
<PropertyGroup>
  <RestoreFallbackFolders>clear<RestoreFallbackFolders>
</PropertyGroup>
```

Note: If *clear* appears in a set of values Restore will fail with an error message telling the user to fix their property values.

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

## Work needed
1. Project system needs to send these new properties during nomination
1. Add all settings to the dg spec and read it at restore time (mostly complete)

## Additional considerations
* Perf impact of reading three additional properties

