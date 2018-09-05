
## Problem
* Currently the VS PM UI shows a list of the installed packages and the updates available for them, a similar functionality with the CLI is needed.
* Unlisted/Vulnerable/Obsolete packages should be flagged both in the PM UI and using CLI commands. See issue: Deprecate obsolete, vulnerable or legacy packages [#2867](https://github.com/NuGet/Home/issues/2867).


## Solution

### CLI

We should have a `list package` command that can list the installed packages for the developers. For the first version, the `list package` command will work only with the __dotnet command line__ with __package reference projects__. Support for nuget.exe and package config projects will come in later versions.

#### Default command

_Running in the project folder - Shows all packages, by default_
```
> dotnet list package <optional project path or solution path>

Project '.\project.csproj' has the following package references
   [netstandard1.0]:
   Top-level Package             Requested        Resolved
   > NETStandard.Library   (A)   2.0.3            2.0.3
   > nuget.versioning            [4.0.0, 4.3.0)   4.0.0
   > SlowCheetah.Xdt             1.0.1-beta       1.0.1-beta

   [net45]:
   Transitive Package            Requested        Resolved
   > NETStandard.Library   (A)   2.0.3            2.0.3
   > nuget.versioning            [4.0.0, 4.3.0)   4.0.0
   > SlowCheetah.Xdt             1.0.1-beta       1.0.1-beta

(A) : Auto-referenced package
>
```
__Behavior clarification:__
- __Requested column__ will include the version that the developer has indicated in the package reference tags within the project file, which means it can be a range. __Resolved column__ will include the version that the project is currently using and will always be a single value.
- In the absence of an assets file, the command will show a message saying that an assets file was not found, and will suggest running `dotnet restore` before running list.

The command will have the following options:

| `list package` command option | Behavior |
|:---- |:----|
| `--outdated` | Displays the latest version for the packages that could be updated. |
| `--include-transitive` | Displays the transitive packages in the result. |
| `--framework <FRAMEWORK>` | Displays only the packages applicable for a specific target framework. Supports multiple frameworks by using `--framework` multiple times. |

___

#### Outdated

Example:

_Running in the project folder with the option `--outdated`_
```
> dotnet list package --outdated

Project 'project' has the following updates to its packages
   [.netstandard1.0]:
   Top-level Package       Requested        Resolved     Latest
   > nuget.versioning      [4.0.0, 4.3.0)   4.0.0        4.7.0
   > SlowCheetah.Xdt       1.0.1-beta       1.0.1-beta   1.1.7-beta

   [net45]:
   Transitive Package      Requested        Resolved     Latest
   > nuget.versioning      [4.0.0, 4.3.0)   4.0.0        4.7.0
   > SlowCheetah.Xdt       1.0.1-beta       1.0.1-beta   1.1.7-beta

>
```

__Behavior Clarification:__
- `--outdated` will consider the prerelease packages when looking for the latest package **only if** the resolved version is a prerelease version or the option `--include-prerelease` is used. For example, `nuget.versioning` has a later prerelease package than `4.7.0`, but it is not shown in the result, while `SlowCheetah.Xdt` has the latest prerelease in the result.

___

#### Include Tansitive

Example:

_Running in the project folder with the option `--include-transitive`_
```
> dotnet list package --include-transitive

Project 'project' has the following package references
   [netstandard1.0]:
   Top-level Package             Requested        Resolved
   > NETStandard.Library   (A)   2.0.3            2.0.3
   > nuget.versioning            [4.0.0, 4.3.0)   4.0.0
   > SlowCheetah.Xdt             1.0.1-beta       1.0.1-beta

   Transitive Package                          Resolved
   > Microsoft.NETCore.Platforms               1.1.0
   > Microsoft.CSharp                          4.3.0
   > Microsoft.Web.Xdt                         2.1.2
   > System.Collections                        4.3.0
   > System.ComponentModel.TypeConverter       4.3.0
   > System.Diagnostics.Debug                  4.3.0
   > System.Diagnostics.Tools                  4.3.0
   > System.Globalization                      4.3.0
   > System.IO                                 4.3.0
   > System.Linq                               4.3.0
   > System.Linq.Expressions                   4.3.0
   > System.Net.Primitives                     4.3.0
   > System.ObjectModel                        4.3.0
   > System.Reflection                         4.3.0
   > System.Reflection.Extensions              4.3.0
   > System.Reflection.Primitives              4.3.0
   > System.Resources.ResourceManager          4.3.0
   > System.Runtime                            4.3.0
   > System.Runtime.Extensions                 4.3.0
   > System.Runtime.Serialization.Primitives   4.3.0
   > System.Text.Encoding                      4.3.0
   > System.Text.Encoding.Extensions           4.3.0
   > System.Text.RegularExpressions            4.3.0
   > System.Threading                          4.3.0
   > System.Threading.Tasks                    4.3.0
   > System.Xml.ReaderWriter                   4.3.0
   > System.Xml.XDocument                      4.3.0

   [net45]:
   Top-level Package             Requested        Resolved
   > NETStandard.Library   (A)   2.0.3            2.0.3
   > nuget.versioning            [4.0.0, 4.3.0)   4.0.0
   > SlowCheetah.Xdt             1.0.1-beta       1.0.1-beta

   Transitive Package                                    Resolved
   > Microsoft.NETCore.Platforms                         1.1.0
   > Microsoft.Web.Xdt                                   2.1.2
   > System.Runtime.InteropServices.RuntimeInformation   4.3.0

(A) : Auto-referenced package
>
```
__Behavior clarification:__
- Transitive packages are not requested by the developer, as a result, they do not have a requested column.
- `--include-tranistive` could be used with other options like `--outdated`.
___

#### Framework

Example:

_Running in the project folder with the option `--framework netstandard1.0`_
```
> dotnet list package --framework netstandard1.0

Project 'project' has the following package references
   [netstandard1.0]:
   Top-level Package             Requested        Resolved
   > NETStandard.Library   (A)   2.0.3            2.0.3
   > nuget.versioning            [4.0.0, 4.3.0)   4.0.0
   > SlowCheetah.Xdt             1.0.1-beta       1.0.1-beta

(A) : Auto-referenced package
>
```

__Behavior Clarification:__
- `--framework` supports including multiple frameworks by using `--framework` multiple times.
- `--framework` supports specific runtime identifier by using `/`. For example, `dotnet list package --framework netstandard1.0/win-x64`.

___

#### Outdated specific options

The following options work only with `--outdated`:

| `list package --outdated` command option | Behavior |
|:---- |:----|
| `--include-prerelease` | Considers the prerelease versions when looking for the latest version. Can only be used with `--outdated` present. |
| `--highest-patch` | Considers only up to the latest patch from the requested version when looking for the latest version. Can only be used with `--outdated` present. |
| `--highest-minor` | Considers only up to the latest minor from the requested version when looking for the latest version. Can only be used with `--outdated` present. |
| `--config <config file>` | Uses only the sources in the config file to look for the latest version. Can only be used with `--outdated` present. |
| `--source <source name>` | Uses only the given source to look for the latest version. Can only be used with `--outdated` present. Supports multiple sources by using `--source` multiple times. |

___

#### Include prerelease

Example:

_Running in the project folder with the option `--outdated --include-prerelease`_
```
> dotnet list package --outdated --include-prerelease

Project 'project' has the following updates to its packages
   [netstandard1.0]:
   Top-level Package       Requested        Resolved     Latest
   > nuget.versioning      [4.0.0, 4.3.0)   4.0.0        4.8.0-preview1.5156
   > SlowCheetah.Xdt       1.0.1-beta       1.0.1-beta   1.1.7-beta

   [net45]:
   Top-level Package       Requested        Resolved     Latest
   > nuget.versioning      [4.0.0, 4.3.0)   4.0.0        4.8.0-preview1.5156
   > SlowCheetah.Xdt       1.0.1-beta       1.0.1-beta   1.1.7-beta

>
```

___

#### Highest patch

Example:

_Running in the project folder with the option `--outdated --highest-patch`_
```
> dotnet list package --outdated --heighest-patch

Project 'project' has the following updates to its packages
   [netstandard1.0]:
   Top-level Package       Requested        Resolved     Latest patch
   > nuget.versioning      [4.0.0, 4.3.0)   4.0.0        4.3.0
   > SlowCheetah.Xdt       1.0.1-beta       1.0.1-beta   1.0.9-beta

   [net45]:
   Top-level Package       Requested        Resolved     Latest patch
   > nuget.versioning      [4.0.0, 4.3.0)   4.0.0        4.3.0
   > SlowCheetah.Xdt       1.0.1-beta       1.0.1-beta   1.0.9-beta

>
```

___

#### Highest minor

Example:

_Running in the project folder with the option `--outdated --highest-minor`_
```
> dotnet list package --outdated --highest-minor

Project 'project' has the following updates to its packages
   [netstandard1.0]:
   Top-level Package       Requested        Resolved     Latest minor
   > nuget.versioning      [4.0.0, 4.3.0)   4.0.0        4.7.0
   > SlowCheetah.Xdt       1.0.1-beta       1.0.1-beta   1.1.7-beta

   [net45]:
   Top-level Package       Requested        Resolved     Latest minor
   > nuget.versioning      [4.0.0, 4.3.0)   4.0.0        4.7.0
   > SlowCheetah.Xdt       1.0.1-beta       1.0.1-beta   1.1.7-beta

>
```
__Behavior clarification:__
- `--highest-patch` and `--highest-minor` will be constrained by the patch and minor, repectively, of the **resolved** version.
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

___

### Out of scope: Deprecated

- Listing deprecated packages will not be supported in the current version for `list package`. The feature is likely to be implemented once deprecation is implemented on the server side as planned in this pull request.
