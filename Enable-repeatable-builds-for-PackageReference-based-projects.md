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
This feature will be behind a setting. This will let us roll out the feature without impacting developers who do not expect to see an additional file. 

If a lock file is present in a project, NuGet will use this lock file for `restore`.

To bootstrap the creation of lock file, you can do one of the following:

1. Specify the following setting in nuget.config:

```
<config>            
     <add key="RestoreWithLockFile" value="true">
</config>
```
2. On VS, set the "Restore with lock file" option in nuget settings:
![image](https://user-images.githubusercontent.com/14800916/38658987-2bec7168-3ddc-11e8-8657-d42d405453f0.png)

3. Set the following MSBuild property for the project:
E.g. In the project file:
```
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp2.0</TargetFramework>
    <RestoreWithLockFile>true</RestoreWithLockFile>
  </PropertyGroup>
```

If this property is set using any of the 3 mechanism specified above, NuGet `restore` will create the lock file with the following message (in default verbosity):
```
RestoreWithLockFile property is set but no lock file found - generating lock file nuget.packages.lock. 
Subsequently this lock file will be used to restore packages and restore will not modify the lock file. Learn more: https://aka.ms/nuget-lock-file
```
 
### What happens when this property `RestoreWithLockFile` is not set but lock file is present in the project's root folder?

NuGet will use the lock file if present. The setting `RestoreWithLockFile` is only to bootstrap the creation of this lock file.

### Which commands can modify the lock file? 

Following is a list of commands/actions with information whether they can modify the lock file or not:
*If the lock file is not present, any command that can modify the lock file will generate/create it (if `RestoreWithLockFile` is set).*

| Tool+command/action | Can modify/generate lock file? |
|:--- |:---|
| VS PM UI `Install` action | Yes |
| VS PMC `Install-Package` command | Yes |
| `dotnet add package` | Yes |
| `dotnet add reference` | Yes |
| **`dotnet update package`** * | Yes |
| `dotnet restore` | **No** |
| `msbuild /t:restore` | **No** |
| `msbuild /restore` | **No** |
| `dotnet restore -recompute-dependencies` | **Yes** |
| VS `rebuild` action | **No** |
| VS `clean` + `build` action | **No** |
 
* New commands

## Lock file properties

* The lock file will be called `packages.lock,json` 
* The lock is created in the project **root** directory by default. 
* The lock file name/path can be overridden with the following MSBuild property:
  - If you want to name the lock file based on the project name:
    ```<NuGetLockFilePath>$(MSBuildProjectName).packages.lock.json</NuGetLockFilePath>```
  - If you want to put all the lock files together in some `LockFiles` directory:
    ```<NuGetLockFilePath>../LockFiles/$(MSBuildProjectName).packages.lock.json</NuGetLockFilePath>```
* We recommend this file should be checked in to the source repository.

### Lock file format
* The lock file format should be such that it is:
  	- Concise
	- Human readable
	- Easy to diff
	- Performant
	- machine diff-able/merge-able
* The file contents should be in order so that the changes to this file can be readable by a user and diff can be easily understood during commits. The lock file should contain all the dependencies in order:
  * All the direct dependencies in case insensitive alphabetical order first 
  * All the transitive dependencices next, in case insensitive alphabetical order
  * Each package entry in the lock file should list its direct dependencies.
* The lock file needs to contain all the information needed to make the decision to modify/re-generate the lock file based on the requirements mentioned in the above section.
* The lock file should contain SHA512 hash for each package dependency. Upon restore, NuGet will scan through the packages and and SHA values in the lock file and error out if they do not match.
* The integrity field is purposefully made extensible so that any integrity check can be plugged in later, if required.

*Sample lock file*
```	
# THIS IS AN AUTO-GENERATED FILE. DO NOT EDIT THIS FILE DIRECTLY.	
# YOU SHOULD CHECK THIS FILE INTO YOUR SOURCE REPOSITORY.	
# LEARN MORE - https://aka.ms/nuget-lockfile	
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

## Restore scenarios

A normal `restore` action will fail in the following scenarios:
* Manually Addition of a new PackageReference node
* Manual update of PackageReference(s) 
* Addition of a Project reference - may fail
* Target framework changes

In all the above cases, user can run one of the following to update the lock file:

| Action/Command | Result |
|:--- |:---|
|  VS `install`, `dotnet add package <packageID>`,  **`dotnet install package <packageID>`** *  | Installs the package. Modifies the lock file.  |
| **`dotnet install package`** * without a packageID on the project root folder | Installs any additional package required. Modifies the lock file.  |
| VS `update`, **`dotnet update package <packageID>`** * | Updates the package.  Modifies the lock file. Does not update the floating version |
| VS `recompute dependencies`, `dotnet restore --recompute-dependencies`, **`msbuild /t:restore /p:recompute-dependencies`**,  **`msbuild /restore /p:recompute-dependencies`** | Re-computes the package dependency graph. Updates the floating package version in the lock file to the latest available. Modifies the lock file if required. |

A normal `restore` action will **not fail** in the following scenarios:
* Change in sources - unless package not found as mentioned in the lock file.

### Restore option: `Recompute package dependencies`

A normal restore will fail or will be ineffective in the following scenarios:
* User manually adds/modifies a `PackageReference` to the project file
* User wants to restore to the latest version of a package specified with floating range \*

For all the above cases, user would need to use an additional parameter with restore:

### CLI
```
> dotnet restore --recompute
```
```
> nuget restore -recompute
```
```
> msbuild /t:restore --recompute
```
*Visual Studio*
![image](https://user-images.githubusercontent.com/14800916/38649066-6ce2f56c-3da9-11e8-9e62-f669d4ce9dda.png)

### Exception
The idea of lock file is to enable repeatable build across space and time. 

### `restore` command options, at a glance

| Option | Restore behavior |
|:---- |:--- |
| `--recompute` | Explicit command to overwrite the lock file. Recomputes the package dependency graph. |
| `--ignore-lock-file` | Restore without the lock file. Lock file is neither considered for restore nor touched. |
| `--lock-file=<lock file path>` | Uses the lock file provided as part of the command option. Overrides any other setting for lock file |
| `--strict-lock` | Always fails if the package resolution is different than what is present in the lock file | 
| `--update-lock-file` | Updates lock file as part of restore if required. This is **not** same as `--recompute` option. `--recompute` always forces re-computation of the package graph. This option, however, does not do so, if not required. But it will also not error out if the lock file is updated. Eg. `restore --update-lock-file` may not fetch the latest version for a floating package (due to NoOp). |

# Future backlog
This spec is only to solve the [repeatable build problems](#problem) through NuGet generated lock file. This does not solve the following requirements from the [[PackageReference enhancements]] document. These features will have their own specs published, in near future:

| # | Requirement | Issue | 
|:--- |:-----------|:--------|
| R2 | Developers would like to control the packages and their versions that are allowed to be used across the team/product | [6464](https://github.com/NuGet/Home/issues/6764) |
| R3 | Developers would like to control dependency resolution behaviors |  [#5553](https://github.com/nuget/home/issues/5553) [#912](https://github.com/NuGet/Home/issues/912) |

