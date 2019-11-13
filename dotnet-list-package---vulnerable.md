* Status: **Reviewing**
* Authors: [Xavier Decoster](https://github.com/xavierdecoster)

## Issue

[8716](https://github.com/nuget/home/issues/8716) - Support Package Vulnerability feature in clients (phase 1)

## Problem Background

Developers take dependency on a set of packages directly or indirectly (through transitive dependencies) and have no way to understand if any of their dependencies bring in any known vulnerabilities.

NuGet should be able to flag vulnerable packages used in projects/solutions. To support this capability from the commandline, a new `dotnet.exe` CLI option will be introduced.

This CLI command provides a way to verify *already installed* (direct and indirect) dependencies against known vulnerabilities.

## Who are the customers

All .NET Core customers.

## Requirements

It must be possible to combine the `--vulnerable` command option with the following flags:

* `--include-transitive`
* `--source`
* `--config`
* `--framework`
* `--interactive`
* `--help`

Combining the `--vulnerable` command option with the following flags will not be supported at this time:

* `--outdated`
* `--deprecated`
* `--offline`

The following flags will be ignored (but not error) when combined with the `--vulnerable` command option:

* `--include-prerelease` (currently requires `--outdated` option)
* `--highest-minor` (currently requires `--outdated` option)
* `--highest-patch` (currently requires `--outdated` option)

An informational message will be printed when the ignored command options are used:

```shell
The command option(s) '--include-prerelease', '--highest-minor', and '--highest-patch' are ignored by this command.
```

## Solution

We'll introduce a new `--vulnerable` command option to the existing `dotnet list package` command.
This command option will offer a detailed view of already installed packages with known vulnerabilities.

The default behavior for `dotnet list package` will be to add a marker next to the packages that have known vulnerabilities (as well as those that are known to be deprecated or outdated). This is a change in behavior, as the default `dotnet list package` command will now try to reach servers online to get additional metadata.

To continue supporting the old behavior, a new `--offline` command option will be introduced.

### DotNet CLI

The `dotnet` CLI repository must be updated to:

* forward the `--vulnerable` command option for the `dotnet list package` command,
* forward the `--offline` command option for the `dotnet list package` command,
* handle the invalid combinations of command options, such as `--vulnerable --outdated`, `--vulnerable --deprecated`, and any combination of the new `--offline` option with either `--outdated`, `--deprecated`, or `--vulnerable` options.

### NuGet xplat CLI

The `nuget` xplat CLI code must be updated to:

* handle the `--vulnerable` command option for the `dotnet list package` command,
* handle the `--offline` command option for the `dotnet list package` command,
* handle the invalid combinations of command options, such as `--vulnerable --outdated`, `--vulnerable --deprecated`, and any combination of the new `--offline` option with either `--outdated`, `--deprecated`, or `--vulnerable` options.

### Command Output

#### Help

The `--help` output must be updated to include the new `--vulnerable` and `--offline` options. Other existing options should refer to the `--vulnerable` and `--offline` options as needed.

```shell
Usage: dotnet list <PROJECT | SOLUTION> package [options]

Arguments:
  <PROJECT | SOLUTION>   The project or solution file to operate on. If a file is not specified, the command will search the current directory for one.

Options:
  -h, --help                                Show command line help.
  --outdated                                Lists packages that have newer versions. Cannot be combined with `--deprecated`, `--vulnerable`, or `--offline` options.
  --deprecated                              Lists packages that are deprecated. Cannot be combined with `--outdated`, `--vulnerable`, or `--offline` options.
  --vulnerable                              Lists packages that have known vulnerabilities. Cannot be combined with `--outdated`, `--deprecated`, or `--offline` options.
  --offline                                 Prevents online resources to be queried for information about newer package versions, deprecation metadata, or known vulnerabilities. Cannot be combined with `--outdated`, `--deprecated`, or `--vulnerable` options.
  --framework <FRAMEWORK | FRAMEWORK\RID>   Chooses a framework to show its packages. Use the option multiple times for multiple frameworks.
  --include-transitive                      Lists transitive and top-level packages.
  --include-prerelease                      Consider packages with prerelease versions when searching for newer packages. Requires the '--outdated' option.
  --highest-patch                           Consider only the packages with a matching major and minor version numbers when searching for newer packages. Requires the '--outdated' option.
  --highest-minor                           Consider only the packages with a matching major version number when searching for newer packages. Requires the '--outdated' option.
  --config <CONFIG_FILE>                    The path to the NuGet config file to use. Requires the '--outdated', '--deprecated', or `--vulnerable` option.
  --source <SOURCE>                         The NuGet sources to use when searching for newer packages. Requires the '--outdated', '--deprecated', or `--vulnerable` option.
```

#### List

The default behavior for `dotnet list package` will be to reach out to the configured online sources to retrieve information about newer package version, deprecation metadata, and known package vulnerabilities. The output will be updated to include markers for each.

```shell
dotnet list package
```

Lists the package references for a project or solution.

```shell
> dotnet list package

The following sources were used:
   nuget.org - https://api.nuget.org/v3/index.json
   Local - C:\NuGet\NuGetLocal

Project 'MySampleProject' has the following package references
   [netcoreapp2.1]:
   Top-level Package               Requested   Resolved
   > Microsoft.ML                  0.11.0      0.11.0
   > Microsoft.NETCore.App   (A)   [2.1.0, )   2.1.0
   > My.Outdated.Package           0.1.0       0.1.0 (O)
   > My.Deprecated.Package         1.0.0       1.0.0 (D)
   > My.Vulnerable.Package         1.0.1       1.0.1 (V)

(A): Auto-referenced package(s).
(O): Outdated package(s). Use 'dotnet list package --outdated' for more info.
(D): Deprecated package(s). Use 'dotnet list package --deprecated' for more info.
(V): Vulnerable package(s). Use 'dotnet list package --vulnerable' for more info.
```

#### Outdated

```shell
dotnet list package --outdated
```

Lists which installed packages have newer versions available.

If any of the outdated packages is vulnerable, a `(V)` marker is printed next to the version indicating the package version is known to be vulnerable.

*Combining the `--vulnerable` option with the `--outdated` option is (currently) not supported. When doing so, the following error message will be presented:*

```shell
> dotnet list package --outdated --vulnerable

Invalid command. Combining '--outdated' and '--vulnerable' options is not supported.
```

**Important! Installed packages that are vulnerable but not outdated will not be part of this command's output.**

A description is printed to the console explaining what the `(V)` marker means.

```shell
> dotnet list package --outdated

The following sources were used:
   nuget.org - https://api.nuget.org/v3/index.json
   Local - C:\NuGet\NuGetLocal

3 packages need your attention - 2 outdated, 1 deprecated, 1 vulnerable.

Package                Requested   Resolved    Latest
EntityFramework        6.1.2       6.1.2       6.2.0
NUnit                  2.4.0 (D)   2.6.4       3.8.1  
My.Sample.Pkg          2.1.3 (V)   4.1.0       4.1.0

(D): Deprecated package(s). Use 'dotnet list package --deprecated' for more info.
(V): Vulnerable package(s). Use 'dotnet list package --vulnerable' for more info.
```

*Combining the `--outdated` option with the `--offline` options is not supported. When doing so, the following error message will be presented:*

```shell
> dotnet list package --outdated --offline

Invalid command. Combining the '--outdated' and '--offline' options is not supported.
```

#### Deprecated

```shell
dotnet list package --deprecated
```

Lists which installed packages have been deprecated.

If any of the deprecated packages is vulnerable, a `(V)` marker is printed next to the version indicating the package version is known to be vulnerable.

*Combining the `--deprecated` option with the `--vulnerable` option is (currently) not supported. When doing so, the following error message will be presented:*

```shell
> dotnet list package --deprecated --vulnerable

Invalid command. Combining '--deprecated' and '--vulnerable' options is not supported.
```

**Important! Installed packages that are vulnerable but not deprecated will not be part of this command's output.**

A description is printed to the console explaining what the `(V)` marker means.

```shell
> dotnet list package --deprecated

The following sources were used:
   nuget.org - https://api.nuget.org/v3/index.json
   Local - C:\NuGet\NuGetLocal

Project `ClassLibrary1` uses the following deprecated packages
   [netcoreapp2.0]:
   Top-level Package              Requested  Resolved    Reason                 Alternative
   > My.Legacy.Package            2.0.0      2.0.0       Legacy                 My.Awesome.Package >= 3.0.0
   > My.Buggy.Package             1.1.0      1.1.0       Critical Bugs          My.NotBuggy.Package >= 2.0.0
   > My.Deprecated.Package        3.2.1      3.2.1       Other                  My.NotBuggy.Package >= 2.0.0
   > My.CompletelyBroken.Package  0.9.0 (V)  0.9.0       Legacy, Critical Bugs  My.Awesome.Package >= 1.0.0

> To see all packages including transitive packages, additional option `--include-transitive` can be used.
(V): Vulnerable package(s). Use 'dotnet list package --vulnerable' for more info.
```

The table output for `--deprecated` should also include a new `Requested` column to improve clarity and for consistency with `--outdated` and `--vulnerable` commands.

*Combining the `--deprecated` option with the `--offline` options is not supported. When doing so, the following error message will be presented:*

```shell
> dotnet list package --deprecated --offline

Invalid command. Combining the '--deprecated' and '--offline' options is not supported.
```

#### Vulnerable

```shell
dotnet list package --vulnerable
```

Lists which installed packages have known vulnerabilities.

```shell
> dotnet list package --vulnerable

The following sources were used:
   nuget.org - https://api.nuget.org/v3/index.json
   Local - C:\NuGet\NuGetLocal

Project `ClassLibrary1` uses the following vulnerable packages
   [netcoreapp2.0]:
   Top-level Package              Requested  Resolved    Vulnerability
   > My.Vulnerable.Package        2.0.0      2.0.0       https://URL-to-vuln-details
   > My.Fixed.Package             1.1.0      1.1.0       https://URL-to-vuln-details

 > To see transitive packages, add the `--include-transitive` option.
 > To see prerelease packages, add the `--include-prerelease` option.
```

*Combining the `--vulnerable` option with the `--offline` options is not supported. When doing so, the following error message will be presented:*

```shell
> dotnet list package --vulnerable --offline

Invalid command. Combining the '--vulnerable' and '--offline' options is not supported.
```

*Combining the `--vulnerable` option with the `--deprecated` option is (currently) not supported. When doing so, the following error message will be presented:*

```shell
> dotnet list package --vulnerable --deprecated

Invalid command. Combining '--vulnerable' and '--deprecated' options is not supported.
```

*Combining the `--vulnerable` option with the `--outdated` option is (currently) not supported. When doing so, the following error message will be presented:*

```shell
> dotnet list package --vulnerable --outdated

Invalid command. Combining '--vulnerable' and '--outdated' options is not supported.
```

#### Flagged during/after restore

This is out of scope for the MVP of this feature, but is on the roadmap for a future iteration.

#### Examples

* List vulnerable package references of a specific project (top-level dependencies only):

  ```shell
  dotnet list package MyProject.csproj --vulnerable
  ```

* List vulnerable package references, including transitive dependencies:

  ```shell
  dotnet list package --vulnerable --include-transitive
  ```

* List vulnerable package references for a specific target framework:

  ```shell
  dotnet list package --vulnerable --framework netcoreapp3.0
  ```

### Open Questions

* We need a single command to show *all updates, vulnerabilities and deprecations* as well. How? `nuget audit`?

## References

* [NuGet/Home#8087](https://github.com/NuGet/Home/issues/8087)
* [NuGet/Home#8716](https://github.com/nuget/home/issues/8716)
* [dotnet list package docs](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-list-package)