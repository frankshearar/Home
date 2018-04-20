## Issue
| Requirement | Issue |
|:---|:---|
| Developers would like to control the packages and their versions that are allowed to be used across the team/product | [6764](https://github.com/NuGet/Home/issues/6764) |
# Context
[PackageReference requirements summary](https://github.com/NuGet/Home/wiki/PackageReference-enhancements) | Epic issue [#6763](https://github.com/NuGet/Home/issues/6763)

## Who is the customer?
Enterprise customers with huge code-base spanning 100s of projects. 

## Problem
#### Developers would like to specify control the packages and their versions that are allowed to be used in their projects or solutions across the team/product

| # | Problem statement |
|:--- |:---------------|
| PRS7 | Developers cannot define an allowed list of packages that can be used in an application development across projects/solutions/repos |
| PRS8 | Developers cannot restrict to use the same version of a given package across projects/solutions/repos |
| PRS9 | Developers find it difficult to use a predetermined allowed version (or range) of given package across projects/solutions/repos |

With huge code bases, complex project structures with large set of packages to deal with, it becomes really hard for developers to keep consistency on the package and versions used across the projects/solutions. Developers need a way to control the packages they would like to use downstream from a repository level or a solution level. They would like to ensure that only specific versions can be used throughout their code base.

## Solution

### Centrally Managing Package Versions

To get started, you will need to create an MSBuild props file at the root of the solution/repo named `Packages.props` that declares `Package` items.

In this example, packages like `Newtonsoft.Json` are set to exactly version `10.0.1`.  All projects that reference this package will be locked to that version.  If someone attempts to specify a version in a project they will encounter a build error.

**Packages.props**
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

**SampleProject.csproj**
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

### Global Package References
Some packages should be referenced by all projects in your tree. This includes packages that do versioning, extend your build, or do any other function that is needed repository-wide. 

**Packages.props**
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

### Enforcement

If a user attempts to add a version to a project, they will get a build error:

```
The package reference 'Newtonsoft.Json' should not specify a version.  Please specify the version in 'C:\repo\Packages.props'.
```

If a user attempts to add a package that does not specify a version in `Packages.props`, they will get a build error:

```
The package reference 'Newtonsoft.Json' must have a version defined in 'C:\repo\Packages.props'.
```

### Extensibility

Setting the following properties control how Traversal works.

| Property                            | Description |
|-------------------------------------|-------------|
| `CentralPackagesFile `  | The full path to the file containing your package versions.  Defaults to `Packages.props` at the root of your repository. |

**Example**

Use a custom file name for your project that defines package versions.
```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <CentralPackagesFile>$(MSBuildThisFileDirectory)MyPackageVersions.props</CentralPackagesFile>
  </PropertyGroup>
</Project>
```
