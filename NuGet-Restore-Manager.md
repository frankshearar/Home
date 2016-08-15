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

## Evidence
_What is the evidence that behaves us to act?_
_Evidence can be impassioned tweets or mails, mile long conversations in issues, rants on blogs, sweet sweet data from telemetry!!! If you can show pain, you can rally the troops._

## Current Behavior
In VS2015 NuGet employs different mechanisms to initiate restore operation.

| Project Model | Project Open | Sidecar File Save | Package Manager UI | Project Build |
| --- | --- | --- | --- | --- |
| Packages.config | If restore is needed and any NuGet UI is open, we’ll yellow bar notify them to “restore” via clicking a button. | Not a scenario to only edit the file, because you wouldn’t get references added or packages downloaded. *websites are different… | Adaptions to UI requests are made right then. | Build will do a restore, if one is needed. |
| UWP style project.json | | | Adaptions to UI requests are made right then. | Restore happens (currently started by NuGet vsix in OnBuild event – downside is that the vsix needs to be loaded before build) |
| Xproj style project.json | Restore happens (.net core project system does this) | Restore happens (.net core project system does this) | Adaptions to UI requests are made right then. |
		

Support .NET Core and UWP projects.
Replace WebTools restore.
Enable intellisense early as restore happens.
Restore on build, avoid modal restore popup. 
Support VS "15" project update scenario
No-op optimization.
project.json changes.
Have unified architecture to support all VS project models

## Solution
### Design meeting notes
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