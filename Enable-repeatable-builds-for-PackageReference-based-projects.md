Status: **Incubation**

# Issue
The work for this feature and the discussion around the spec is tracked here - **Enable repeatable builds for PackageReference based projects [#5602](https://github.com/NuGet/Home/issues/5602)**

All related specs/issues at a glance: 

| Title | Issue | Spec |
|:------------- |:-------------:| -----:|
| **Enable repeatable builds for PackageReference based projects (via lock file)**  | **[#5602](https://github.com/nuget/home/issues/5602)** | **[Incubation](https://github.com/NuGet/Home/wiki/Enable-repeatable-builds-for-PackageReference-based-projects)** |
| Manage allowed packages for a solution (or globally)  | TBD |  [Incubation](https://github.com/NuGet/Home/wiki/Manage-allowed-packages-for-a-solution-%28or-globally%29) |
| Allow users to determine package resolution strategy during package restore - direct or transitive | [#5553](https://github.com/nuget/home/issues/5553) | TBD |


# Context

Projects that use PackageReference to manage NuGet dependencies, only provide direct package dependencies. The transitive closure for the dependencies happen at the restore time.  
Refer to the dependency resolution algorithm for NuGet. Overall here is the summary of dependency resolution:

## Direct dependency resolution:
1. If exact version is specified - NuGet tries to resolve to the exact version. If not, it resolves to next highest version i.e. the lowest version equal to or near to the version specified. 
E.g. 
	
   `<PackageReference Include="My.Sample.Lib" Version="4.5.0" />`

   a. NuGet resolves to version 4.5.0 if present in the feed. 

   b. If Feed has only these versions: 4.0.0, 4.6.0, 5.0.0 then NuGet resolves to 4.6.0 

2. If a range is specified - NuGet resolves to the lowest version specified in that range or that satisfies the floating expression.
E.g.

   Feed has only these versions for My.Sample.Lib: 4.0.0, 4.6.0, 5.0.0

   a. Range is specified:
		
   `<PackageReference Include="My.Sample.Lib" Version="[4.0.0, 5.0.0]"/>`
		
      NuGet resolves to the 4.0.0 here. 

   b. Range is specified contd..
		
   `<PackageReference Include="My.Sample.Lib" Version="[4.1.0, 5.0.0]"/>`
		
      NuGet resolves to the 4.6.0 here.
	
3. If a floating version is specified is specified - NuGet resolves to the highest version that satisfies the floating expression.
E.g.	
   
   Floating version is specified:

   (Feed has only these versions for My.Sample.Lib: 4.0.0, 4.6.0, 5.0.0)

   `<PackageReference Include="My.Sample.Lib" Version="4.*"/>`
		
    NuGet resolves to 4.6.0 here.

    NuGet resolves to next-highest-version-available* on the feed if there are no versions matching the floating expression. i.e. if 4.0.0 and 4.6.0 were not present on the feed, NuGet would have resolved to 5.0.0 even though the floating expression says 4.*. This, IMO, is a bug: https://github.com/NuGet/Home/issues/5097
		
## Transitive dependency resolution
In case of transitive dependencies, the resolution is always to the lowest version specified in the dependency version or version ranges as specified here.

There are additional mechanisms to resolve conflict in dependency versions and those are resolved through "Nearest wins" and "Cousin dependencies" algorithm as discussed in details in the documentation.
		
# Problem
Users want their builds to be repeatable if the code doesn't change irrespective of when and where the build happens.
Refer to the context on NuGet dependency resolution. Because of the various ways by which dependency resolution happen, for a given project, the restore can bring in different versions of the package dependencies when run at different times and different places even when there is no change in package dependencies specified in the project file. This can be due to the following factors:
1. External: When package publishers change (add/remove) the packages on the feed(s).
2. Internal: When the nuget.config (if not checked-in) points to different feeds across builds.

This feature aspires to solve this issue and enable repeatable builds.

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
This will be an intrinsic feature for NuGet and hence the lock file should be generated by default for all NuGet restores. However, the feature would be behind a feature flag during it's preview. This will enable better dogfooding of the feature and help us make changes before we make this a public feature. To try this **preview** feature, user needs would need to set the following property in NuGet.config
```
<EnableRestoreLockFile>True</EnableRestoreLockFile>`
```

### What happens when this property `EnableRestoreLockFile` is set to false (not set) but lock file is present in the project's root folder?
In this case, the lock file should be disregarded with a warning message:
```
NUxxxx: The nuget.dependencies.lock file for this project file is being disregarded because EnableRestoreLockFile is not set (or set to false). Consider either deleting the nuget.dependencies.lock or setting EnableRestoreLockFile to fix this warning.
```

## Lock file properties

Assuming the `EnableRestoreLockFile` is set to `true`, following are the properties of lock file:
* The lock file is called `nuget.dependencies.lock` 
* The lock is created in the project **root** directory by default. 
* We recommend this file should be checked in to the source repository.
* The file contents should be in order so that the changes to this file can be readable by a user and diff can be easily understood during commits. The lock file should contain all the dependencies in order (either all in alphabetical order or direct first and followed by transitive in alphabetical order) and each dependency should list its dependencies.
* [**Not-MVP**] Users can override the path of the lock file using the following options:
  * In NuGet.config file (this will be a relative path):
    ```
    <RestoreLockFilePath>../LockFiles/$(MSBuildProjectName).nuget.dependencies.lock</RestoreLockFilePath>
    ```
    In this case all the lock files for the projects get created and used from the LockFile folder in the solution folder
  * In the project file:
    ```
    <RestoreLockFilePath>$(MSBuildProjectName).nuget.dependencies.lock</RestoreLockFilePath>
    ```

### When is lock file created/re-generated?

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

