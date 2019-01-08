# Restore fails in Visual Studio when dependent project not part of current solution

* Status: **Implemented**
* Author(s): [Ashish Jain](https://github.com/jainaashish)

## Issue

[5820](https://github.com/NuGet/Home/issues/5820) - Restore inside Visual Studio fails for projects when they reference other projects not included in solution

## Problem

While restoring a project which uses `PackageReference`, NuGet also go through each P2P (Project2Project) reference through VS DTE to gather all the transitive dependencies but when any of the P2P reference is either not part of current solution or unloaded, NuGet can't get this information and hence fail the restore which also fails the build if build was the uber operation.

This issue will become more apparent with Visual Studio 2019 which have option to selectively load some projects in large solutions.

## Who are the Customers

All NuGet users who uses PackageReference to manage their NuGet dependencies and wants to load only subset of their large solutions inside Visual Studio or have multiple solutions with cross project references.

## Solution

The proposed solution is to persist the NuGet dependency graph for each project while restoring and used that persisted information in future restores if some of the P2P references becomes unavailalbe at later stage. NuGet already evaluates the whole dependency graph as part of each restore and persist the SHA hash of this graph in `obj\<projectname>.nuget.cache`, so along with hash, we'll also persist the whole graph in a separate file called `obj\<projectname>.nuget.dgspec.json`.

This solution will still not solve first restore and customers still need to either run first restore from command-line or make sure full solution is loaded inside VS to generate this new file. Once this file is generated then we'll continue to use that for unavailalbe projects.

We already persist this dependency graph if you're running NuGet debug bits or if you set `NUGET_PERSIST_NOOP_DG` environment variable to true. With this proposal, we're making it a first class support to persist this graph to a welll defined file path with each restore.

This dependency graph has two sections:  
A. "restore", which is a list of project paths needs to be restored. This will always be single value of current project for which this graph has been calculated.  
B. "projects", list of `PackageSpec` objects for each project which is part of current project dependency graph. This `PackageSpec` object contains all the relevant details, required by NuGet to successfully do a restore.  

So lets look at a simple exaple to see how this proposed solution will work. Assume there is a solution solutionA which has two projects: projectA and projectB. ProjectA has a project reference to ProjectB.

1. First restore will generate `obj\projectA.csproj.nuget.dgspec.json` and `obj\projectB.csproj.nuget.dgspec.json` for the respective project's build output folder.

2. Next restore inside Visual Studio,  
  a. If nothing changed in projects/ solution, then NuGet doesn't consume anything from this new file and restore NoOp is successful (Current behavior).  
  b. If nothing changed and projectB not loaded, then NuGet consume missing `PackageSpec` for projectB from this new file and successfully complete the restore NoOp for projectA (New behavior).  
  c. If projectA changed but both projects are still loaded, then NuGet doesn't consume anything from this new file. Run restore and update this new file as well as other NuGet artifacts (Current behavior).  
  d. If projectA changed and projectB not loaded, then NuGet consume `PackageSpec` for projectB from this new file and complete the restore for projectA. NuGet will also update this new file with new changes along with persisting packageB `PackageSpec` (New behavior).  

## Open Questions

1. Any concern for file name?  
A. For now we're going ahead with `obj\<projectname>.nuget.dgspec.json`

2. Should there be an option for customers to opt-out of this experience? Should customer be allowed to say that dont use persisted information instead fail restore if you can't find my project dependencies?  
A. For now we decided not to do this.  

3. Should we update our existing `NU1105` error message when there is no dgspec file or it's still missing some projects? So instead of asking them to load or add project into current solution, ask them to run a restore from command line and then come back in VS.  
A. Yes, we'll update the message slightly to reflect this change.

4. Should we remove persiting the same dg file with current ENV variable and Debug mode?  
A. Yes, we'll remove those files.
