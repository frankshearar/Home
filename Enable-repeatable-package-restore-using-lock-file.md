* Status: Incubation
* Author(s): [Anand Gaurav](https://github.com/anangaur) ([@adgrv](https://twitter.com/adgrv))

## Context

* Spec: [[Centrally managing NuGet packages]] - summarizes the requirements around managing packages at a solution or repo level
* Spec: [[Centrally managing NuGet package versions]] - detailed specification for managing packages at a solution or a repo level.

## Requirements
Developers would like to have repeatable builds (restores) across time and space

| # | Problem statement |
|:--- |:---------------|
| 1 | Developers do not have confidence that NuGet will restore to the same full closure of package dependencies when they build it on Dev machine vs. CI/CD machines |
| 2 |  Developers would like to be aware of any unintended changes to their package dependency closure including transitive ones |

Input to NuGet is a set of Package References from the project file (Top-level/Direct dependenices) and the output is a full closure/graph of all the package dependencies including transitive dependencies. Ideally, NuGet should always produce the same full closure of package dependencies if the input PackageReferences do not change. NuGet tries to do this but in some cases it is unable to do this:

* You use floating versions like `<PackageReference Include="My.Sample.Lib" Version="4.*"/>`. While your intention is to float to the latest version as required but only with an explicit gesture but also want to lock to specific version even with floating version across time/space, unless I ask NuGet to float during the restore.

* A newer version of the package matching PackageReference version requirements is published. E.g. 

  Day 1: if you specified `<PackageReference Include="My.Sample.Lib" Version="4.0.0"/>` but the versions available on the 
  NuGet repositories were 4.1.0, 4.2.0 and 4.3.0. In this case, NuGet would have resolved to  4.1.0 (nearest match)

  Day 2: Version 4.0.0 gets published. NuGet will now find the exact match and start resolving to 4.0.0

* A given package version is removed from the repository. Though nuget.org does not allow package deletions, not all package repositories have this constraints. Private/Internal repositories including folder/share based repositories may allow package deletions as well. This will result in NuGet finding the best match when it cannot resolve to the deleted versions and thereby changing the full closure of dependencies for the project.

* A repository you installed the package version from is no longer online or is degraded. In case when you have listed multiple repositories as sources in the nuget.config file, NuGet picks the package from the repository that responds fastest. So if other repositories have the same package but different versions of the package, the resolved version may be different.

  The same problem can happen if you have different nuget.config files with different sources (repositories) at different 
  places. E.g. Dev machines may have an additional local share repository while CI/CD machine may not.
 
* If the same package ID+version resolves to different packages across different sources, NuGet cannot ensure the same package(with the same hash) will be resolves every time. It will also not warn/error out in such cases.

* [Future - R3] Users have been asking for an ability to define the resolution strategy of transitive dependencies as it existed with package.config. Once we implement this feature, when a then any update to a transitive package on repository can change the full closure of package dependencies.

  E.g. If Project1 depends on PackageA(v1.0.0) which depends on PackageB(>=2.0.0)

  `Project1--> PackageA(1.0.0) --> PackageB(>=2.0.0)` 

  Today NuGet, by default, pins to the lowest version for any transitive dependency. And hence any update to PackageB does 
  not have any impact on the resolved packages graph in the above case. But once we enable this feature and let users 
  decide to float to the latest version, with every update to PackageB on the repository, will have an impact on the 
  resolved version in your project during restore.

In addition to the above, when developers add/remove a package dependency to a project, they would like to be told about any unintended changes in the full package dependency closure. (PRS2)

Consider the following scenario:
 
`Project1 --> PackageA(1.0.0) --> PackageB(>=2.0.0)` 

Now lets say you bring in another dependency on PackageX(3.0.0) with some transitive dependencies. And suddenly you build breaks because the PackageX(3.0.0) had the following transitive dependencies:

```
Project1--> PackageA(1.0.0) --> PackageB(>=2.0.0)
       |--> PackageX(3.0.0) --> PackageB(>=4.0.0)
```

So now instead of PackageB(2.0.0), NuGet resolves to PackageB(**4.0.0**) that may have breaking changes. Obviously this is due to an intentional package install but the transitive closure happens behind the scenes without letting the users know the changes in transitive dependency versions. Sometimes this is not ideal. Users would like to know the difference in package dependency graphs irrespective of whether the change is related to direct or indirect/transitive dependencies.


## Solution - Summary
* A lock file has the package dependency graph for the project/solution/repo that includes both the direct as well as transitive dependencies.
* Lock files will used when the property `RestoreWithLockFile` is set in the context of the project.
* Lock file will be updated when a package is added or updated.
* NuGet will use a lock file to `restore` packages. 
  * If a lock file is present and is **not** out of sync (with user changes), `restore` will use the lock file to fetch all the packages. 
  * If a lock file is not present or **out of sync**, `restore` will create/update the lock file with the latest changes.
  * There will be modes (using MSBuild property and *(Not MVP)* command line options) to control the behavior of `restore` with lock files i.e. whether NuGet can update lock file with `restore` action.
* There will be 2 scopes for the creation/working of the lock file:
  * At project level - In this case the lock file is created per project.
  * At a central level when the [packages are managed at a solution or a repo level](https://github.com/NuGet/Home/wiki/Centrally-managing-NuGet-packages) - In this case the lock file is also created centrally in the same folder as the `packages.props` file.

## Solution - Details 

### Bootstrapping/Enabling the lock file

### Lock file format and details captured in it

### Lock file working 

### Project vs. Central lock file 

### Extensibility (Not MVP)

### Visual Studio Experience

Details
* If a central package version management file (default `packages.props` file is present, NuGet will not just use this file for manage package versions, but it will also create a lock file - `packages.lock.json` in the same folder as the central package version management file. This file will have the full package closure - direct and transitive across the projects in a repo.
* If you `restore` in a project's context, NuGet will refer the `packages.lock.json` to get the package closure and restore those packages.
* If there is any of the following discrepancies while resolving package dependencies for a project, `restore` will error out.
  * `PackageReference` in project file does not have a corresponding entry in the `packages.lock.json` and/or `packages.props`
  * Version mismatch between `packages.lock.json` and `packages.props`
* In order to update the lock file with new restore graph, you can run `restore` with option `update-lock-file`:
  ```
  > dotnet restore --update-lock-file
  ```
* Specifying a `<Package>` node in the `packages.props` does not mean the lock file will also contain this package information. The lock file is updated on a package reference addition to a project. 

*packages.lock.json*
```
{	
  "version": 1.0,	
  "metadata1":"value1",
  ...other metadata fields...
  "dependencies": {	
    "netcoreapp2.0": {	
      "Contoso.Base": {	
        "type": "direct",	
        "requested": "3.0.0",	
        "resolved": "3.0.0",
        "integrity":"SHA512-#fVXsnMP2Wq84VA533zj0a/Et+QoLoeNpVXsnMP2Wq84l+hsUxfwunkbqoIHIvpOqwQ/+HIvprVKs+QOihnkbqod=="		
      }	
  ...	
```
