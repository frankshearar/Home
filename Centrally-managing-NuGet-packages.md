**Status**: Incubation, **PM**: [Anand Gaurav](https://github.com/anangaur)

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
Enterprise customers with huge code-base spanning 100s of projects. 

## Evidence
* Developers need this and have worked their way around in multiple ways. Some define the package version as a variable and then define all the version variables in a central file to control this behavior. E.g.
  ```
  <PackageReference Include="My.Sample.Lib" Version="$(MySampleLibVersion)" />
  ```
* MSBuild team has built an SDK project to implement this behavior: [Microsoft.Build.CentralPackageVersions](https://github.com/Microsoft/MSBuildSdks/tree/master/src/CentralPackageVersions#microsoftbuildcentralpackageversions). This spec has taken inspiration from this.
 
## Solution

### Summary
* Package Versions can be centrally managed in `packages.props` file located at the repo root.
* Package references (`PackageReference`) managed at each project level without any version information.
* Managing packages for the repo/projects:
  * Adding packages references not listed in `packages.props` will be an error by default. An option to update `packages.props` file as part of adding the package reference will be available.
  * Updating package reference per project will be an error. An option to update in the `packages.props` file will be available.
  * Removing/uninstalling package references per project is allowed. There will be an option to do the same in the `packages.props` file.
* All the referenced packages and their transitive dependencies are locked in a lock file - `packages.lock.json` present at the level (same folder) as the `packages.props` file.
  * NuGet `restore` action, by default, will always resolve the dependencies using the lock file.
    * It will check if a referenced package (`PackageReference` in the project file) is already present in the `packages.props` file as well as the `packages.lock,json` file. If not, it errors out.
    * It will check if the version specified in `packages.props` match with `packages.lock.json`. If not, it errors out.
    * The lock file contains the integrity data (SHA) for each of the packages listed in it. Restore does a post step to validate the integrity of the packages. It errors out if the integrity check fails. This option to include integrity will be configurable using a MSBuild property.
  * NuGet `restore -update-lock-file` action will be able to recompute dependencies and overwrite the lock file. A similar experience on VS will be available.

### Details - Centrally managing package versions

To get started, you will need to create an MSBuild props file at the root of the solution/repo named `packages.props` that declares `Package` items.

In this example, packages like `Newtonsoft.Json` are set to exactly version `10.0.1`.  All projects that reference this package will be locked to that version.  If someone attempts to specify a version in a project they will encounter a build error.

*Packages.props*
```xml
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <ItemGroup>
    <!-- Implicit Package References -->
    <Package Include="Microsoft.NETCore.App"    Version="[2.0.5]" />
    <Package Include="NETStandard.Library"      Version="[1.6.1]" />

    <Package Include="Microsoft.NET.Test.Sdk"   Version="[15.5.0]" />
    <Package Include="MSTest.TestAdapter"       Version="[1.1.18]" />
    <Package Include="MSTest.TestFramework"     Version="[1.1.18]" />
    <Package Include="Newtonsoft.Json"          Version="[10.0.1]" />
  </ItemGroup>
</Project>
```

*SampleProject.csproj*
```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" />
  </ItemGroup>
</Project>
```
Each project still has a `PackageReference` but must not specify a version.  This ensures that the correct packages are referenced for each project.

Listing **Implicit packages** in the central ... 

*Enforcement*

```csharp
// package exists in packages.props
ProjectA> dotnet add package netwonsoft.json 
Successfully added package 'Newtonsoft.Json' to ProjectA. 

// package does not exist in packages.props
ProjectA> dotnet add package netwonsoft.json 
NU1xxx: Cannot add package 'Newtonsoft.Json' to ProjectA as it is not specified in the central package version management 
 file '<path>\packages.props'. Use the command 'dotnet add package newtonsoft.json --version <version number> --update-version-management-file' to add the package reference to the project and update the central package version management file '<path>\packages.props', together.

ProjectA> dotnet add package newtonsoft.json --version 10.0.3 --update-version-management-file
Successfully added package 'Newtonsoft.Json version 10.0.3' '<path>\packages.props'.
Successfully added package 'Newtonsoft.Json' to ProjectA. 

ProjectA> dotnet add package netwonsoft.json --version 11.0.1
NU1xxx: You cannot specify the package version while adding a package reference. Package version version information is managed in the central package version management file '<path>\packages.props'. Use the command 'dotnet add package newtonsoft.json --version <version number> --update-version-management-file' to update the package version in the central package version management file '<path>\packages.props'.

//Removing a package reference in a project
ProjectA> dotnet remove package netwonsoft.json
Successfully removed package 'Newtonsoft.Json' from ProjectA. 

//Removing a package reference and the version info from packages.props
ProjectA> dotnet remove package netwonsoft.json --update-version-management-file
Successfully removed package 'Newtonsoft.Json' from ProjectA. 
Successfully removed package version information from the central package version management file '<path>\packages.props'.

//Removing a package version info from packages.props when the package reference does not exist
ProjectA> dotnet remove package netwonsoft.json --update-version-management-file
Package 'Newtonsoft.Json' is not referenced in ProjectA. 
Successfully removed package version information from the central package version management file '<path>\packages.props'.
```
#### Global Package References
Some packages may be referenced by all projects in your tree. This includes packages that do versioning, extend your build, or do any other function that is needed repository-wide. 

*Packages.props*
```xml
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <ItemGroup>
    <Package Include="Nerdbank.GitVersioning" Version="[2.1.16]" PrivateAssets="All" />

    <PackageReference Include="Nerdbank.GitVersioning" Condition=" '$(EnableGitVersioning)' != 'false' " />
  </ItemGroup>
</Project>
```
`Nerdbank.GitVersioning` will be a package reference for all projects.  A property `EnableGitVersioning` has been added for individual projects to disable the reference if necessary.

### Details: Locking package dependencies
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

### Extensibility

| Property                            | Description |
|-------------------------------------|-------------|
| `CentralPackagesFile `  | The full path to the file containing your package versions.  Defaults to `Packages.props` at the root of your repository. |
| `` | |

*Example*

Use a custom file name for your project that defines package versions.
```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <CentralPackagesFile>$(MSBuildThisFileDirectory)MyPackageVersions.props</CentralPackagesFile>
  </PropertyGroup>
</Project>
```

### Making it work for existing projects
Existing projects will have versions in their project files. There are two approaches for these projects to make use of the `CentralPackagesFile`:

#### Enable users to update the `PackageReferences`

TBD.

Additionally generate the `CentralPackagesFile` to get started.

TBD.


```
>
```


#### MSBuild option to ignore the versions mentioned in `PackageReference` nodes 
TBD.

### VS experience
TBD.

## FAQs

### How to enable this feature?
To enable just put a `packages.props` file in the solution root or repo root directory that is evaluated by `msbuild.exe` during build. Additionally you can specify `CentralPackagesFile` property indicating where to look for this file. The same will enable the feature.

### How do I have a given set of package versions for all the projects but a different set for a specific package?
To override the global packages' version constraints for a specific project, you can define `packages.props` file in the project root directory. This will override the global settings from the global/repo-level `packages.props` file.

You can also specify `CentralPackagesFile` property indicating where to look for this file for a given project in the project file or in the `directory.build.props` file that gets evaluated for a given project.

### What happens when there are multiple `packages.props` file available in a project's context?
In order to remove any confusion, the `packages.props` or the `CentralPackagesFile` specification nearest to the project will override all others. At a time only one `packages.props` file is evaluated for a given project.

E.g. in the below scenario

```
Repo
 |-- packages.props
 |-- foobar.packages.props
 |-- Solution1
     |-- packages.props
     |-- Project1
     |-- Project2
         |-- packages.props
     |-- Project3
         |-- directory.build.props   // specifies CentralPackagesFile = path to foobar.packages.props
 |-- Solution2
     |-- Project4
```

In the above scenario:
* Project1 will refer to only `Repo\Solution1\packages.props`
* Project2 will refer to only `Repo\Solution1\Project2\packages.props`
* Project3 will refer to only `Repo\foobar.packages.props`
* Project4 will refer to only `Repo\packages.props`



