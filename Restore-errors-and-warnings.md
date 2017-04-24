This document contains a list of all warnings and errors that may occur during restore.

## Warnings

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
