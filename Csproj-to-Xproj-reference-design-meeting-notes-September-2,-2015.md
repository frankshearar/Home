Please use the design meeting [discussion issue](https://github.com/NuGet/Home/issues/1320) to provide feedback, ask questions, etc.

### Goal and approach
The goal is to reference an xproj from a csproj.

The approach is to treat xproj as a nuget package. Both packages.config and project.json can reference the xproj package.

### Project dependency closure
Scenario, transitive restore for non-project.json projects.
P1 (non-nuget) -> P2 (project.json) -> Package A
P1 needs package A

The entire closure is needed at restore time. This may have a perf impact.

DNX beta8 includes relative paths between projects in the lock file.

The lock file needs to supply the TxM that was used for each project reference. Ex: A->B targets { framework: net45 }

### MSBuild
MSBuild does not support multiple outputs currently which may be an issue for projects where different outputs are needed for different projects referencing the project if the frameworks do not match.

GetTargetPath in MSBuild needs to take a context so that it will return the right outputs for the referencing project.
A new target could be added if modifying the current targets is too risky.
If GetTargetPath is empty another target could be checked.

### Scenarios
Csproj (non-nuget) -> Xproj

Csproj (packages.config) -> Xproj

Csproj (project.json) -> Xproj -> Csproj (packages.config)
Xproj converts the packages.config into a project.json already.

Csproj (project.json) -> Xproj -> Csproj (project.json)
This should work already but there are known bugs.

UWP -> C++ -> Csproj (winmd) -> xproj
There are also scenarios with Managed C++ projects referencing csproj.

Javascript scenarios will need to be supported also.








