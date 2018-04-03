Status: **Incubation**

# Issue
| # | Requirement | Issue # | 
|:--- |:---|:---|
| **R1** | **Developers would like to have repeatable builds (restores) across time and space** | **[#5602](https://github.com/nuget/home/issues/5602)** |



# Context
* [PackageReference requirements summary](https://github.com/NuGet/Home/wiki/PackageReference-enhancements) | Epic issue [#6763](https://github.com/NuGet/Home/issues/6763)
* [Package dependency resolution](https://github.com/NuGet/Home/wiki/PackageReference-enhancements#dependency-resolution)
* [NuGet Actions (current)](https://github.com/NuGet/Home/wiki/PackageReference-enhancements#nuget-actions)

# Problem
### R1 - Developers would like to have repeatable builds (restores) across time and space

| # | Problem statement |
|:--- |:---------------|
| PRS1 | Developers do not have confidence that NuGet will restore to the same full closure of package dependencies when they build it on Dev machine vs. CI/CD machines |
| PRS2 |  Developers would like to be aware of any unintended changes to their package dependency closure including trasitive ones |

Input to NuGet is a set of Package References from the project file (Top-level/Direct dependenices) and the output is a full closure/graph of all the package dependencies including transitive dependencies. Ideally, NuGet should always produce the same full closure of package dependencies if the input PackageReferences do not change. NuGet tries to do this but in some cases it is unable to do this:

* A newer version of the package matching PackageReference version requirements is published. E.g. 

  Day 1: if you specified `<PackageReference Include="My.Sample.Lib" Version="4.0.0"/>` but this versions available on the 
  NuGet repositories were 4.1.0, 4.2.0 and 4.3.0. In this case, NuGet would have resolved to  4.1.0 (nearest match)

  Day 2: Version 4.0.0 gets published. NuGet will now find the exact match and start resolving to 4.0.0

* A given package version is removed from the repository. Though nuget.org does not allow package deletions, not all package repositories have this constraints. Private/Internal repositories including folder/share based repositories may allow package deletions as well. This will result in NuGet finding the best match when it cannot resolve to the deleted versions and thereby changing the full closure of dependencies for the project.

* A repository you installed the package version from is no longer online or is degraded. In case when you have listed multiple repositories as sources in the nuget.config file, NuGet picks the package from the repository that responds fastest. So if other repositories have the same package but different versions of the package, the resolved version may be different.

  The same problem can happen if you have different nuget.config files with different sources (repositories) at different 
  places. E.g. Dev machines may have an additional local share repository while CI/CD machine may not.
 
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
       |--> PackageX(3.0.0) --> PackageY(3.0.0) --> PackageZ (1.0.0) --> PackageB(4.0.0)
```

So now instead of PackageB(2.0.0), NuGet resolves to PackageB(**4.0.0**) that may have breaking changes. Obviously this is due to an intentional package install but the transitive closure happens behind the scenes without letting the users know the changes in transitive dependency versions. Sometimes this is not ideal. Users would like to know the difference in package dependency graphs irrespective of whether the change is related to direct or indirect/transitive dependencies.

# Who is the customer?

While all users whose projects are PackageReference based would need this feature but for Enterprises this is really crucial (hygiene factor) for their CI/CD scenarios. 

# Evidence

We have had multiple internal partners reaching out to us from VS Team Services, Bing, Windows, Azure for this feature. Customers and community members have also asked for this feature. Refer to the following GitHub issues and comments on them:
* Lineups #2572 <https://github.com/NuGet/Home/issues/2572> 
* Why must resolve to Lowest Version? Allow users to determine package resolution strategy during package restore #5553 <https://github.com/NuGet/Home/issues/5553> 
* Twitter thread:
  * https://twitter.com/stimms/status/885268856960196612

# Solution

The proposal is to create a lock file per project that will persist the package dependency graph for the project and unless the lock file is updated due to an explicit action by the user, restore uses lock file to get all the package dependencies.

## How to enable the Lock file feature?
This will be an intrinsic feature for NuGet and hence the lock file should be generated by default for all NuGet restores. However, the feature would be behind a feature flag to start with. This will let us roll out the feature without impacting developers who do not expect to see an additional file. To enable this feature, one would need to set the following property in NuGet.config:

```
<config>            
     <add key="UseLockFileForRestore" value="true">
</config>
```

### What happens when this property `UseLockFileForRestore` is set to false (not set) but lock file is present in the project's root folder?

If a lock file is present, NuGet will use the lock file for install, update or restore irrespective of whether the property is set or not. 

### What are behaviors for install, update and restore w.r.t. lock file?

#### Package Install
Packages (for PackageReference based projects) can be installed through the following methods:
* VS PM UI `Install` action
* VS PMC `Install-Package` command
* `dotnet add package` command
* **`dotnet install package`** *

\* New command

### What happens to existing projects when the property `UseLockFileForRestore` is set and there is no lock file present?In this case, 
* Any `install` or `update` command/action will create the lock file. 
* `restore` command/action will error out as it cannot find a lock file to use to restore. It will also print a message to indicate which command to run to generate the lock file for the first time.
* `restore --force` command/action will be able to generate the lock file, if not present. 

**Note**: Details on the new commands or changes in their behavior are covered later.

## Lock file properties

* The lock file will be called `nuget.dependencies.lock` 
* The lock is created in the project **root** directory by default. 
* We recommend this file should be checked in to the source repository.
* The file contents should be in order so that the changes to this file can be readable by a user and diff can be easily understood during commits. The lock file should contain all the dependencies in order:
  * All the direct dependencies in alphabetical order first 
  * All the transitive dependencices next, in alphabetical order
  * Each package entry in the lock file should list its direct dependencies.
* [**Not-MVP**] Users can override the path of the lock file using the following options. Details - TBD.
  
### When is lock file created/re-generated?

| Tool+command/action | Can modify/generate lock file? |
|:--- |:---|
| VS PM UI `Install` action | Yes |
| VS PMC `Install-Package` command | Yes |
| `dotnet add package` | Yes |
| **`dotnet install package`** * | Yes |
| **`dotnet update package`** * | Yes |
| `dotnet restore` | **No** |
| `msbuild /t:restore` | **No** |
| `msbuild /restore` | **No** |
| `dotnet restore --force` | **Yes** |
| VS `rebuild` action | **Yes** ? |
| VS `clean` + `build` action | **No** ? |
 
* New commands

The lock file should be created/regenerated in one of the following cases:

* The lock file should be created if it does not exist. 
* If any of the following changes, lock file is modified:
  * Project's direct package dependencies change
  * P2P reference change
  * Target framework(s) change
_[**Not-MVP**] The changes made to the lock file are minimal in nature to improve performance. i.e. if a new package is added then only the new package and its dependent info is added to the lock file without affecting rest of the dependencies (if possible)._

From a user's perspective the following will modify/re-generate the lock file:
* **Update** or **Update All**: An update command should get the latest packages for the project. For floating versions, the update should just update to the latest version as per the floating version constraint without overwriting the floating version specified.
* **Restore -Force**: re-computes the restore graph without updating the package dependencies' versions unless the dependencies have floating versions. 

**Open**: There is a concept of `restore -Force` today in the context of `restore NoOp` and we should evaluate what `restore -Force` should mean that does not confuse users.

Summary of actions with effect on the lock file: 

| User action        | Lock file update?           | Comments  |
|:------------------- |:-------------|:-----|
| Add/Install package   | Yes | Only the new package and dependent tree is updated, if possible. |
| Remove package      | Yes      |  Only the package being removed and its dependent packages are removed without affecting the rest of the dependency graph, if possible  |
| Add a package source (feed) to nuget.config | No |  |
| Add/Remove a project reference | Yes | |
| Add/Remove/Change target framework | Yes | |
| First restore | Yes | when lock file is not present |
| Successive restores | No | |
| ReBuild | No | |
| Clean; Build | No |
| Update one/all dependencies | Yes | |
| restore --force | Yes | This forces re-evaluation of package dependency tree and hence lock file may be modified |

### Lock file format
The lock file needs to contain all the information needed to make the decision to modify/re-generate the lock file based on the requirements mentioned in the above section.

Options:
* Simple format (similar to yarn.lock). Eg.

``` 
# THIS IS AN AUTO-GENERATED FILE. DO NOT EDIT THIS FILE DIRECTLY.
# YOU SHOULD CHECK THIS FILE INTO YOUR SOURCE REPOSITORY.
# LEARN MORE - https://aka.ms/nuget-lockfile

version: 1.0.0
dependencies:
  netcoreapp:
   packageA-1.0.0:1.0.1#figMxwHAzvZt2VA533zj0a/Et+QoLoeNpVXsnMP2Wq84l+hsUxfwunkbqoIHIvpOqwQ/+YA3rVKs+QOihummfA=="
     packageD-4.0.0
     packageE-5.0.0
     packageF-6.0.0
   packageB-2.0.0:2.0.0#xwHAzvZt2VA533zj0wunkbqoIHIvqfigMxwHAzvZt2VA533figMxwHAzvZt2VA533wQ/+YA3rVKs+QOihummfA=="
     packageE-4.5.0
     packageF-5.9.1
     packageG-6.0.0
   packageC-3.0.0:3.0.0#xwHAzvZt2VA533zj0wunkbqoIHIvqfigMxwHAzvZt2VA533figMxwHAzvZt2VA533wQ/+YA3rVKs+QOihummfA=="
     packageF-6.0.0
     packageG-7.0.0
     packageH-8.0.0
   packageX-2.0.0:2.0.0#xwHAzvZt2VA533zj0wunkbqoIHIvqfigMxwHAzvZt2VA533figMxwHAzvZt2VA533wQ/+YA3rVKs+QOihummfA=="
     packageG-7.0.0
     packageH-11.0.0
     packageY-7.0.0
   packageD-:4.0.0#rasMxwHAzvZt2VA533zj0a/Np+QoLoeNpVXsnMP2Wq84l+hsUxfwunkbqoIHIvpOqwQ/+YA3rVKs+QOihuVKs+Q="
  ...
```

* Json - similar to project.assets.json. Eg.

```
# THIS IS AN AUTO-GENERATED FILE. DO NOT EDIT THIS FILE DIRECTLY.
# YOU SHOULD CHECK THIS FILE INTO YOUR SOURCE REPOSITORY.
# LEARN MORE - https://aka.ms/nuget-lockfile
{
  "version": 1.0,
  "dependencies": {
    "netcoreapp2.0": {
      "Contoso.Base": {
        "type": "direct",
        "requested": "3.0.0",
        "resolved": "3.0.0@https://www.nuget.org/api/v2/package/Contoso.Base/3.0.0#fVXsnMP2Wq84VA533zj0a/Et+QoLoeNpVXsnMP2Wq84l+hsUxfwunkbqoIHIvpOqwQ/+HIvprVKs+QOihnkbqod==",
        "dependencies": {
        }
      }
  ...
```

* A MSBuild props file - What are the benefits?

## Restore options

* By default, the package hash (SHA512) should be captured for every resolved dependency in the lock file. While restoring the packages, the hash should be checked for packages restored (in cache or otherwise). 
  * This behavior can be overridden by setting the following property in the **project file** or in one of the evaluated **NuGet.config** files:

   `<DisablePackageHashInLockFile>True</DisablePackageHashInLockFile>`  _**TBD** - exact property name_

* If the user needs to re-evaluate package dependency graph on each restore, he/she should be able to do this.
  * This is useful when floating versions are used and in Development environment user wants to get to the latest version every time.
  * This will have performance impact and hence the user should be warned about this using a numbered NuGet warning:
  ```
  NUxxxx: Detected the use of `restore -Force`. This option forces restore graph evaluation on every restore and can cause degraded performance.  
  ```
  * This can be done from command line by
  ```
  > nuget.exe restore -Force
  ```
  ```
  > msbuild /t:restore -Force
  ```
  ```
  > dotnet.exe restore --force
  ```
  * Alternatively, VS will have an option to force restore on a solution level restore.
  * [Open] Still debating if we should allow this for each restore by setting the following property in the **project file** or in one of the evaluated **NuGet.config** files: 

    `<ForceRestoreAlways>True</ForceRestoreAlways>`  _**TBD** - exact property name_

  * [Open] Still debating if we should allow this by setting the ENVIRONMENT variable (All supported platforms - Windows, Mac, Linux) - `NUGET_FORCE_RESTORE_ALWAYS` to **true**

## How does it help?
This ensures that any restore which can be part of different builds across place and time, will end up getting the same package dependencies.

Explicit user actions like update, add package dependency and remove package dependency will result in regeneration of the locks file. A message would be printed when this happens. 

It will also result in checkout of the locks file when the package dependency graph changes which means that the developer would know that the package dependencies may have changed. If this happens inadvertently, the lock file changes can be discarded and the original one restored to get the same repeatable build as before.

## Limitations and Out of scope

TBD