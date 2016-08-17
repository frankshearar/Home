# NuGet Restore Manager
Introducing NuGet Restore Manager (NRM) - unified component handling restore operations in VS "15" in efficient and user friendly fashion.

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
### Future VS "15" Restore Behavior
| Project Model | Project Open | Project.json File Save | Package Manager UI | Settings Change, dependent project happens, lock file gone | Project Build |
| --- | --- | --- | --- | --- | --- |
| automatic restore | Restore Happens | Restore Happens | Adaptions to UI requests are made right then. | Restore Happens | Restore is assumed to be not needed. |
| on-build only | | | same | | Restore happens |
| manual only | | | | | |


### Tenets
1. Native support for .NET Core and UWP projects restore. Replace WebTools restore.
1. Unified architecture for all VS project models.
1. Lightweight restore agent (NuGet.RestoreManager.dll). Loads fast- either as a 2nd packagedef in vsix or as a new mef component exported from vsix. No dependencies.
1. Project load cannot be blocked by NRM. Avoid any heavy activity within time to first edit.
1. Restore is the first part of build.
1. Understand when a restore is needed.
  * Packages.config – read xml file, and verify packages are installed in proj package folder.
  * UWP – lightweight noop pass – does assets file exist…are all libraries listed in the assets file installed in fallback folders/or user package folder. Minimize the package is installed verification (compare hashes of inputs). Are nuget.config files same that were used to build assets files. if project.json file is different than the one used to build the assets file…
  * Csproj with package refs only - the same as UWP except theres no project.json. Don't watch for csproj file changes. Listen to project system event (new event that needs to be raised by legacy project system)
  * Csproj with package refs and CPS - restore projects are nominated by CPS.


Separate Note: will we have a ResolvePackageReferences. So that we can have a target add a packageref

In VS, we do not call msbuild. We call restore with appropriate state.

### Open Issues
- [ ] How to bootstrap NRM on project open or new?
  * Export a MEF component
  * Second packagedef in vsix. Have an async package -- v2 vsix can do it.
- [ ] How to get info from VS at bootstrap time? 
  * Packages.config - project directories, packages directory
    * NuGet.Config discovery is expensive.
    * Don't use com marshalling, marshall ourselves (JTF), run at a priority less than user input [NOTE: do this more places…in NuGet code]
  * UWP Project.json - dg info (needs to keep updated)
    * can be gotten from solutionbuildmanager.getprojectdependencies(). We don't need updates … we just get the projectdependencies() when we decide the project likely needs to be restored.
  * Csproj with package refs only - dg info (updated), plus whatever restore algorithm needs ??? 
  * Csproj with package refs and CPS - not needed at bootstrap time, see below. cps will find us via a mef import
- [ ] *any* changes watcher
  * Project.json
  * .csproj not needed to be watched.
  * packages.config is not monitored today. POR is the same for the NRM.
  * Settings changes (nuget.config)
    * What if user adds new nuget.config in a search path? User needs to force a restore. [use vs file monitor…much better than .net fx one]
  * Note: need logic for a blank file on how to persist projectref
  * Dependency changes / dg info for uwp (check if we need this as separate update)
  * Assets file presence
- [ ] CPS "Restore Nominator" Capability - csproj integration, capabilities?
  * When nominating, CPS supplies project dir, intermediate dir, restore output type (uap, netcore), dg graph. This info could be done either way. Pass in from nominator, or query async apis at that point.
- [ ] How to block the build until we have full info from VS?
  * Virtual Project?
  * The queuing method returns a task (IVSTask). NRM would “complete” the task once a no-op restore is done…or a restore. 
VS/project system would block build while any tasks are still not complete.
We either need to have a non-project that can participate?
Or we need to use a virtual project?
Or we have onbuild event…we kick vs into build state and we cancel the original build.  After restore you kick off build again.
 
SolutionBuildManager sjhould provide an async pre-build event. We would drain all restores…then build

- [ ] Throttling and dials to control how often a restore can happen.
- [ ] Surfacing restore errors regardless of when restore happens.
- [ ] Coordination with Project system. Should they show 1000 intellisense errors before restore is done?
While restore happens do not show errors
How does VS know? Need to pull in Roslyn IDE --- Jason/Kevin/etc…
Restore fails … doing a build … how can these errors still show?
Show the errors in the asset file, or next to the assets file? who puts them back in the error list. How does that work for CPS/csproj.

- [ ] UI Notification of restore process… ideally, we show progress bar in vs status bar.
- [ ] Coordinate VS restore with command line restore