* Status: Incubation
* Author(s): [Anand Gaurav](https://github.com/anangaur) ([@adgrv](https://twitter.com/adgrv))

## Issue
NuGet outdated, check or equivalent functionality [#5762](https://github.com/nuget/home/issues/5762)

## Problem
* Currently the VS PM UI shows the packages for which updates are available, a similar functionality with the CLI is much needed. This **should work with all package format types** including PackageReference (v4), project.json(v2) and packages.config (v1).
* Unlisted/Vulnerable/Obsolete packages should be flagged both in the PM UI and using CLI commands. See issue: Deprecate obsolete, vulnerable or legacy packages [#2867](https://github.com/NuGet/Home/issues/2867)


## Solution

### CLI

We should have an `outdated` command that can tell users the outdated packages.

_Running in the project folder - Shows only packages that need updates, by default_
```
> dotnet nuget outdated <optional project name or project file>

4 packages need your attention - 3 outdated, 1 deprecated.

Package                Current     Wanted      Latest   
EntityFramework        6.1.2       6.1.2       6.2.0   
NUnit                  2.4.0       2.6.4       3.8.1  
My.Sample.Pkg          2.1.3 (D)   4.1.0       4.1.0

(D): Deprecated package(s). Use '--show-deprecated' option for more info.
```

_Show pre-release package updates_
```
> dotnet nuget outdated --include-prerelease

4 packages need your attention - 3 outdated, 1 deprecated.

Package                Current     Wanted      Latest   
EntityFramework        6.1.2       6.1.2       7.0.0-beta4   
NUnit                  2.4.0       2.6.4       3.8.1  
My.Sample.Pkg          2.1.3 (D)   4.1.0       4.1.0

(D): Deprecated package(s). Use '--show-deprecated' option for more info.
```

_Show all packages in the project_
```
> dotnet nuget outdated --include-prerelease

Total 5 packages. 
4 packages need your attention - 3 outdated, 1 deprecated.

Package                Current     Wanted      Latest   
Newtonsoft.Json        11.0.2      11.0.2      11.0.2
EntityFramework        6.1.2       6.1.2       7.0.0-beta4   
NUnit                  2.4.0       2.6.4       3.8.1  
My.Sample.Pkg          2.1.3 (D)   4.1.0       4.1.0

(D): Deprecated package(s). Use '--show-deprecated' option for more info.
```

_Show [deprecated packages](https://github.com/NuGet/Home/issues/2867) in the project_
```
> dotnet nuget outdated --show-deprecated

Found 3 deprecated packages.

Package                Current       Fixed in   Latest    Deprecation reason   
My.Sample.Pkg          2.1.3         2.2.0      4.1.0     Vulnerable (CVE#<CVE#>): <Author's comment>
Demo.Utilities         1.1.0         -          -         Legacy: Please use My.Demo.Utilities package instead.
Sample.Functions       0.1.0-beta2   1.0.0      2.1.0     Deprecated: Newer stable version available.
```

_Show outdated packages in the solution_
```
> dotnet nuget outdated

<Project name>
<List of outdated packages>

<Project name>
<List of outdated packages>

<Project name>
<List of outdated packages>

(D): Deprecated package(s). Use '--show-deprecated' option for more info.
Total 54 outdated need your attention - 47 outdated, 7 deprecated. 
```

### VS UI
TBD
