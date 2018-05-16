* Status: Incubation
* Author(s): [Anand Gaurav](https://github.com/anangaur) ([@adgrv](https://twitter.com/adgrv))

## Requirements

As Noah, a developer who uses NuGet in an enterprise,
* I would like to manage all the package versions centrally at a repo level so that they do not go out of sync in the various projects in the repo.
* I can specify a range or float to latest version of a given package so that all projects in the repo always gets the latest available version when desired.
* I can lock down the package graph for the repo so that my builds are repeatable
  * I can lock down direct and transitive dependencies so that the build behavior does not change by resolving to a different version of a package version due to unavailability of previously resolved version.
  * I am able to lock down the integrity of the package so that so that the build behavior does not change by resolving to a different package with same ID+version. 
  * I am able to lock the floating versions of packages to their respective resolved versions unless I explicitly ask the package to resolve to latest version. 
* When I modify a common project that's a dependency of multiple projects in the repo, I should not be required to multiple checkins corresponding to each of the dependent projects.
  * E.g. Project A->B->C->D->...->X. If I change the `PackageReference` for X, I should be making limited changes related to only Project X and not multiple changes for each of the projects that depend on it. (This would be case with per project lock file).

Extensibility requirements:
* I am able to define a different set of packages and versions for a specific project in the repo that differs from the centrally managed packages+versions.
  * Correspondingly, I am able to lock down a different package graph for a specific project.
* (?) I am able to define package versions based on conditions.
  * Correspondingly, I am able to lock down different graphs based on these conditions.

Content Governance requirements (P2)
* I am able to scan the packages (full closure) used in the repo to flag non-compliance, licensing and vulnerabilities.

# Context
[PackageReference requirements summary](https://github.com/NuGet/Home/wiki/PackageReference-enhancements) | Epic issue [#6763](https://github.com/NuGet/Home/issues/6763)

## Who is the customer?
Customers with huge code-base spanning 100s of projects. 

## Evidence

*Central package version management*
* Developers need this and have worked their way around in multiple ways. Some define the package version as a variable and then define all the version variables in a central file to control this behavior. E.g.
  ```
  <PackageReference Include="My.Sample.Lib" Version="$(MySampleLibVersion)" />
  ```
* MSBuild team has built an SDK project to implement this behavior: [Microsoft.Build.CentralPackageVersions](https://github.com/Microsoft/MSBuildSdks/tree/master/src/CentralPackageVersions#microsoftbuildcentralpackageversions). This spec has taken inspiration from this.
 
*Repeatable builds*
We have had multiple internal partners reaching out to us from VS Team Services, Bing, Windows, Azure for this feature. Customers and community members have also asked for this feature. Refer to the following GitHub issues and comments on them:
* Lineups #2572 <https://github.com/NuGet/Home/issues/2572> 
* Why must resolve to Lowest Version? Allow users to determine package resolution strategy during package restore #5553 <https://github.com/NuGet/Home/issues/5553> 
* Twitter thread:
  * https://twitter.com/stimms/status/885268856960196612

## Solution

### Summary

**Note:**  Assumption is that the central packages version management file (CPVMF) is a MSBuild file assuming MSBuild would provide free parsing logic (and other benefits) for the file. If not, it can be any format that NuGet can parse (including nuget.config). The exact format is TBD and is implementation detail. The rest of the doc assumes it to be a MSBuild props file. 
 
* Package Versions can be centrally managed in `packages.props` file located at the solution or repo root.
* Package references (`PackageReference`) managed at each project level without any version information.
* Managing packages for the repo/projects:
  * Adding packages references not listed in `packages.props` will be an error by default. An option to update `packages.props` file as part of adding the package reference will be available.
  * Updating package reference per project will be an error. An option to update in the `packages.props` file will be available.
  * Removing/uninstalling package references per project is allowed. There will be an option to do the same in the `packages.props` file.
* All the referenced packages and their transitive dependencies are locked in a lock file - `packages.lock.json` present at the level (same folder) as the `packages.props` file.
  * NuGet `restore` action, by default, will always resolve the dependencies using the lock file.
    * It will check if a referenced package (`PackageReference` in the project file) is already present in the `packages.props` file as well as the `packages.lock,json` file. If not, it errors out.
    * It will check if the version specified in `packages.props` match with `packages.lock.json`. If not, it errors out.
    * The lock file contains the integrity data (SHA) for each of the packages listed in it. Restore does a post step to validate the integrity of the packages. It errors out if the integrity check fails. This option to include integrity will be configurable using a MSBuild/NuGet property.
  * NuGet `restore -update-lock-file` action will be able to recompute dependencies and overwrite the lock file. A similar experience on VS will be available.
  * NuGet install/update actions will modify the lock file if it modifies `packages.props` file.

### Solution Details
* [[Centrally managing NuGet package versions]]
* [[Enable repeatable package restore using lock file]]