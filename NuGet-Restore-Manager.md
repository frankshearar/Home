# NuGet Restore Manager
Introducing NuGet Restore Manager - unified component handling restore operations in VS "15" in efficient and user friendly fashion.

## Problem
User experience in VS "15" is broken when working with .NET Core and UWP projects (or any other project facilitating project.json dependency management). 

- Intellisense feature is not available until build is started or "Restore NuGet Packages" menu is clicked. This problem occurs when existing project was checked out from a git repository with no project.lock.json file added.

- Changes made to project.json (or msbuild project file in future) via manual file editing are not reflected in Intellisense immediately. Restore operation is not triggered automatically. NuGet Package Manager UI won't suggest restoring packages as it does in packages.config world.

- Restore operation blocks UI with showing modal restore popup dialog.

  [[https://github.com/NuGet/Home/raw/alpanov/restore-manager-spec/resources/RestoringPackagesPopup.png|alt=restoring-packages]]

## Who is the customer?
Software developers using Visual Studio "15" to build .NET Core and UWP applications and libraries.

## Current Behavior
In VS2015 NuGet employs different mechanisms to initiate restore operation.

| Project Model | Project Open | Sidecar File Save | Package Manager UI | Project Build |
| --- | --- | --- | --- | --- |
| Packages.config | If restore is needed and any NuGet UI is open, we’ll yellow bar notify them to “restore” via clicking a button. | Not a scenario to only edit the file, because you wouldn’t get references added or packages downloaded. *websites are different… | Adaptions to UI requests are made right then. | Build will do a restore, if one is needed. |
| UWP style project.json | | | Adaptions to UI requests are made right then. | Restore happens (currently started by NuGet vsix in OnBuild event – downside is that the vsix needs to be loaded before build) |
| Xproj style project.json | Restore happens (.net core project system does this) | Restore happens (.net core project system does this) | Adaptions to UI requests are made right then. |
		

## Solution
### Future VS15 Behavior
| Project Model | Project Open | Project.json File Save | Package Manager UI | Settings Change, dependent project happens, lock file gone | Project Build |
| --- | --- | --- | --- | --- | --- |
| csproj + project.json (for both UWP or .NET Core) | Restore Happens | Restore Happens | Adaptions to UI requests are made right then. | Restore Happens | |

### Tenets
1. Native support for .NET Core and UWP projects restore. Replace WebTools restore.
2. Unified architecture for all VS project models.
3. Lightweight restore agent (NuGet.RestoreManager.dll). Loads fast- either as a 2nd packagedef in vsix or as a new mef component exported from vsix. No dependencies.
4. Understand when a restore is needed.
  * Packages.config – read xml file, and verify packages are installed in proj package folder.
  * UWP – lightweight noop pass – does assets file exist…are all libraries listed in the lock file installed in fallback folders/or user package folder. Is nuget.config newer than assets file. if p.j file is different than the one used to build the assets file…
  * Csproj only

### Open Issues
- [ ] How to bootstrap N.R.M. on project open or new?
	- Mef component
	- Second packagedef in vsix
- [ ] How to get info from VS at bootstrap time? 
	- Packages.config - project directories, packages directory
		- NuGet.Config discovery is expensive
	- UWP Project.json - dg info (needs to keep updated)
- [ ] *any* changes watcher
	- Project.json
	- Settings changes
	- Dependency changes / dg info for uwp (check if we need this as separate update)
	- Lock file presence
- [ ] CPS "Restore Nominator" Capability - csproj integration, capabilities?
	- When nominating CPS supplies project dir, intermediate dir, restore output type (uap, netcore), dg graph
- [ ] How to block the build until we have full info from VS?
	- Virtual Project?
- [ ] Throttling and dials to control how often a restore can happen.
- [ ] Coordination with Project system … should they show 1000 errors…