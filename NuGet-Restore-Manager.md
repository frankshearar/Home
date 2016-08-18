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

<!--
<sub>...</sub> is used to make font size small
-->

| Project Model | Project Open | Sidecar File Save | Package Manager UI | Project Build |
| --- | --- | --- | --- | --- |
| Packages.config | <sub>If restore is needed and any NuGet UI is open, we’ll yellow bar notify them to “restore” via clicking a button.</sub> | <sub>Not a scenario to only edit the file, because you wouldn’t get references added or packages downloaded. *websites are different…</sub> | <sub>Adaptions to UI requests are made right then.</sub> | <sub>Build will do a restore, if one is needed.</sub>
| UWP style project.json | | | <sub>Adaptions to UI requests are made right then.</sub> | <sub>Restore happens (currently started by NuGet vsix in OnBuild event – downside is that the vsix needs to be loaded before build)</sub>
| Xproj style project.json | <sub>Restore happens (.net core project system does this)</sub> | <sub>Restore happens (.net core project system does this)</sub> | <sub>Adaptions to UI requests are made right then.</sub>
		

## Solution
### Future VS "15" Restore Behavior
| Project Model | Project Open | Project.json File Save | Package Manager UI | Settings Change, dependent project happens, lock file gone | Project Build |
| --- | --- | --- | --- | --- | --- |
| automatic restore | <sub>Restore Happens</sub> | <sub>Restore Happens</sub> | <sub>Adaptions to UI requests are made right then.</sub> | <sub>Restore Happens</sub> | <sub>Restore is assumed to be not needed.</sub>
| on-build only | | | <sub>same</sub> | | <sub>Restore happens</sub>
| manual only


### Tenets
1. Unified architecture for all VS project models.
1. Native support for .NET Core restore. Replace WebTools restore.
1. Lightweight restore agent (NuGet.RestoreManager.dll). Loads fast. No dependencies.
  * Project load cannot be blocked by NRM. Avoid any heavy activity within time to first edit.
1. Export MEF restore packages service. Implements standard "generic" restore service interface.
1. Restore is the first part of build.
1. Evaluate when a restore is needed.

## Design Notes

Separate Note: will we have a ResolvePackageReferences. So that we can have a target add a packageref

In VS, we do not call msbuild. We call restore with appropriate state.

### Bootstrap the NRM
Define second packagedef in vsix by introducing new `VsPackage`.
Inherits from `AsyncPackage`.
Loads on specific `UIContext` asynchronously.
Exports MEF restore component/service.

### Evaluate when restore is needed

| Project Model | When | Description
| --- | --- | --- |
| All | <sub>*any* changes watcher</sub> | <sub>Project.json<br>.csproj not needed to be watched.<br>Note: need logic for a blank file on how to persist projectref<br>Dependency changes / dg info for uwp (check if we need this as separate update)<br>Assets file presence</sub>
| | <sub>Settings changes (nuget.config)</sub> | <sub>What if user adds new nuget.config in a search path? User needs to force a restore. [use vs file monitor…much better than .net fx one]>/sub>
| Packages.config | <sub>Always</sub> | <sub>read xml file, and verify packages are installed in proj package folder.</sub>
| | <sub>Bootstrap</sub> | <sub>Read project directories, packages directory.<br/>NuGet.Config discovery is expensive.<br>Don't use com marshalling, marshall ourselves (JTF)<br>Run at a priority less than user input [NOTE: do this more places…in NuGet code]</sub>
| | <sub>Tracking Changes| <sub>packages.config is not monitored today. POR is the same for the NRM.</sub>
| UWP | | <sub>lightweight noop pass – does assets file exist…are all libraries listed in the assets file installed in fallback folders/or user package folder. Minimize the package is installed verification (compare hashes of inputs). Are nuget.config files same that were used to build assets files. if project.json file is different than the one used to build the assets file…</sub>
| | <sub>Bootstrap</sub> | <sub>dg info (needs to keep updated)<br>can be gotten from solutionbuildmanager.getprojectdependencies(). We don't need updates … we just get the projectdependencies() when we decide the project likely needs to be restored.</sub>
| Csproj with package refs only | | <sub>the same as UWP except theres no project.json. Don't watch for csproj file changes. Listen to project system event (new event that needs to be raised by legacy project system).</sub>
| | <sub>Bootstrap</sub> | <sub>dg info (updated), plus whatever restore algorithm needs ???</sub>
| Csproj with package refs and CPS | | <sub>restore projects are nominated by CPS.</sub>
| | <sub>Bootstrap</sub> | <sub>not needed at bootstrap time, see below. cps will find us via a mef import</sub>

### Open Issues
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