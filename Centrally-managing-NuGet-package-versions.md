* Status: Incubation
* Author(s): [Anand Gaurav](https://github.com/anangaur) ([@adgrv](https://twitter.com/adgrv))

### Requirements
Refer to the spec:[[Centrally managing NuGet packages]] for the list of requirements and summary of the solution. This spec details out the solution for managing package versions, centrally at a solution (or repo) level.

*For lock file details, refer to the spec: [[Enable repeatable package restore using lock file]]. This spec does not discuss anything about the lock file*

### Solution Details

To get started, you will need to create an MSBuild props file at the root of the solution/repo named `packages.props` that declares `Package` items.

In this example, packages like `Newtonsoft.Json` are set to version `10.0.1`.  The `PackageReference` in the projects would not specify the version information. All projects that reference this package will refer to version `10.0.1` for `Newtonsoft.json`.

*Packages.props*
```xml
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <ItemGroup>
    <Package Include="MSTest.TestAdapter" Version="1.1.0" />
    <Package Include="MSTest.TestFramework" Version="1.1.18" />
    <Package Include="Newtonsoft.Json" Version="10.0.1" />
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

**Implicit package**: **Implicit packages** are not listed in the central packages version management (CPMVF) file.

**Transitive dependencies**: One should not be listing the transitive dependencies either in the CPVMF or as `PackageReferece` for the projects. If listed in the CPVMF, the same version will be used in full package closure during restore and the same will be locked. 

*Commands*

```bash
// package exists in packages.props
ProjectA> dotnet add package netwonsoft.json 
Successfully added package 'Newtonsoft.Json' to ProjectA. 

// package does not exist in packages.props
ProjectA> dotnet add package netwonsoft.json 
Successfully added package 'Newtonsoft.Json 11.0.1' in '<path>\packages.props'.
Successfully added package 'Newtonsoft.Json' to ProjectA. 

// package exists in packages.props. Trying to update to a new version
ProjectA> dotnet add package netwonsoft.json --version 11.0.1
NU1xxx: You cannot specify the package version while adding a package reference. Package version version information is managed in the central package version management file '<path>\packages.props'. Use the command 'dotnet add package newtonsoft.json --version <version number> --update-version-management-file' to update the package version in the central package version management file '<path>\packages.props'.

ProjectA> dotnet add package newtonsoft.json --version 11.0.1 --update-version-management-file
Successfully update package 'Newtonsoft.Json' version from 10.0.2 to 11.0.1' in '<path>\packages.props'.
Successfully added package 'Newtonsoft.Json' to ProjectA. 

//Removing a package reference in a project - Does not remove from CPVMF
ProjectA> dotnet remove package netwonsoft.json
Successfully removed package 'Newtonsoft.Json' from ProjectA. 

//Removing a package from the solution/Repo - removes from CPVMF as well as from all the projects
SolutionDir> dotnet remove package netwonsoft.json
Successfully removed package 'Newtonsoft.Json' from 'packages.props'. 
Successfully removed package 'Newtonsoft.Json' from ProjectA, ProjectB and ProjectD.

// Consolidate packages in the CPVMF - removing packages that are not referenced in the projects
ProjectA> dotnet nuget consolidate [path to CPVMF or a solution]
Consolidating packages in '<path>\packages.props'...
Added 0 packages
Removed 3 packages that were not referenced in any of the projects.

// Consolidate existing PackageReference with versions in Project files into a `packages.props` file
SolutionDir> dotnet nuget consolidate --dry-run 
No 'packages.props' file found. Running consolidate in dry run mode...
Packages referenced in the projects:
NewtonSoft.Json 11.0.3
   ProjectA references 10.0.2
   ProjectB references 11.0.3
My.Sample.Lib 2.1.0
   ProjectA references 2.1.0
   ProjectC references 1.*
My.Utilities.Package [2.0, 4.1.0]
   ProjectB references [2.0, 3.1.0]
   ProjectD references 4.1.0
.
.
To create the consolidated package version management file packages.props at <path> and to remove the version information from the project files, run 'dotnet nuget consolidate [path to packages.props]'

```

### **[Not MVP]** Extensibility 

| Property                            | Description |
|-------------------------------------|-------------|
| `CentralPackagesFile `  | The full path to the file containing your package versions.  Defaults to `packages.props` at the root of your repository. |

*Examples*

Use a custom file name for your project that defines package versions.
```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <CentralPackagesFile>$(MSBuildThisFileDirectory)MyPackageVersions.props</CentralPackagesFile>
  </PropertyGroup>
</Project>
```

### Bootstrapping

#### How to enable this the central package version management + locking? 
The functionality will be enabled if
* There is `packages.props` file present in the recursive path for a project.
* If the `CentralPackagesFile` MSBuild property exists for a project. 

#### How do I transform my existing projects to use this functionality?
Existing projects will have versions in their project files. You can run the consolidate command to create the CPVMF and remove the version information from the project files.

```
// Consolidate existing PackageReference with versions in Project files into a `packages.props` file
SolutionDir> dotnet nuget consolidate --dry-run 
No 'packages.props' file found. Running consolidate in dry run mode...
Packages referenced in the projects:
NewtonSoft.Json 11.0.3
   ProjectA references 10.0.2
   ProjectB references 11.0.3
My.Sample.Lib 2.1.0
   ProjectA references 2.1.0
   ProjectC references 1.*
My.Utilities.Package [2.0, 4.1.0]
   ProjectB references [2.0, 3.1.0]
   ProjectD references 4.1.0
.
.
To create the consolidated package version management file packages.props at <path> and to remove the version information from the project files, run 'dotnet nuget consolidate [path to packages.props]'
```

#### How does `restore` work?
* Project restore - When a project is restored, it restores as it does today. It looks at the CPVMF to get the package versions. If a package is not listed in the CPVMF but is referenced in the project, it errors out.
* Solution restore - Same as today, except when it finds a more packages in CPVMF than in the project, it prunes/consolidates CPVMF i.e. removes the extra packages stated in CPVMF but not referenced in any of the projects.

#### Where are `PrivateAssets`/`ExcludeAssets`/`IncludeAssets` defined?
These are per project properties and should be defined in the `PackageReference` nodes in the project file.

#### How does restore NoOp work i.e. when does NuGet try to actually restore or choose not to restore?
The current logic is used except that the package versions are referenced from the CPVMF. In future, we can look to optimize the NoOp restore with lock file but is `Out of Scope` for this spec.

#### **[Not MVP]** I consume the same project in different solutions. How do I want to use the central package version management file in one and not the other?
This will require the `PackageReference` nodes to have the version info but ignore it in the solution where central package version management file is used. This may be achieved by a MSBuild property `RetainPackageReferenceVersions`. 

When this property is set,
* `dotnet add package` will add the package version info both in the CPVMF as well as in the project files
* `restore` will just **ignore** the version info in the `PackageReference` nodes in the project files. [Open] Should it warn?
* The `dotnet nuget consolidate` command will put the same version info in the `PackageReference` nodes in all the project files in addition to putting the version info in the CPVMF.

#### How do I have a given set of package versions for all the projects but a different set for a specific project?
To override the global packages' version constraints for a specific project, you can define `packages.props` file in the project root directory. This will override the global settings from the global/repo-level `packages.props` file. *For this case, the lock file `packages.lock.json` will be generated at the project root directory.*

**[Not MVP]** You can also specify `CentralPackagesFile` property indicating where to look for this file for a given project in the project file or in the `directory.build.props` file at the project root directory that gets evaluated for a given project.

#### What happens when there are multiple `packages.props` file available in a project's context?
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

#### Can I specify NuGet sources in the packages.props file?
This is not part of the spec/feature but specifying sources in the packages.props file seems like a good idea.


### VS experience
TBD.