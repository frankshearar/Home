## NuGet Restore No-Op

A different no-op for VS was implemented in VS 2019.7, see [spec].(https://github.com/NuGet/Home/blob/dev/designs/VisualStudio-PartialRestoreOptimization.md)
### What is NuGet Restore No-Op
Currently in 3 of the 4 NuGet Clients (NuGet.exe, dotnet.exe, MSBuild) we do a full restore at every restore call.
The VS Client currently has a non-configurable no-op implemented. 
The primary goal of this feature is to optimize restore so that if the restore result is up-to-date, the corresponding restore operation is not executed. 
The secondary goal of this feature is to unite the restore behavior across all 4 clients. 
Meaning a project was restored by a commandline client, when it's loaded into VS, under normal scenarios it should not need to restore.

### Benefits
The benefits are a faster and more seamless restore experience, avoid doing redundant work when the user has 

### No-Op Behavior
| Project Model | NuGet.Exe | VS Client | dotnet.exe | MSBuild |
| --- | --- | --- | --- | --- |
| Packages.config | <sub>Not covered. NuGet.exe restores packages.config projects first</sub> | <sub>Not covered. VS Client restores packages.config projects first</sub> | <sub>Not covered. packages.config projects are ignored</sub> | <sub>Not covered. packages.config projects are ignored</sub>
| project.json | <sub>Covered.</sub> | <sub>Covered.</sub> | <sub>Covered.</sub> | <sub>Covered.</sub>
| Package Reference | <sub>Covered.</sub> | <sub>Covered.</sub> | <sub>Covered.</sub> | <sub>Covered.</sub>

### Usage:
The no-op restore will run by default. If you wish to skip the no-op, you can use -force as such:
* NuGet.exe restore MyProject.csproj -force 
* Dotnet.exe restore MyProject.csproj -force 
* MSBuild.exe /t:restore MyProject.csproj /p:RestoreForce
* Visual Studio - NuGet Package Manager - rebuild / Clean, which will delete the cache file, and then any restore.

## Design Specification 

### What is the input/output to restore
* Inputs affecting NuGet restore of a project:
    * projectName.csproj
    * NuGet.config(s)
    * Properties passed in msbuild
    * Environment variables
        * NUGET_PACKAGES & NUGET_FALLBACK_PACKAGES
        * Environment variables that affect msbuild and change pack ref/proj refs

* Output of NuGet restore:
    * project.assets.json
    * projectName.csproj.nuget.props
    * projectName.csproj.nuget.g.targets

### How do we know if we can no-op?
We need to answer 3 questions to determine whether the restore operation will be a no-op. 

* Are all variables for resolving the packages correct?
    * Package References 
    * Project References
    * Target Frameworks
    * Runtime Identifiers
    *  Guardrails 
* Are all the correct packages there? 
    * Are the same sources used? 
        * NuGet.Config
        * -S SourceName

* Have the NuGet Global Packages Folder & Fallback folder paths changed?
    * NuGet.Config
    * Environment Variables
    * Do the hashes from project.assets.json match the ones found in the Global Packages/ Fallback Folders 
	
* Does restore no-op for all child projects?
    * Self-explanatory, due to the transitive nature of package references, all child projects of the project being restore need to no-op too. 

### Implemented solution

Leverage the existing dgspec graph. 

Contents of the dg spec:
* Dependencies
* Target Frameworks
* RIDs
* Guardrails
		
The dg spec almost covers all the factors we need to check to perform the no-op. 
To make it complete we need to add the nuget settings. 
That include the resolved source and the Environment variables, more specifically NUGET_PACKAGES & NUGET_FALLBACK_PACKAGES folders. 

Content that need to be added to the dg spec:
* Sources
* NUGET_PACKAGES path
* NUGET_FALLBACK_PACKAGES path
* Used configuration file paths. 

Since the config files sometimes contain credentials, we need to persist the used configuration files. 
Since we don't want to write these credentials in the dg spec, we persist these configuration file paths, so that we reconstruct the settings object to ask for the credentials. is 

Once we have generated the dg spec, we will hash the contents and write it into a json file.
**projectName.type.nuget.cache**

Example content:

	{
	"dgspec" : "BA7816BF8F01CFEA414140DE5DAE2223B00361A396177A9CB410FF61F20015AD"
        "success" : "true"
        "version": 1
	}

This json file will live in BaseIntermediateOutputPath next to the project.assets.json file.
At each restore, we will compare the evaluated hash of the dg spec with the one currently in the BaseIntermediateOutputPath if existing.
If the hashes are the same, we then verify that the assets.json file exists, and then we verify all the packages exist on disk. Lastly we verify that msbuild files are on disk if required. If all these conditions are fulfilled  we no-op.

Step by step list on where we add the no-op logic. 

1. Read Settings (Includes Sources & resolve the paths for the packages folders)
2. Build DG SPEC
3. Hash the contents of the dg spec
4. Compare hash value with the dgspec value in projectName.type.nuget.cache if existing.  If true GOTO 5
5. Verify that the project.assets.json file exists. If true GOTO 6
6. Verify that all packages exist on disk (in packages folder). If this condition is satisfied too, we stop here since we have a no-op restore, otherwise we continue
7. Verify that the msbuild files exist (props, targets)
7. Restore as usual
8. If the restore is successful we write the projectName.type.nuget.cache with the new hash
	
Of course in case of a recursive restore we will need to do the same logic for all of the projects.

### Tool No-Op restore

If you're unfamiliar with tools, you can find more info here:
https://github.com/dotnet/docs/blob/master/docs/core/tools/extensibility.md#per-project-based-extensibility       
https://github.com/NuGet/Home/wiki/DotnetCliToolReference-restore 

Basically .NET Core projects can have a DotnetCliToolReference item, and when that is present, we will end up running 2 different restores for that project file, 1 of the package reference there, 1 for the tool. 
The tools contain a reference to the project that declared them, but that's only for logging purposes. The project that declares them does not affect the restore. 
The tools assets file is in the Packages folders .tools\toolName\version\tfm\
The cache file of the tool will be * toolName.nuget.cache *. 

An important thing to note here is that within a project, the tool declarations will be deduped, meaning if 10 ASP.NET projects in a solution have the same tool declaration, only 1 restore will happen. 
Figuring out the cache/assets file path for the tools is not that straightforward, since we don't know what version was actually restored, rather the requested one. Also the requested version could be a floating version, making it even more complex. 

So in the tools case, what we will do, is look at the packages folder and resolve all available versions of requested tool. 
If one matches, we attempt to use the cache file there. The uniqueName for the tool "project" restore is contains the version range of the requested tool. 
Meaning in package spec used to generate the hash for the cache file will contain the version range such as [1.0.0,).
Once the cache/lock files are resolved, the no-op restore continues as before. 

### Open Issues


The No-Op feature does not cover all cases. 

#### Floating/Open ended version requested - *Scenario breaking change*
Since the No-Op evaluation needs to be minimal, we avoiding checking the sources, so the case where a new version of the requested package is uploaded, or a version of the package is unlisted it might change the package resolution behavior.

Example:
```
<ItemGroup>
    	<!-- ... -->
    	<PackageReference Include="My.Awesome.Package" Version="3.6.*" />
	<!-- ... -->
</ItemGroup>
```
At the moment of the first restore, version="3.6.0" was the latest version on the package source.
Sometime between that restore and the new restore, version="3.6.1" was uploaded to the source. 
Running a full restore would resolve 3.6.1 and downloaded that version of the package. However since the restore inputs have not changed, restore will think no action needs to be taken.

This might cause issues for scenarios like the following:
A project(ProjX) that depends on a local feed that contains the latest bits of the desired projects (ProjA, ProjB etc).
ProjA is built and the nupkgs from that dropped into the local feed for the ProjX. 
ProjX is restored and packed. 
The expectation is that the nupkg generated by projX has the latest ProjA nupkg that was built. 
However since the restore parameters have not changed, the restore of ProjX might no-op. 
The workaround is to simply use the force option for scenarios like this. 

##### Potential improvements
* Make the cache expire, based on a timestamp. The timestamp can be either in the cache or just use the timestamp of when the file was written. This doesn't fix the above scenario though, it makes it less likely.
* Print a warning whenever we no-op on a project that has open ended/ floating dependencies. Caveat is this could become really noisy and users might hate it.

Tracking here: https://github.com/NuGet/Home/issues/5445

#### Cross-Client no-op

The secondary target of this feature is to unify the restore behavior among all our clients. 
One can restore with NuGet.exe, open VS, and then auto-restore no-ops. When that, the user can build use msbuild /t:restore and that would no-op as well. 

At this point that's not the case. 
The dg spec generated for msbuild/nuget.exe/dotnet.exe case is not the same as the one generated by VS. Some of those inconsistencies have been resolved, while others haven't. 

Here's a table of the presumed behavior (meaning, as far as we know, there might be some crazy scenarios in which this breaks, and when encountered, we'll want to fix them. 

| Project Model | NuGet.Exe & VS Client | NuGet.exe & dotnet.exe | NuGet.exe & MsBuild | VS Client & dotnet.exe | VS Client & MsBuild | MsBuild & dotnet.exe |
| --- | --- | --- | --- | --- | --- | --- |
| .NET Core - SDK based projects | <sub>Consistent</sub> | <sub>Consistent.</sub> | <sub>Consistent.</sub> | <sub>Consistent.</sub> | <sub>Consistent.</sub> | <sub>Consistent.</sub> |
| Legacy Package Reference projects| <sub>Inconsistent.</sub>| <sub>Consistent.</sub> | <sub>Consistent.</sub> | <sub>Inconsistent.</sub> | <sub>Consistent.</sub> | <sub>Inconsistent.</sub> |
| Project.Json  | <sub>Consistent</sub> | <sub>Consistent.</sub> | <sub>Consistent.</sub> | <sub>Consistent.</sub> | <sub>Consistent.</sub> | <sub>Consistent.</sub> |

Tracking issue for these inconsistencies here: https://github.com/NuGet/Home/issues/5444#### Tool No-Op issues
The design for tools has a lot of holes itself, as we bind it to a project, but it's actually global etc. 
Due to that there's a great amount of cases where tool restore will not work. 

The simple scenario here if we have 2 different version declarations that resolve to the same version. 
```
  <ItemGroup>
    <DotNetCliToolReference Include="dotnet-api-search" Version="0.9.5" />
  </ItemGroup>
```
and 
```
  <ItemGroup>
    <DotNetCliToolReference Include="dotnet-api-search" Version="1.0.0" />
  </ItemGroup>
```
Both these tools restores would resolve to 1.0.0 and both would end up writing to the same cache file.
In this case we're left at the mercy of the scheduler. Pigeonhole says only 1 restore can no-op, but since these tasks run asynchronously, it may happen that the cache file by the one that CAN no-op has been overriden by another one before we read the cache file. 
Due to multiple threads trying to access the lock/cache files, there's many inconsistencies in the tool no-op. 
All the known issues with tool no-op are related to the fact that multiple restores compete for the same assets/cache files. 

Tracking here: https://github.com/NuGet/Home/issues/5443
#### Legacy Package Reference Install followed by a non-force restore does not no-op.
This is an issue due to the fact that the install command in legacy projects installed top level (no framework), but in the generation of the package spec for the restore, it contains the references in both top level and per tfm.
Tracked here: https://github.com/NuGet/Home/issues/5419
