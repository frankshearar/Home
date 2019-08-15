* Status: **Reviewed**
* Author(s): [Anand Gaurav](https://github.com/anangaur) ([@adgrv](https://twitter.com/adgrv)), [Xavier Decoster](https://github.com/xavierdecoster) ([@xavierdecoster](https://twitter.com/xavierdecoster))

## Issue
Deprecate obsolete, legacy packages [#2867](https://github.com/NuGet/Home/issues/2867)

## Problem
1. There are certain cases when a package author should be able to let the package consumers know that a certain package (version) should no longer be used i.e. **deprecated**. As summarized in one of the comments on the issue, following are the most important cases for package deprecation:
  * This package is legacy and no longer maintained
  * The package has bug/issues that makes the package unusable
  * Other: (other custom reasons like performance, privacy, private package, etc.) 

  Today, package authors can un-list a package but there is no explicit feedback to the package consumers that a certain package version should no longer be used once its already in the project's list of packages (direct or full closure including transitive packages). 

2. In addition to the above, there is no mechanism for the package authors to suggest a new package or a new version to the package consumers where the issue (due to which the package was deprecated) has been fixed.

## Out of scope
* Deprecating packages because it contains a vulnerability. This is a separate feature and is maintained here: [\[Security\] Flag vulnerable packages #8087](https://github.com/NuGet/Home/issues/8087)

## Solution
### Marking a package as deprecated (Server)
Package publishers should be able to mark package (and version(s)) as **deprecated** with one of the reasons as mentioned above.

### Publisher experience (on NuGet.org)
1. Package author goes to the `Packages`\\`<PackageID>`\\**`Manage`** page and expands the **`Deprecate package(s)`** section.

2. Is required to **select version(s)**: By default, the version from the package version page which led to this page, is selected. Authors can choose one or many versions.

3. **Select reason(s)**: One or many of the following reasons can be selected
  
  * This package is **legacy** and no longer maintained
  * The package has **critical bugs** that make it unusable
  * **Other**: (other custom reasons like performance, privacy, private package, etc.).

4. **Provide alternate package** (optional)

  * The package needs to be an existing package on NuGet.org.
  * Once a valid package is selected, the version dropdown is populated with available versions. By default it is set to `Latest stable`. (If all versions are pre-release, it states `Latest`).
  * While selecting alternate recommended package, there would be auto-complete and verification built-in for existing packages. If a non-existent package ID is entered, the save button will be disabled and error message printed.
  * The alternate package cannot be an unlisted package at the time of deprecation.
  * The alternate package can be the same package ID as the one being deprecated, but the alternate package version may not be (one of) the version(s) being deprecated.

5. **Provide custom message** (optional)

  * This is mandatory if the reason `Other` is selected.
  * This is free text field that authors can fill in with any valid reason.
  * [**Open**] Supports English only or UTF-8 characters.
  * This message will currently not be exposed in NuGet clients.

![image](https://user-images.githubusercontent.com/14800916/56618151-9349ba00-65d6-11e9-9ab3-372bc47f0c96.png)

  
#### Additional scenarios

* What happens when multiple versions are selected that had existing deprecation states?
  
A: The changes are additive in nature where possible. i.e. for reasons, its additive. Alternate package and custom message fields will get overwritten. When such selections are done, There are warnings for each of the sections stating this. E.g. of warning message
  * Select reason(s): `Some of the package versions selected were already deprecated. The reasons selected here will be `**`added `**`to the existing reasons for those versions.`
  * Provide alternate package: `Some of the package versions selected were already deprecated. The alternate package selected here will  `**`override `**`any existing details provided.`
  * Provide custom message: `Some of the package versions selected were already deprecated. The custom message provided here will `**`override `**`any existing details provided.`
  
* How to undeprecate package versions?

A: The same way as deprecate packages. Just clearing all the fields and saving it will un-deprecate. There will be a confirmation window to confirm this action.

### CLI (author) experience

There is no client command required for the MVP. 

## Consumer experience
### NuGet.org
The deprecation information needs to be shown on nuget.org below the package ID+version - minimized by default and detailed when expanded. E.g.

Minimized:
![image](https://user-images.githubusercontent.com/14800916/56994087-f3021100-6b52-11e9-82a5-6d3f4b66213b.png)

Expanded:
![image](https://user-images.githubusercontent.com/14800916/56994111-057c4a80-6b53-11e9-83db-e8ddc4f568f0.png)


### Visual Studio
This deprecation information should be shown on:
1. [**P0**] PMUI Installed tab

Minimized:
![image](https://user-images.githubusercontent.com/14800916/57113048-9855ea00-6cf7-11e9-8324-b17d64bff8b0.png)

Expanded:
![image](https://user-images.githubusercontent.com/14800916/57117455-035fea80-6d11-11e9-9ea6-870d5edb0a12.png)

2. PMUI Updates tab
Not required for MVP. The details page will show the deprecated information as shown for the `Installed` section.

3. In the package details page upon selection of a deprecated package
The details page will show the deprecated information as shown for the `Installed` section. 

4. [**Cut**]* Upon restore .
   1. Information message, by default, on restore (for verbosity = normal and above). This is not applicable for NoOp restore. i.e. the deprecation message will only be shown when a full restore takes place that includes re evaluation of dependency graph.
   2. [**Not MVP**] Warning message, upon property setting, on restore i.e. when `RestoreWarnForDeprecatedPackage` set to `true`. An equivalent available in nuget.oconfig as well and PM UI as well.

\* 4 is **CUT** due to the following reasons and open questions. This may be re-visited based on how we implement the restore time auditing of packages for vulnerabilities. 
* Currently there is caching at mutliple levels and hence latest deprecation information may not be feasible without degrading `restore` perf significantly
* Should this check be offline or always online?
* Should this be a general protocol that any NuGet Server can implement? Or nuget.org only?
* Should this information be cached in Global Packages Folder? If so, how can users get out of the stale deprecation information? Same as first question.
* Privacy concerns.

### CLI

#### Definitions

* An **outdated** package has a newer version available in one of the package sources.
* A **deprecated** package is marked as deprecated in one of the package sources.
* A **deprecated** package is not necessarily **outdated**, but an outdated package may be deprecated.

#### Scope

* Add marker `(D)` next to deprecated package versions in output of `dotnet list package --outdated`
* Support new `dotnet list package --deprecated` command
* Prefer optimal output formatting to facilitate parsing deprecation information

#### Out-of-scope:

* Detecting deprecated packages currently does not result in error codes to support breaking builds on CI systems
* Parseable output format using `--output json` is useful, but demands a spec of its own for this command, and is not restricted to deprecation info. This is tracked by issue [NuGet/Home#7752](https://github.com/NuGet/Home/issues/7752).
* Additional verbosity levels are a potential future expansion, but out of scope for the MVP of this feature.
* We won't show `author message` as part of deprecation info at this point.
* We won't be flagging deprecated packages yet during or after `restore`.

#### Help

The `--help` output must be updated to include the new `--deprecated` option. Other existing options should refer to the `--deprecated` option as needed.

```
Usage: dotnet list <PROJECT | SOLUTION> package [options]

Arguments:
  <PROJECT | SOLUTION>   The project or solution file to operate on. If a file is not specified, the command will search the current directory for one.

Options:
  -h, --help                                Show command line help.
  --outdated                                Lists packages that have newer versions.
  --deprecated                              Lists packages that have been deprecated.
  --framework <FRAMEWORK | FRAMEWORK\RID>   Chooses a framework to show its packages. Use the option multiple times for multiple frameworks.
  --include-transitive                      Lists transitive and top-level packages.
  --include-prerelease                      Consider packages with prerelease versions when searching for newer packages. Requires the '--outdated' or '--deprecated' option.
  --highest-patch                           Consider only the packages with a matching major and minor version numbers when searching for newer packages. Requires the '--outdated' or '--deprecated' option.
  --highest-minor                           Consider only the packages with a matching major version number when searching for newer packages. Requires the '--outdated' or '--deprecated' option.
  --config <CONFIG_FILE>                    The path to the NuGet config file to use. Requires the '--outdated' or '--deprecated' option.
  --source <SOURCE>                         The NuGet sources to use when searching for newer packages. Requires the '--outdated' or '--deprecated' option.
```

#### dotnet list package --outdated

Lists which installed packages have newer versions available.

If any of the *outdated* packages is deprecated, a `(D)` marker is printed next to the version indicating the package is deprecated. 

**Important! Installed packages that are deprecated but not outdated will not be part of the output.**

A description is printed to the console explaining what the `(D)` marker means.

```
> dotnet list package --outdated

The following sources were used:
   nuget.org - https://api.nuget.org/v3/index.json
   Local - C:\NuGet\NuGetLocal

3 packages need your attention - 2 outdated, 1 deprecated.

Package                Current     Wanted      Latest   
EntityFramework        6.1.2       6.1.2       6.2.0   
NUnit                  2.4.0       2.6.4       3.8.1  
My.Sample.Pkg          2.1.3 (D)   4.1.0       4.1.0

(D): Deprecated package(s). Use 'dotnet list package --deprecated' for more info.
```

#### dotnet list package --deprecated

Lists which installed packages have been deprecated.

```
> dotnet list package --deprecated

The following sources were used:
   nuget.org - https://api.nuget.org/v3/index.json
   Local - C:\NuGet\NuGetLocal

Project `ClassLibrary1` uses the following deprecated packages
   [netcoreapp2.0]:
   Top-level Package              Resolved     Reason                 Alternative
   > My.Legacy.Package            2.0.0        Legacy                 My.Awesome.Package >= 3.0.0
   > My.Buggy.Package	          1.1.0        Critical Bugs          My.NotBuggy.Package >= 2.0.0
   > My.Deprecated.Package        3.2.1        Other                  My.NotBuggy.Package >= 2.0.0
   > My.CompletelyBroken.Package  0.9.0        Legacy, Critical Bugs  My.Awesome.Package >= 1.0.0

> To see all packages including transitive packages, additional option `--include-transitive` can be used. 
```

#### dotnet list package --outdated --deprecated

This is a weird scenario:
* Either the output is filtered to search for newer packages;
* or the output is filtered to search for deprecated packages (which are not necessarily outdated)

Combining `--outdated` and `--deprecated` options will not be supported.

Proposed error message:

```
Invalid command. Combining '--outdated' and '--deprecated' options is not supported.
```

#### Flagged during/after `restore`

During restore, NuGet should inform with the same text as shown in the VS UI.

* **Out of scope**:
  * Showing deprecation warning if the package is served from a repository that does not support deprecation.
  * Ability to show deprecation warnings for packages served from file/UNC based repositories.
  * [**CUT**] NuGet `restore` showing warning consistently when the same package is hosted on more than 1 repository where one repository has deprecated the package but others have not. This is because `restore` picks up the package from the source that responds the fastest. **Note**: This information should be shown on package listing (VS package manager or CLI) i.e. even if one of the sources has the deprecation information. 
  * [**CUT**] Showing deprecation warning if package is sourced from a repository that does not have deprecation information even if it hosts packages from another repository, supporting deprecation functionality, as upstream or offline copy.