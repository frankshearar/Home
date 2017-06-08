# [WIP][Spec] Full support of the Lightweight Solution Load

## Problem

NuGet client in VS is not fully functional in LSL mode. Currently only restore operation is supported. The main problem is `NuGetProject` and other components are not aware of LSL mode as all information from VS are retrieved via `EnvDTE` or CPS API.

## Solution

NuGet VS client needs to support LSL mode without breaking it.

Development activity is tracked in NuGet/Home#5058.

### List of NuGet operations per NuGet package management project model

Operation|packages.config|project.json|PackageRef|.NET Core
--- | --- | --- | --- | ---
Restore | <sub>Supported</sub> | <sub>Supported</sub> | <sub>Supported</sub> | <sub>N/A</sub>
Manage packages (solution)| <sub>Supported</sub> | <sub>Supported</sub> | <sub>Supported</sub> | <sub>N/A</sub>
Manage packages (project)| <sub>Loads on right-click</sub> | <sub>Loads on right-click</sub> | <sub>Loads on right-click</sub> | <sub>N/A</sub>
Install package (PM)| <sub>Loads project</sub> | <sub>Supported</sub> | <sub>Loads project</sub> | <sub>N/A</sub>
Uninstall package (PM)| <sub>Loads project</sub> | <sub>Supported</sub> | <sub>Loads project</sub> | <sub>N/A</sub>
Update package (PM)| <sub>Loads project</sub> | <sub>Supported</sub> | <sub>Loads project</sub> | <sub>N/A</sub>
Execute `init.ps1` (PMC)| <sub>Supported</sub> | <sub>Supported</sub> | <sub>Supported</sub> | <sub>N/A</sub>
`Add-BindingRedirect` | <sub>Loads project</sub> | <sub>Loads project</sub> | <sub>Loads project</sub> | <sub>N/A</sub>
`Find-Package` | <sub>N/A</sub> | <sub>N/A</sub> | <sub>N/A</sub> | <sub>N/A</sub>
`Get-Package` | <sub>Supported</sub> | <sub>Supported</sub> | <sub>Supported</sub> | <sub>N/A</sub>
`Get-Project` | <sub>Loads project</sub> | <sub>Loads project</sub> | <sub>Loads project</sub> | <sub>N/A</sub>
`Install-Package` | <sub>Loads project</sub> | <sub>Supported</sub> | <sub>Loads project</sub> | <sub>N/A</sub>
`Open-PackagePage` | <sub>N/A</sub> | <sub>N/A</sub> | <sub>N/A</sub> | <sub>N/A</sub>
`Sync-Package` | <sub>Supported</sub> | <sub>Supported</sub> | <sub>Loads project</sub> | <sub>N/A</sub>
Tab expansion (PMC)| <sub>Supported</sub> | <sub>Supported</sub> | <sub>Supported</sub> | <sub>N/A</sub>
`Uninstall-Package`| <sub>Loads project</sub> | <sub>Supported</sub> | <sub>Loads project</sub> | <sub>N/A</sub>
`Update-Package` | <sub>Loads project</sub> | <sub>Supported</sub> | <sub>Loads project</sub> | <sub>N/A</sub>
`IVsPackageInstallerServices`| <sub>Supported</sub> | <sub>Supported</sub> | <sub>Supported</sub> | <sub>N/A</sub>

### Shared Infrastructure

A project adapter will be introduced to abstract away communication with `EnvDTE` and CPS API where needed. The adapter implementation will be able to detect LSL mode and switch between data sources and APIs. It will force load project if no equivalent data is available in LSL DB.

Project services will be introduced where project system specific implementation is needed thus the project adapter alone won't fit.

### packages.config

In LSL mode it facilitates `PackagesConfigReader` to retrieve package references. Works correctly in LSL mode for majority of read-only operations. For install, uninstall, update operations will force load project as it needs to update assembly references in the project.

### project.json
Utilizes `JsonPackageSpecReader` to get a PackageSpec for the project.

### PackageRef
Relies on `IDeferredProjectWorkspaceService` to get all necessary project data.

### .NET Core (Roslyn project system)
Does not support LSL. No action needed.