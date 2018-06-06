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
* The lock file should be checked into the source repository.
* Lock file will be used if any of the following is true:
  * A lock file is present in the context of the project (either at project level or [centrally](https://github.com/NuGet/Home/wiki/Centrally-managing-NuGet-packages)).
  * If [packages are managed centrally](https://github.com/NuGet/Home/wiki/Centrally-managing-NuGet-packages), a central lock file is always used. 
  * If the property `RestoreWithLockFile` is set in the context of the project.
* Lock file will be updated when a package is added or updated.
* NuGet will use a lock file to `restore` packages. 
  * If a lock file is present and is **not** [out of sync](#out-of-sync) (with user changes), `restore` will use the lock file to fetch all the packages. 
  * If a lock file is not present or **[out of sync](#out-of-sync)**, `restore` will create/update the lock file with the latest changes.
  * There will be modes (using MSBuild property and command line options) to control the behavior of `restore` with lock files i.e. whether NuGet can update lock file with `restore` action.
* There will be 2 scopes for the creation/working of the lock file:
  * At project level - In this case the lock file is created per project.
  * At a central level when the [packages are managed at a solution or a repo level](https://github.com/NuGet/Home/wiki/Centrally-managing-NuGet-packages) - In this case the lock file is also created centrally in the same folder as the `packages.props` file.

## Solution - Details 

### Bootstrapping/Enabling the lock file
Lock file will be used if any of the following is true:
  * A lock file is present in the context of the project (either at project level or [centrally](https://github.com/NuGet/Home/wiki/Centrally-managing-NuGet-packages)).
  * If [packages are managed centrally](https://github.com/NuGet/Home/wiki/Centrally-managing-NuGet-packages), a central lock file is always used. 
  * If the property `RestoreWithLockFile` is set in the context of the project.

By setting `RestoreWithLockFile` to `false`, you can skip using lock file even if the lock file is present in the context of the project. 

A warning will be raised by NuGet for this scenario if a **lock file is present** but the property `RestorePackagesWithLockFile` is set to **false**:
```
NU1xxx: <TBD text>
```
The default name of the lock file will be `packages.lock.json`.

### Lock file format and details captured in it

We need to make sure of the following properties for a lock file:
* The lock file format should be such that it is:
  * Concise
  * Human readable
  * Easy to diff
  * Performant - parsing and processing
* Perception - The lock file should not be mistaken for a MSBuild props file that users can modify and check in.
* The integrity of the package SHA512 or equivalent should be also persisted along with the lock file. 
  * (Not MVP) There should be an option to opt out of this. *Why will this be required?*

*Sample lock file - `packages.lock.json`*
```
{	
  "version": 1.0,	
  "metadata1":"value1",
  ...other metadata fields...
  "dependencies": {	
    "netcore2.0": {	
      "Contoso.Base": {	
        "type": "direct",	
        "requested": "3.0.0",	
        "resolved": "3.0.0",
        "integrity":"SHA512-#fVXsnMP2Wq84VA533zj0a/Et+QoLoeNpVXsnMP2Wq84l+hsUxfwunkbqoIHIvpOqwQ/+HIvprVKs+QOihnkbqod=="
        "dependencies": {
             "Contoso.Core": "1.2.3",
             "Fabrikam.Utilities": "[3.1.0]"
         }		
      }	
      "Contoso.Core": {
        "type": "transitive",	
        "requested": "1.2.3",	
        "resolved": "1.2.3",
        "integrity":"SHA512-#xScnMP2Wq84VA533zj0a/Et+QoLoeNpVXsnMP2Wq84l+hsUxfwunkbqoIHIvpOqwQ/+HIvprVKs+QOihnkbmoq=="
        "dependencies": {
           ...
           ...
         }
  ...	
```

### Lock file working 

Once the feature is enabled,
* `Install `- action will update the lock file, if required. Eg. the following command will not just add `PackageReference` node in the project file (and CPVMF, if versions are managed centrally) but also update the lock file:
  ```
  > dotnet add package My.Sample.Lib
  ```

* `Uninstall `- action will update the lock file, if required. Eg. the following command will not just remove `PackageReference` node in the project file (and CPVMF, if versions are managed centrally) but also update the lock file:
  ```
  > dotnet remove package My.Sample.Lib
  ```

* `Restore `- action will use the lock file to get and restore the full closure of the packages if the lock file is **not [out of sync](#out-of-sync)**. 
  * If the lock file is [out of sync](#out-of-sync), restore command will update the lock file with the latest resolved closure of packages. It will do so with a warning: 
  ```
  NU1xxx: <TBD text>
  ```
  * There would be an option to control the above restore behavior. Refer to the [Extensibility](#extensibility) section for details.

### Project vs. Central lock file 

* If the packages are managed centrally at a solution/repo level, then the lock file will be also generated centrally at the same level as the CPVMF - `packages.props` file. 
* You can also have CPVMF applicable for most projects in a solution/repo [but manage versions separately for a few projects](https://github.com/NuGet/Home/wiki/Centrally-managing-NuGet-package-versions#how-do-i-have-a-given-set-of-package-versions-for-all-the-projects-but-a-different-set-for-a-specific-project) (eg. test projects, legacy projects where the versions requirements do not match with the central version required for most of the projects). 
  * In this case the lock file will also be generated per `packages.props` file i.e. a central lock file for the projects that abide by the CPVMF and separate lock files for the rest few projects.

#### Nuances - per project lock file
* Per project lock file can have lot of duplicate entries if you are consuming the same packages from other projects in your solution/repo.
* There could be different versions of the same package listed in different projects even if a project depends on another project. Eg. ProjectA references Project B with following package dependencies:
  ```
  ProjectA
  |-- Pkg-Q 2.0.0
  |-- ProjectB
      |-- Pkg-Q 3.0.0
  ```

  The lock file for `ProjectA` will list `Pkg-Q 2.0.0` while lock file for `ProjectB` will list `Pkg-Q 3.0.0`
 
* When you have a common project that's a dependency of multiple projects in the repo, you will be required to checkin/commit multiple lock files corresponding to each of the dependent projects in addition to the lock file of the common project.

  E.g. In Project `A->B->C->D->...->X` dependency tree, if you change the `PackageReference` for project `X`, the lock file of not just project `X` changes but when you build the solution, lock files of all the projects right from `A` to `X` will change requiring you to checkin/commit multiple files some of which you never worked on. For these scenarios managing dependencies and lock file at central solution/repo level helps as you have to deal with just one lock file change (+ CPVMF `packages.props` file change). 

#### Nuances - central lock file
* Whenever you manage package versions centrally at a solution/repo level, you get a central lock file for all your projects.
* The lock file lists all the packages mentioned in the CPVMF and not the packages referenced in each of your projects. If you run `restore` centrally i.e. at a solution or a repo folder that has the CPVMF, then the unused packages (packages not referenced by any projects) are garbage collected and removed from the CPVMF and hence the lock file. However there are chances of CPVMF listing more packages than referenced in all the projects under the solution/repo.
* The lock file for the whole solution and not for individual projects. So when you add/remove packages from individual projects that does not impact CPVMF, then this change is not reflected in the lock file. 

**Note**: Package restores would be repeatable irrespective of whether you use per project lock file or a central solution/repo level lock file.

### Extensibility

| Option | Values | Description |
|:--- |:--- |:--- |
| `RestorePackagesWithLockFile` | `true`\| **`false`** | Enables lock file - `packages.lock.json` usage with restore. Default is `false` |
| `UpdateLockFileOnRestore` | **`warn`** | Default option - `restore` will update but warn if lock file is [out of sync](#out-of-sync). |
|| `allow`| `restore` will update the lock file if it is [out of sync](#out-of-sync) but will not warn. |
|| `deny` | `restore` will fail if the lock file is [out of sync](#out-of-sync). Useful for CI builds when you do not want the build to continue if the package closure has changed than what is present in the lock file. | 
| `NuGetLockFilePath` | `<PathToLockFile>` | ***[Not MVP]*** Path to lock file if you want to rename or change the location of the lock file. The name should always be *lock.json. |

### Visual Studio Experience
For MVP, there is no VS experience needed. Except for the error/warning messages that will be visible in VS.

### Key terms
#### Out of sync
The lock file is said to be _out of sync_ with the project when
* A project has different set of dependencies than listed in the lock file.
* A project adds reference to another project that changes the full packages closure.

For projects that have centrally managed packages, the lock file will be _out of sync_ with the `packages.props` when:
* The `packages.props` has different set of dependencies than listed in the lock file.

Summary table:
| Type of change | Modifies lock file? |
|:---- |:--- |
| Add/Remove/Change package reference | Yes |
| Change PrivateAssets/ExcludeAssets/IncludeAssets | Yes |
| Change Target Framework | Yes |
| Change runtime identifier | Yes |
| Add/Remove sources | No | 
| Add/Remove fallback folder | No |
| Change project name | No |