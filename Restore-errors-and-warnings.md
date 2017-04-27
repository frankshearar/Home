This document contains a list of all warnings and errors that may occur during restore.

# Updated Errors in 4.3.0

## Missing packages and projects

### NU2001

#### Issue
The package id cannot be found on any sources.

#### Common causes
The correct package source is missing or the package id has a typo.

#### Example
```
Unable to find package 'a'. No packages exist with this id in source(s): http://dotnet.myget.org/F/nuget-build/, https://api.nuget.org/v3/index.json
```
#### NU2002

#### Issue
The package id is found but a version within the specified dependency range cannot be found on any of the sources.

#### Common causes
The correct package source is missing or the dependency range is incorrect. The range might be specified by a package and not the user.

The user may need to switch to an available version if this package is referenced by the project directly.

#### Example
```
Unable to find package 'a' with version >= 1.0.0
  - Found 2839 version(s) in http://dotnet.myget.org/F/nuget-build/  [ Nearest version: 1.0.0-beta ]
  - Found 0 version(s) in https://api.nuget.org/v3/index.json
```

#### NU2003

#### Issue
No stable versions were found in the dependency range. Pre-release versions were found but are not allowed.

#### Common causes
The project specified a stable version for the dependency range. Users need to change this to include pre-release versions.

#### Example
```
Unable to find a stable package 'a' with version >= 1.0.0
  - Found 2839 version(s) in http://dotnet.myget.org/F/nuget-build/  [ Nearest version: 1.0.0-beta ]
  - Found 0 version(s) in https://api.nuget.org/v3/index.json
```

## Errors in 4.2.0

| Code | Project | Group | Message | Fields | Comments |
|:----:| ------- | ----- | ------- | ------ | -------- |
| | DependencyResolver | Http | The feed {0} lists package {1} but multiple attempts to download the nupkg have failed. The feed is either invalid or required packages were removed while the current operation was in progress. Verify the package exists on the feed and try again | uri, package | Feed is likely corrupt, recently added this message |
| | NuGet.Commands | resolver | Failed to resolve conflicts for {0} | graph | |
| | NuGet.Commands | resolver | Unable to satisfy conflicting requests for {0}: {1} | packages | combines all conflicts into a single message |
| | NuGet.Commands | compat | {0} {1} provides a compile-time reference assembly for {2} on {3}, but there is no compatible run-time assembly | package, framework | Log_MissingImplementationFx |
| | NuGet.Commands | compat | {0} {1} provides a compile-time reference assembly for {2} on {3}, but there is no run-time assembly compatible with {4} | package, framework | Log_MissingImplementationFxRuntime |
| | NuGet.Commands | compat | Package {0} {1} is not compatible with {2} | package, framework | Log_PackageNotCompatibleWithFx (complete message) |
| | NuGet.Commands | compat | Project {0} is not compatible with {1} | project, framework | Log_ProjectNotCompatibleWithFx |
| | NuGet.Commands | notfound | Unable to resolve {0} for {1} | project, graph | A package cannot be found on the feeds. (Highest priority to improve ) |
| | NuGet.Commands | resolver | Cycle detected: a -> b -> a | packages | circular dependency |
| | NuGet.Commands | resolver | Version conflict detected for {0} | |
| | NuGet.Commands | summary | One or more projects are incompatible with {0} | project | remove this? |
| | NuGet.Commands | summary | One or more packages are incompatible with {0} | project | remove this? |
| | NuGet.Commands | input | The project {0} does not specify any target frameworks in {1} | project, path | |
| | NuGet.Protocol | http | ``<http exception>`` | | PromptForProxyCredentialsAsync |
| | NuGet.Procotol | http | Failed to download package {0} from {1} ``<exception>`` | package, source | download error |
| | NuGet.Commands | package | Unknown build action | | nuspec contains invalid contentFiles data |
| | NuGet.Commands | disk | Failed to find a project to restore in the folder {0} | UnauthorizedAccessException |
| | NuGet.Commands | input | Ambiguous project name {0} | project | root project has a duplicate |
| | NuGet.Commands | internal | Missing external reference metadata for {0} | project | project data was not provided, internal issue |
| | NuGet.Commands | input | Invalid input {0}. Valid file names are project.json or *.project.json | | no longer needed? |
| | NuGet.Commands | input | File not found {0} | path | input file was not found |
| | NuGet.Protocol | feed | InvalidDataException {0} | Url | invalid registration file InvalidDataException(registrationUri.AbsoluteUri) | 
| | NuGet.Protocol | feed | The JSON document is not an object. | | |
| | NuGet.Protocol | feed | The JSON document is not complete. | | |
| | NuGet.Protocol | feed | An error occurred while retrieving package metadata for {0} from source {1} | package, source | exception from source |
| | NuGet.Protocol | feed | Error downloading {0} from {1} | | |
| | NuGet.Protocol | feed | The V2 feed at {0} returned an unexpected status code {1} {2} | | |
| | NuGet.Protocol | feed | The path {0} for the selected source could not be resolved. | | |

## Warnings in 4.2.0

| Code | Project | Group | Message | Fields | Comments |
|:----:| ------- | ----- | ------- | ------ | -------- |
| | NuGet.Build.Tasks | Input | Unable to find a project to restore! | | |
| | NuGet.Protocol | Http | An invalid cache entry was found for URL {0} and will be replaced | Uri | thrown for any invalid data |
| | NuGet.Protocol | Http | ``<Message from server>`` | | This contains whatever the server wants to display to the user from X-NuGet-Warning |
| | NuGet.Protocol | Http | Unable to find package | Package Id | |
| | NuGet.Protocol | Nupkg | The file {0} is corrupt | path | Nupkg is corrupt or unreadable |
| | NuGet.Protocol | Local | The folder {0} contains an invalid version | path | v3 folder with invalid sub folder |
| | NuGet.Protocol | Local | ``<File system exception message>`` | | |
| | NuGet.Commands | Input | The folder {0} does not contain a project to restore | path | |
| | NuGet.Commands | Package | Dependency specified was {0} {1} but ended up with {2} {3} | packages | Version bumped |
| | NuGet.Commands | Project | Package {0} was restored using {1} instead the project target framework {2}. This may cause compatibility problems | package, framework | imports |
| | NuGet.Commands | Package | Detected package downgrade: {0} from {1} to {2} | package, versions | Downgrade warning |
| | NuGet.Commands | Input | Unknown Compatibility Profile: {0} | compat profile | bad input from project |
