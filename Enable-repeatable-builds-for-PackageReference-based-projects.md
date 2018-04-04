Status: **Incubation**

# Issue
| # | Requirement | Issue | 
|:--- |:---|:---|
| **R1** | **Developers would like to have repeatable builds (restores) across time and space** | **[#5602](https://github.com/nuget/home/issues/5602)** |



# Context
* [PackageReference requirements summary](https://github.com/NuGet/Home/wiki/PackageReference-enhancements) | Epic issue [#6763](https://github.com/NuGet/Home/issues/6763)
* [Package dependency resolution](https://github.com/NuGet/Home/wiki/PackageReference-enhancements#dependency-resolution)

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
This will be an intrinsic feature for NuGet and hence the lock file should be generated by default for all NuGet restores. However, the feature would be behind a feature flag to start with. This will let us roll out the feature without impacting developers who do not expect to see an additional file. To enable this feature, one would need to set the following property in 

NuGet.config:

```
<config>            
     <add key="UseLockFileForRestore" value="true">
</config>
```
 
Project file:
```
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp2.0</TargetFramework>
    <UseLockFileForRestore>true</UseLockFileForRestore>
  </PropertyGroup>
```

### What happens when this property `UseLockFileForRestore` is not set but lock file is present in the project's root folder?

NuGet will warn and continue without using the lock file.
```
NUxxxx: The nuget.dependencies.lock file for this project file is being disregarded because UseLockFileForRestore is not set (or set to false). Consider either deleting the nuget.dependencies.lock or setting UseLockFileForRestore to fix this warning.
```
### What happens to existing projects when the property `UseLockFileForRestore` is set and there is no lock file present?In this case, 
* Any `install` or `update` command/action will create the lock file. 
* `restore` command/action will error out as it cannot find a lock file to use to restore. It will also print a message to indicate which command to run to generate the lock file for the first time.
* `restore --force` command/action will be able to generate the lock file, if not present. 

**Note**: Details on the new commands or changes in their behavior are covered later.

## Which commands can modify the lock file? 

Following is a lost of commands/actions with information whether they can modify the lock file or not:
*If the lock file is not present, any command that can modify the lock file will generate/create it (if `UseLockFileForRestore` is set).*

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

## Lock file properties

* The lock file will be called `nuget.dependencies.lock` 
* The lock is created in the project **root** directory by default. 
* We recommend this file should be checked in to the source repository.
* [**Not-MVP**] Users can override the path of the lock file using the following options. Details - TBD.

### Lock file format
* The file contents should be in order so that the changes to this file can be readable by a user and diff can be easily understood during commits. The lock file should contain all the dependencies in order:
  * All the direct dependencies in alphabetical order first 
  * All the transitive dependencices next, in alphabetical order
  * Each package entry in the lock file should list its direct dependencies.
* The lock file needs to contain all the information needed to make the decision to modify/re-generate the lock file based on the requirements mentioned in the above section.
* The lock file should contain SHA512 hash for each package dependency. Upon restore, NuGet will scan through the packages and and SHA values in the lock file and error out if they do not match.

Options (My recommendation):
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

## Restore scenarios

A normal `restore` action will fail in the following scenarios:
* Manually Addition of a new PackageReference node
* Manual update of PackageReference(s) 
* Addition of a Project reference 
* Target framework changes

In all the above cases, user can run one of the following to update the lock file:

| Action/Command | Result |
|:--- |:---|
|  VS `install`, `dotnet add package <packageID>`,  **`dotnet install package <packageID>`** *  | Installs the package. Modifies the lock file. Does not update the floating version |
| **`dotnet install package`** * without a packageID on the project root folder | Installs any additional package required. Modifies the lock file. Does not update the floating version |
| VS `update`, **`dotnet update package <packageID>`** * | Updates the package.  Modifies the lock file. Does not update the floating version |
| VS `restore force`, `dotnet restore --force`, **`msbuild /t:restore /p:force`**,  **`msbuild /restore /p:force`** | Re-computes the package dependency graph. Updates the floating package version in the lock file to the latest available. Modifies the lock file if required. |

A normal `restore` action will **not fail** in the following scenarios:
* Change in sources - unless package not found as mentioned in the lock file.

### VS restore force option
![image](https://user-images.githubusercontent.com/14800916/38281497-ad663c16-375f-11e8-8dbb-7ebafbcb20c5.png)

# Out of scope

This spec is only to solve the [repeatable build problems](#problem) through NuGet generated lock file. This does not solve the following requirements from the [[PackageReference enhancements]] document:

| # | Requirement | Issue | 
|:--- |:-----------|:--------|
| R2 | Developers would like to control the packages and their versions that are allowed to be used across the team/product | [6464](https://github.com/NuGet/Home/issues/6764) |
| R3 | Developers would like to control dependency resolution behaviors |  [#5553](https://github.com/nuget/home/issues/5553) [#912](https://github.com/NuGet/Home/issues/912) |


The following are also out-of-scope:
### Locking the dependency graph at a solution or repo level
IMO, this should be solved as part of managing/controling packages at a solution/repo level as called out in the requirement **R2** above. This means that different projects in a solution may end up with different versions of a package depending on the version as mentioned in the `PackageReference` node in the project file. This feature does not intend to solve the consolidation of package versions across projects in a solution or across solutions in a repo.
