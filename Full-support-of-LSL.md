## [WIP][Spec] Full support of the Lightweight Solution Load

NuGet VS client needs to support LSL mode without breaking it.

### List of NuGet operations
Operation|packages.config|project.json|PackageRef|.NET Core
--- | --- | --- | --- | ---
Manage packages (solution)| Full | Full | Full | Full
Manage packages (project)| TBD
Install package (PM)| Force load
Uninstall package (PM)| Force load
Update package (PM)| Force load
Execute scripts (PMC)| Full
Add-BindingRedirect | Force load
Find-Package | Full
Get-Package | Full
Get-Project | Force load
Install-Package | Force load
Open-PackagePage | TBD
Sync-Package | TBD
Tab expansion (PMC)| TBD
Uninstall-Package| Force load
Update-Package| Force load
IVsPackageInstaller| TBD



