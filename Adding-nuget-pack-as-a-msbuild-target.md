# Title
Adding pack as a msbuild target for csproj.

## Problem
As part of an effort to move restore, build, package and publish to a unified msbuild pipeline in cross platform environments, we need to have a fully .NET Core implementation of pack. dotnet pack will need to be replaced to call into the new pack target in msbuild that can run cross platform and support cross targeting scenarios. As a result, project.json will go away and all metadata and package references from nuspec and project.json will move into csproj.


## Solution
In order for msbuild to be able to gather all the inputs, all metadata from project.json and nuspec will move into csproj file. Below is how the new metadata would look like in csproj, and how it will map to output nuspec:

Attribute/NuSpec Value| MSBuild Property | Default | Notes
--- | --- | --- | ---
Id|PackageId|AssemblyName|
Version|PackageVersion|Version|
Authors|Authors||Needs discussion on default
Owners|N/A|Not present in NuSpec|
Description|Description|empty
Copyright|Copyright|empty
Summary|N/A|Not present in NuSpec|Being removed
RequireLicenseAcceptance|PackageRequireLicenseAcceptance|false
LicenseUrl|PackageLicenseUrl|empty
ProjectUrl|PackageProjectUrl|empty
IconUrl|PackageIconUrl|empty
Tags|PackageTags|empty
ReleaseNotes|PackageReleaseNotes|empty
RepositoryUrl|RepositoryUrl|empty
RepositoryType|RepositoryType|empty
PackageType|`<PackageType>DotNetCliTool, 1.0.0.0;Package, 2.0.0.0</PackageType>`||NuGet will go offline and figure this out