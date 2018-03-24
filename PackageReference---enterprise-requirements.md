Status: **Incubation**

[Context and Key Terms](#context-and-key-terms)

# Requirements

All related specs/issues at a glance: 

| # | Requirement | Issue # | 
|:--- |:-------------|:--------|
| R1 | Developers would like to have repeatable builds (restores) across time and space | **[#5602](https://github.com/nuget/home/issues/5602)** |
| R2 | Developers would like to be aware of any unintended changes to their package dependency closure including trasitive ones | **[#5602](https://github.com/nuget/home/issues/5602)** |
| R3 | Developers would like to control dependency resolution behaviors |  [#5553](https://github.com/nuget/home/issues/5553) [#5553](https://github.com/NuGet/Home/issues/5553) |
| R4 | Developers would like to control the packages and their versions that are allowed to be used across the team/product | TBD |
	
# Who is the customer?

Primarily developers in an **enterprise** using `PackageReference` with **huge code base**. 
	
# Problem

### R1 - Developers would like to have repeatable builds (restores) across time and space

| PH# | Problem Hypothesis |
|:--- |:---------------|
| PH1 | Developers do not have confidence that NuGet will restore to the same full closure of package dependencies when they build it on Dev machine vs. CI/CD machines |

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

### R2 - When developers add/remove a package dependency to a project, they would like to be told about any unintended changes in the full package dependency closure.

Consider the following scenario:
 
`Project1 --> PackageA(1.0.0) --> PackageB(>=2.0.0)` 

Now lets say you bring in another dependency on PackageX(3.0.0) with some transitive dependencies. And suddenly you build breaks because the PackageX(3.0.0) had the following transitive dependencies:

```
Project1--> PackageA(1.0.0) --> PackageB(>=2.0.0)
       |--> PackageX(3.0.0) --> PackageY(3.0.0) --> PackageZ (1.0.0) --> PackageB(4.0.0)
```

So now instead of PackageB(2.0.0), NuGet resolves to PackageB(**4.0.0**) that may have breaking changes. Obviously this is due to an intentional package install but the transitive closure happens behind the scenes without letting the users know the changes in transitive dependency versions. Sometimes this is not ideal. Users would like to know the difference in package dependency graphs irrespective of whether the change is related to direct or indirect/transitive dependencies.

### R3 - Developers would like to define dependency resolution behavior to control what suits their scenarios

| PH# | Problem Hypothesis |
|:--- |:---------------|
| PH2 | Developers cannot control the behavior of transitive dependency resolution i.e. if they want to always float to the latest patch version of all the transitive dependencies to help them take the non-breaking updates of such packages |
| PH3 | Developers cannot control the behavior of transitive dependency resolution for specific top-level packages | 
| PH4 | Developers cannot float to a pre-release version of a package using floating versions |
| PH5 | Developers cannot float to specific (rc/beta/alpha) pre-release version of a package using floating versions |

With packages.config, developers had the capability to define the version resolution strategy for transitive dependencies of a package being installed:
![image](https://user-images.githubusercontent.com/14800916/37796161-d72c9bb6-2dd3-11e8-809e-8a06967ec089.png)

This is required many a times as developers would like to always the the patched version of all their dependencies. This is a missing functionality with PackageReference. 

For direct/top level dependencies, with floating versions, today NuGet always resolves to the highest **stable** versions. There is no way to allow to float to pre-release versions. 

### R4 - Developers would like to specify control the packages and their versions that are allowed to be used in their projects or solutions across the team/product

| PH# | Problem Hypothesis |
|:--- |:---------------|
| PH6 | Developers cannot define an allowed list of packages that can be used in an application development across projects/solutions/repos |
| PH7 | Developers cannot restrict to use the same version of a given package across projects/solutions/repos |
| PH8 | Developers find it difficult to use a predetermined allowed version range of given package across projects/solutions/repos |

With huge code bases, complex project structures with large set of packages to deal with, it becomes really hard for developers to keep consistency on the package and versions used across the projects/solutions. Developers need a way to control the packages they would like to use downstream from a repository level or a solution level. They would like to ensure that only specific versions can be used throughout their code base.

# Solution

With all the requirements
1. Ability to define packages dependencies and versions at solution level
1. Ability to define allowed packages dependencies and versions at any level
1. Ability to define transitive package dependency resolution strategy 
1. [Visual Studio] Ability to manage package dependencies and version at solution level  

## Define packages dependencies+versions at solution level

Summary:

### Define a list of allowed package dependencies 
I can define a list of **allowed** package dependencies in the nuget.config file at **solution root** folder by setting the `ManagedPackageDependencies` mode ON.
* It is recommended to define only the top-level dependencies used in projects. 
* Transitive dependencies could be listed too. If NuGet finds a package listed here, it will use the version and properties as listed without trying to resolve.
* In the above case, if the versions required as part of NuGet's transitive dependency resolution does not map to the allowed version listed in nuget.config, NuGet will throw an error. 

Sample nuget.config file:
```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    
    <config>
        <add key="managedPackageDependencies" value="true" />
    </config>

    <packageSources>
        <add key="NuGet.org" value="https://api.nuget.org/v3/index.json" />
        <add key="VSTSFabrikam" value="https://nugetplay.pkgs.visualstudio.com/_packaging/Fabrikam/nuget/v3/index.json" />
        <add key="MyInternalFeed" value="https://myinternalfeed/feed/index.json" />
    </packageSources>

    <packageDependencies>
        <id="Newtonsoft.Json" version="10.0.2" source="NuGet.org"/>
        <id="My.Sample.Lib" version="[3.4.0, 4.0.0)" source="VSTSFabrikam"/>
        <id="My.Floating.Lib" version="3.*" source="MyInternalFeed" includePrerelease="rc;beta"/>
    </packageDependencies>

</configuration>
```

#### Why nuget.config?
Defining the allowed package dependencies in the nuget.config has the following benefits:
* No additional moving part - We do not want to introduce yet another file used only for NuGet. 
* Define package sources and dependencies together - nuget.config file already defines the package sources. It makes it easier to view and define both the package sources and the dependencies together
* Hierarchical evaluation - nuget.config settings are evaluated in hierarchical manner that enables one to use the allowed package dependencies with at the user level or solution level or project level.

### Add package dependency to project that's allowed
I can add only those package dependencies to my project that are already listed as the allowed package dependencies in the \<mysolution>\nuget.config file.

```
> dotnet add package newtonsoft.json -v 11.0.1
Could not find newtonsoft.json version 11.0.1 in the list of package dependencies defined in nuget.config. 
Please relax the allowed versions specified in the nuget.config or installed an allowed version: Newtonsoft.Json 10.0.2
You can also try installing the package without specifying any version.

> dotnet add package newtonsoft.json
Using the managed NuGet dependencies mode - resolving dependencies from the nuget.cofig file.
Successfully installed Newtonsoft.Json version 10.0.2

> dotnet add package Package.NotListed 
Could not find Package.NotListed in the list of package dependencies defined in nuget.config. 
Please add this package as a allowed package dependency in nuget.config. You may also use the 
following command to install this package and add it as an allowed package dependency to the 
nuget.consig file as well:
    dotnet add package Package.NotListed --add-as-allowed-package-dependency
```

### NuGet generated lock file
When I install/update a package dependency to any of my projects, it modifies the nuget generated lock file - nuget.dependencies.lock file.

#### What is install/update in nuget?
When you add a dependency in one of the following ways to a project, it's an install:
* In Visual Studio, you go to PMC/PMUI to find a package and press 'Install'
* `dotnet add package <Package>`
* Directly edit the project file to include a `PackageReference` node. E.g. `<PackageReference Include="newtonsoft.json" Version="10.0.2" />`

In all of the above scenarios, with PackageReference, install action just finds out the compat and adds the `PackageReference` node to the project file. To complete the install process i.e. to bring in the actual package on disk, a restore operation is run. 

Here I will be using the term 'install' to mean the user action of installing (including getting the package on disk using nuget restore action).

### When is the lock file modified?
* Install - When a package is installed to a project that uses allowed packages/versions (from nuget.config file), a `package.dependencies.lock` file is generated if it does not exist already. If the file exists, its modified. 
* Update - the lock file will be modified when package dependencies are updated in the nuget.config file.

### When is the lock file NOT modified?

* A normal restore action (not the one accompanied with install/update) will not modify the package.dependencies.lock file. 
* A restore operation will use the package.dependencies.lock file to get the full package dependencies closure if there were no changes to the package dependencies in nuget.config file or to the individual projects.

Since restore is the common action run when a install/update is done with NuGet, in order to prevent any accidental modification of lock file, there will be an option to fail restore if it requires modification of the package.dependencies.lock file.

**Scenario:** 
* On DEV machine, a developer adds a new package to a project which causes an update to the lock file. Now while checking in the project, he/she disregards changes to the package.dependencies.lock file.  
* On CI/CD machine, when restore runs, it finds out the additional PackageReference in the project file and hence updates the package.dependencies.lock file.
This behavior goes against the repeatbale build expectation, though due to a mistake. To prevent this, on CI/CD machines, once should always call restore with the following option:

```
> dotnet restore --fail-on-lock-file-modify
```

The same can be achieved by setting the environment variable **NUGET_RESTORE_FAIL_LOCK_MODIFY" to `true`.

### package.dependenies.lock propoerties

1. Generated at the solution level
1. Uses the allowed versions to compute the Contains allowed version of the package
2. Contains resolved version of the package
3. Contains whether a dependency is a direct of transitive dependency
4. Contains SHA512 hash of the package resolved

Proposed format:
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

4. With  

## Ability to define allowed packages dependencies and versions at any level
## Ability to define transitive package dependency resolution strategy  
## [Visual Studio] Ability to manage package dependencies and version at solution level  

# Context and Key Terms

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
