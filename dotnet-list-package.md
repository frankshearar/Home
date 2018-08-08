
## Problem
* Currently the VS PM UI shows a list of the installed packages and the updates available for them, a similar functionality with the CLI is needed.
* Unlisted/Vulnerable/Obsolete packages should be flagged both in the PM UI and using CLI commands. See issue: Deprecate obsolete, vulnerable or legacy packages [#2867](https://github.com/NuGet/Home/issues/2867)


## Solution

### CLI

We should have a `list package` command that can list the installed packages for the developers. For the first version, the `list package` command will work only with the __dotnet command line__ with __package reference projects__. Support for nuget.exe and package config projects will come in later versions.

#### Default command

_Running in the project folder - Shows all packages, by default_
```
> dotnet list package <optional project path or solution path>

Project '.\project.csproj' has the following package references -
    'netstandard1.0'
      Top-level Package           Requested           Resolved
      > NETStandard.Library (A)   2.0.3               2.0.3
      > nuget.versioning          [4.0.0, 4.3.0)      4.0.0
      > SlowCheetah.Xdt           1.0.1-beta          1.0.1-beta

    'net45'
      Transitive Package          Requested           Resolved
      > NETStandard.Library (A)   2.0.3               2.0.3
      > nuget.versioning          [4.0.0, 4.3.0)      4.0.0
      > SlowCheetah.Xdt           1.0.1-beta          1.0.1-beta

A: Auto-referenced package
>
```
__Behavior clarification:__
- __Requested column__ will include the version that the developer has indicated in the package reference tags within the project file, which means it can be a range. __Resolved column__ will include the version that the project is currently using and will always be a single value.
- In the absence of an assets file, the command will show a message saying that an assets file was not found, and will suggest running `dotnet restore` before running list.

The command will have the following options:

| `list package` command option | Behavior |
|:---- |:----|
| `--outdated` | Displays the latest version for the packages that could be updated |
| `--deprecated` | Displays the packages that are marked as deprecated |
| `--include-transitive` | Displays the transitive packages in the result |
| `--framework <FRAMEWORK>` | Displays only the packages applicable for a specific target framework |

___

#### Outdated

Example:

_Running in the project folder with the option `--outdated`_
```
> dotnet list package --outdated

Project '.\project.csproj' has the following package references with updates available -
    'netstandard1.0'
      Top-level Package           Requested           Resolved      Latest
      > nuget.versioning          [4.0.0, 4.3.0)      4.0.0         4.7.0
      > SlowCheetah.Xdt           1.0.1-beta          1.0.1-beta    1.1.7-beta

    'net45'
      Transitive Package          Requested           Resolved      Latest
      > nuget.versioning          [4.0.0, 4.3.0)      4.0.0         4.7.0
      > SlowCheetah.Xdt           1.0.1-beta          1.0.1-beta    1.1.7-beta

>
```

__Behavior Clarification:__
- `--outdated` will consider the prerelease packages when looking for the latest package **only if** the requested version is a prerelease version or the option `--include-prerelease` is used. For example, `nuget.versioning` has a later prerelease package than `4.7.0`, but it is not shown in the result, while `SlowCheetah.Xdt` has the latest prerelease in the result.

___

#### Deprecated

Example:

_Running in the project folder with the option `--deprecated`_
```
> dotnet list package --deprecated

Project '.\project.csproj' has the following deprecated package references -
    'netstandard1.0'
      Top-level Package           Requested           Resolved      Suggested
      > SlowCheetah.Xdt (D)       1.0.1-beta          1.0.1-beta    1.1.7-beta

    'net45'
      Transitive Package          Requested           Resolved      Suggested
      > SlowCheetah.Xdt (D)       1.0.1-beta          1.0.1-beta    1.1.7-beta

D: Deprecated packages
>
```
__Behavior clarification:__
- In the case of `--deprecated` and `--outdated` being used together, the deprecated packages will be shown above the outdated packages, and will have `(D)` to indicate deprecation.

___

#### Include Tansitive

Example:

_Running in the project folder with the option `--include-transitive`_
```
> dotnet list package --include-transitive

Project '.\project.csproj' has the following package references -
    'netstandard1.0'
      Top-level Package           Requested           Resolved
      > NETStandard.Library (A)   2.0.3               2.0.3
      > nuget.versioning          [4.0.0, 4.3.0)      4.0.0
      > SlowCheetah.Xdt           1.0.1-beta          1.0.1-beta

      Transitive Package                             Resolved
      > Microsoft.NETCore.Platforms                  1.1.0
      > Microsoft.CSharp                             4.3.0
      > Microsoft.Web.Xdt                            2.1.2
      > System.Collections                           4.3.0
      > System.ComponentModel.TypeConverter          4.3.0
      > System.Diagnostics.Debug                     4.3.0
      > System.Diagnostics.Tools                     4.3.0
      > System.Globalization                         4.3.0
      > System.IO                                    4.3.0
      > System.Linq                                  4.3.0
      > System.Linq.Expressions                      4.3.0
      > System.Net.Primitives                        4.3.0
      > System.ObjectModel                           4.3.0
      > System.Reflection                            4.3.0
      > System.Reflection.Extensions                 4.3.0
      > System.Reflection.Primitives                 4.3.0
      > System.Resources.ResourceManager             4.3.0
      > System.Runtime                               4.3.0
      > System.Runtime.Extensions                    4.3.0
      > System.Runtime.Serialization.Primitives      4.3.0
      > System.Text.Encoding                         4.3.0
      > System.Text.Encoding.Extensions              4.3.0
      > System.Text.RegularExpressions               4.3.0
      > System.Threading                             4.3.0
      > System.Threading.Tasks                       4.3.0
      > System.Xml.ReaderWriter                      4.3.0
      > System.Xml.XDocument                         4.3.0

    'net45'
      Top-level Package           Requested           Resolved
      > NETStandard.Library (A)   2.0.3               2.0.3
      > nuget.versioning          [4.0.0, 4.3.0)      4.0.0
      > SlowCheetah.Xdt           1.0.1-beta          1.0.1-beta

      Transitive Package                                      Resolved
      > Microsoft.NETCore.Platforms                           1.1.0
      > Microsoft.Web.Xdt                                     2.1.2
      > System.Runtime.InteropServices.RuntimeInformation     4.3.0

A: Auto-referenced package
>
```
__Behavior clarification:__
- Transitive packages are not requested by the developer, as a result, they do not have a requested column.
- `--include-tranistive` could be used with other options like `--outdated` and `--deprecated`.
___

#### Framework

Example:

_Running in the project folder with the option `--framework netstandard1.0`_
```
> dotnet list package --framework netstandard1.0

Project '.\project.csproj' has the following package references -
    'netstandard1.0'
      Top-level Package           Requested           Resolved
      > NETStandard.Library (A)   2.0.3               2.0.3
      > nuget.versioning          [4.0.0, 4.3.0)      4.0.0
      > SlowCheetah.Xdt           1.0.1-beta          1.0.1-beta

A: Auto-referenced package
>
```

__Behavior Clarification:__
- `--framework` will support including multiple frameworks where each framework is separated by a `,`.

___

#### Outdated specific options

The following options might be included in the final version:

| `list package` command option | Behavior |
|:---- |:----|
| `--include-prerelease` | Must be used only with `--outdated` present. Considers the prerelease versions when looking for latest |
| `--highest-patch` | Must be used only with `--outdated` present. Considers only up to the latest patch from the requested version when looking for latest |
| `--highest-minor` | Must be used only with `--outdated` present. Considers only up to the latest minor from the requested version when looking for latest |
| `--config <config file>` | Must be used only with `--outdated` present. Uses only the sources in the config file to check for updates |
| `--source <source name>` | Must be used only with `--outdated` present. Uses only the given source to look for updates |

___

#### Include prerelease

Example:

_Running in the project folder with the option `--outdated --include-prerelease`_
```
> dotnet list package --outdated --include-prerelease

Project '.\project.csproj' has the following package references -
    'netstandard1.0'
      Top-level Package           Requested           Resolved      Latest
      > nuget.versioning          [4.0.0, 4.3.0)      4.0.0         4.8.0-preview1.5156
      > SlowCheetah.Xdt           1.0.1-beta          1.0.1-beta    1.1.7-beta

    'net45'
      Top-level Package           Requested           Resolved      Latest
      > nuget.versioning          [4.0.0, 4.3.0)      4.0.0         4.8.0-preview1.5156
      > SlowCheetah.Xdt           1.0.1-beta          1.0.1-beta    1.1.7-beta

>
```

___

#### Highest patch

Example:

_Running in the project folder with the option `--outdated --highest-patch`_
```
> dotnet list package --outdated --heighest-patch

Project '.\project.csproj' has the following package references -
    'netstandard1.0'
      Top-level Package           Requested           Resolved      Latest patch
      > nuget.versioning          [4.0.0, 4.3.0)      4.0.0         4.3.0
      > SlowCheetah.Xdt           1.0.1-beta          1.0.1-beta    1.0.9-beta

    'net45'
      Top-level Package           Requested           Resolved      Latest patch
      > nuget.versioning          [4.0.0, 4.3.0)      4.0.0         4.3.0
      > SlowCheetah.Xdt           1.0.1-beta          1.0.1-beta    1.0.9-beta

>
```

___

#### Highest minor

Example:

_Running in the project folder with the option `--outdated --highest-minor`_
```
> dotnet list package --outdated --highest-minor

Project '.\project.csproj' has the following package references -
    'netstandard1.0'
      Top-level Package           Requested           Resolved      Latest minor
      > nuget.versioning          [4.0.0, 4.3.0)      4.0.0         4.7.0
      > SlowCheetah.Xdt           1.0.1-beta          1.0.1-beta    1.1.7-beta

    'net45'
      Top-level Package           Requested           Resolved      Latest minor
      > nuget.versioning          [4.0.0, 4.3.0)      4.0.0         4.7.0
      > SlowCheetah.Xdt           1.0.1-beta          1.0.1-beta    1.1.7-beta

>
```
__Behavior clarification:__
- `--highest-patch` and `--highest-minor` will be constrained by the patch and minor, repectively, of the **requested** version.
___


_Show list packages in the solution_
```
> dotnet list package

<Project name>
<List of packages>

--------------------------------------------------------

<Project name>
<List of packages>

--------------------------------------------------------

<Project name>
<List of packages>

```

