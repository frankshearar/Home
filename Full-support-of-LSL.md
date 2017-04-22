# [WIP][Spec] Full support of the Lightweight Solution Load

## Problem

NuGet client in VS is not fully functional in LSL mode. Currently only restore operation is supported. The main problem is `NuGetProject` and other componenets are not aware of LSL mode as all information from VS are retrieved via `EnvDTE` or CPS API.

## Solution

NuGet VS client needs to support LSL mode without breaking it.

Development activity is tracked in NuGet/Home#5058.

### List of NuGet operations per project model

Operation|packages.config|project.json|PackageRef|.NET Core
--- | --- | --- | --- | ---
Manage packages (solution)| Supported | Supported | Supported
Manage packages (project)| Loads on right-click | Loads on right-click | Loads on right-click
Install package (PM)| Loads project
Uninstall package (PM)| Loads project
Update package (PM)| Loads project
Execute `init.ps1` (PMC)| Supported
`Add-BindingRedirect` | Loads project
`Find-Package` | Supported
`Get-Package` | Supported
`Get-Project` | Loads project
`Install-Package` | Loads project
`Open-PackagePage` | Supported
`Sync-Package` | Supported
Tab expansion (PMC)| Supported
`Uninstall-Package`| Loads project
`Update-Package` | Loads project
`IVsPackageInstallerServices`| Supported | Supported | Supported

### Shared Infrastructure

A project adapter will be introduced to abstract away communication with `EnvDTE` and CPS API where needed. The adapter implementation will be able to detect LSL mode and switch between data sources and APIs. It will force load project if no equivalent data is available in LSL DB.

### packages.config

In LSL mode it facilitates `PackagesConfigReader' to retrieve package references. Works correctly in LSL mode for majority of read-only operations. For install, uninstall, update operations will force load project as it needs to update assembly references in the project.

### project.json
Utilizes `JsonPackageSpecReader` to get a PackageSpec for the project.

### PackageRef
Relies on `IDeferredProjectWorkspaceService` to get all necessary project data.

### .NET Core (Roslyn project system)
Does not support LSL. No action needed.