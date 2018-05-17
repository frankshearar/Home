* Status: Incubation
* Author(s): [Anand Gaurav](https://github.com/anangaur) ([@adgrv](https://twitter.com/adgrv))

### Requirements
Refer to the spec:[[Centrally managing NuGet packages]] for the list of requirements and summary of the solution. This spec details out the solution for managing package versions, centrally at a solution (or repo) level.

*For lock file details, refer to the spec: [[Enable repeatable package restore using lock file]]*

### Solution Details

To get started, you will need to create an MSBuild props file at the root of the solution/repo named `packages.props` that declares `Package` items.

In this example, packages like `Newtonsoft.Json` are set to version `10.0.1`.  All projects that reference this package will be locked to that version. The `PackageReference` in the projects would not specify the version information.

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
Each project still has a `PackageReference` but must not specify a version.  This ensures that the correct packages are referenced for each project.


**Implicit package**: Listing **Implicit packages** in the central packages version management (CPMVF) file is optional. If this is not provided, the corresponding lock files will not list implicit dependencies. If listed in the CPVMF, however, the same gets persisted as a locked dependency. 

**Transitive dependencies**: One should not be listing the transitive dependencies either in the CPVMF or as `PackageReferece` for the projects. If listed in the CPVMF, the same version will be used in full package closure during restore and the same will be locked. 

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

### Extensibility

| Property                            | Description |
|-------------------------------------|-------------|
| `CentralPackagesFile `  | The full path to the file containing your package versions.  Defaults to `packages.props` at the root of your repository. |
| `LockFile` | The full path of the lock file applicable for a project. Defaults to `packages.lock.json` in the same folder as the `packages.prop`. If specified as `none`, no lock file will be created |

*Examples*

Use a custom file name for your project that defines package versions.
```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <CentralPackagesFile>$(MSBuildThisFileDirectory)MyPackageVersions.props</CentralPackagesFile>
    <LockFile>$(MSBuildThisFileDirectory)MyPackagesLock.lock.json</CentralPackagesFile>
  </PropertyGroup>
</Project>
```

### Bootstrapping

#### How to enable this the central package version management + locking? 
The functionality will be enabled if
* There is `packages.props` file present in the recursive path for a project.
* If the `CentralPackagesFile` MSBuild property exists for a project. 
* Lock file feature is tied to the presence of a central package version management file. It can be disabled using a property as mentioned in the previous section.

#### How do I transform my existing projects to use this functionality?
Existing projects will have versions in their project files. 

*Generate a starting `packages.props` file*

Run the following command on the root of the repo that contains all the projects that you want to manage using the central package management file

```
// create a central package management file for all projects under the current directory
> dotnet restore --generate-version-management-file
<Restore output>
.
.
Created `packages.props` file that you can use to manage the package versions centrally at a repo or solution level. Learn more at https://aka.ms/nuget-centrally-manage-pkg-versions

// Check for issues in pruneing PackageReference  to remove the version attributes from projects under the current directory
> dotnet nuget prune packagereferences --dry-run

// Prune PackageReference  to remove the version attributes from projects under the current directory
> dotnet nuget prune packagereferences
```

#### How is lock file generated? When?
Lock file is generated if CPVMF is present (and lock file is not disabled) using any command that modifies the CPVMF eg. install/update of packages or restore with `--update-lock-file` option. Lock file is always generated for the packages mentioned in the CPVMF.

#### How does `restore` work?
A normal restore on a project just looks at the lock file for the full closure and generates the assets file (`obj/project.assets.json`) for the project based on the `PackageReference`s in the project. It does not recompute the dependencies or tries to get get the full closure every time. For a project with other project references, it gets all the `PackageReference`s across the full project reference closure and generates the assets file based on the closure in the lock file.

#### Where are `PrivateAssets`/`ExcludeAssets`/`IncludeAssets` defined?
These can be defined in the CPVMF but overridden in the project file. 

#### How does restore NoOp work i.e. when does NuGet try to actually restore or choose not to restore?
The current logic is used except that the package versions are referenced from the CPVMF. In future, we can look to optimize the NoOp restore but is `Out of Scope` for this spec.

#### I consume the same project in different solutions. How do I want to use the central package version management file in one and not the other?
This will require the `PackageReference` nodes to have the version info but ignore it in the solution where central package version management file is used. This may be achieved by a MSBuild property `RetainPackageReferenceVersions`

**Not MVP** scenario.

### VS experience
TBD.

## FAQs

### How do I have a given set of package versions for all the projects but a different set for a specific package?
To override the global packages' version constraints for a specific project, you can define `packages.props` file in the project root directory. This will override the global settings from the global/repo-level `packages.props` file. For this case, the lock file `packages.lock.json` will be generated at the project root directory.

You can also specify `CentralPackagesFile` property indicating where to look for this file for a given project in the project file or in the `directory.build.props` file at the project root directory that gets evaluated for a given project.

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
