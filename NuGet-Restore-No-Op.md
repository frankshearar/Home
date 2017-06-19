## NuGet Restore No-Op

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
* MSBuild.exe /t:restore MyProject.csproj /p:force 
* Visual Studio - NuGet Package Manager - rebuild

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

- Potential improvements
** Make the cache expire, based on a timestamp. The timestamp can be either in the cache or just use the timestamp of when the file was written. This doesn't fix the above scenario though, it makes it less likely.
** Print a warning whenever we no-op on a project that has open ended/ floating dependencies. Caveat is this could become really noisy and users might hate it.