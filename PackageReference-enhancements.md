Reference document

[Context and Key Terms](#context-and-key-terms)

# Requirements

All related specs/issues at a glance: 

| # | Requirement | Issue # | 
|:--- |:-----------|:--------|
| R1 | Developers would like to have repeatable builds (restores) across time and space | [#5602](https://github.com/nuget/home/issues/5602) |
| R2 | Developers would like to control the packages and their versions that are allowed to be used across the team/product | [6464](https://github.com/NuGet/Home/issues/6764) |
| R3 | Developers would like to control dependency resolution behaviors |  [#5553](https://github.com/nuget/home/issues/5553) [#912](https://github.com/NuGet/Home/issues/912) |
	
# Who is the customer?

Primarily developers in an **enterprise** using `PackageReference` with **huge code base**. 
	
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

### R2 - Developers would like to define dependency resolution behavior to control what suits their scenarios

| # | Problem statement |
|:--- |:---------------|
| PRS3 | Developers cannot control the behavior of transitive dependency resolution i.e. if they want to always float to the latest patch version of all the transitive dependencies to help them take the non-breaking updates of such packages |
| PRS4 | Developers cannot control the behavior of transitive dependency resolution for specific top-level packages | 
| PRS5 | Developers cannot float to a pre-release version of a package using floating versions |
| PRS6 | Developers cannot float to specific (rc/beta/alpha) pre-release version of a package using floating versions |

With packages.config, developers had the capability to define the version resolution strategy for transitive dependencies of a package being installed:
![image](https://user-images.githubusercontent.com/14800916/37796161-d72c9bb6-2dd3-11e8-809e-8a06967ec089.png)

This is required many a times as developers would like to always the the patched version of all their dependencies. This is a missing functionality with PackageReference. 

For direct/top level dependencies, with floating versions, today NuGet always resolves to the highest **stable** versions. There is no way to allow to float to pre-release versions. 

### R3 - Developers would like to specify control the packages and their versions that are allowed to be used in their projects or solutions across the team/product

| PH# | Problem Hypothesis |
|:--- |:---------------|
| PRS7 | Developers cannot define an allowed list of packages that can be used in an application development across projects/solutions/repos |
| PRS8 | Developers cannot restrict to use the same version of a given package across projects/solutions/repos |
| PRS9 | Developers find it difficult to use a predetermined allowed version range of given package across projects/solutions/repos |

With huge code bases, complex project structures with large set of packages to deal with, it becomes really hard for developers to keep consistency on the package and versions used across the projects/solutions. Developers need a way to control the packages they would like to use downstream from a repository level or a solution level. They would like to ensure that only specific versions can be used throughout their code base.

# Context and Key Terms

Projects that use PackageReference to manage NuGet dependencies, only provide direct package dependencies. The transitive closure for the dependencies happen at the restore time.  
Refer to the dependency resolution algorithm for NuGet. Overall here is the summary of dependency resolution:

## Dependency resolution

### Direct dependency resolution:

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
		
### Transitive dependency resolution
In case of transitive dependencies, the resolution is always to the lowest version specified in the dependency version or version ranges as specified here.

There are additional mechanisms to resolve conflict in dependency versions and those are resolved through [Nearest wins](https://docs.microsoft.com/en-us/nuget/consume-packages/dependency-resolution#nearest-wins) and [Cousin dependencies](https://docs.microsoft.com/en-us/nuget/consume-packages/dependency-resolution#cousin-dependencies) algorithm as discussed in details in the documentation.

## NuGet Actions

### Install

### Update

### Restore

### Restore force?