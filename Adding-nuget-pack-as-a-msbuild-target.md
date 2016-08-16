# Title
Adding pack as a msbuild target for csproj.
## Feedback
All feedback can be provided on the tracking issue: https://github.com/NuGet/Home/issues/2995
## Problem
As part of an effort to move restore, build, package and publish to a unified msbuild pipeline in cross platform environments, we need to have a fully .NET Core implementation of pack. dotnet pack will need to be replaced to call into the new pack target in msbuild that can run cross platform and support cross targeting scenarios. As a result, project.json will go away and all metadata and package references from nuspec and project.json will move into csproj.


## Solution
In order for msbuild to be able to gather all the inputs, all metadata from project.json and nuspec will move into csproj file. Below is how the new metadata would look like in csproj, and how it will map to output nuspec:

Attribute/NuSpec Value| MSBuild Property | Default | Notes
--- | --- | --- | ---
Id|PackageId|AssemblyName|$(AssemblyName) from msbuild
Version|PackageVersion|Version|New $(Version) property from msbuild, is semver compatible. Could be “1.0.0”, “1.0.0-beta”, or “1.0.0-beta-00345”. 
Authors|Authors|username of the current user will be the default value|
Owners|N/A|Not present in NuSpec|
Description|Description|"Package Description"|
Copyright|Copyright|empty
RequireLicenseAcceptance|PackageRequireLicenseAcceptance|false
LicenseUrl|PackageLicenseUrl|empty
ProjectUrl|PackageProjectUrl|empty
IconUrl|PackageIconUrl|empty
Tags|PackageTags|empty
ReleaseNotes|PackageReleaseNotes|empty
RepositoryUrl|RepositoryUrl|empty
RepositoryType|RepositoryType|empty
PackageType|`<PackageType>DotNetCliTool, 1.0.0.0;Package, 2.0.0.0</PackageType>`||


In addition, the following nuspec properties will no longer be supported in csproj file :
* Title
* Summary

###Pack Target Inputs
Properties:
* PackageVersion
* PackageId
* Authors
* Description
* Copyright
* PackageRequireLicenseAcceptance
* PackageLicenseUrl
* PackageProjectUrl
* PackageIconUrl
* PackageReleaseNotes
* PackageTags
* PackageOutputPath
* Configuration
* AssemblyName
* IncludeSymbols
* PackageTypes
* IsTool
* RepositoryUrl
* RepositoryType
* NoPackageAnalysis
* MinClientVersion

Items:
* SourceFiles (if IncludeSymbols = true).
* PackageFiles (needs design, but basically means the list of content files to be included).
* TargetPath (cross targeting scenarios).
* TargetFrameworks (cross targeting scenarios).
* ProjectReferences (has a custom serialization format, more details coming up).

###Scenarios
####PackageIconUrl
As part of the change for feature, https://github.com/NuGet/Home/issues/2582, PackageIconUrl will eventually be changed PackageIconUri and can be relative path to a icon file which will included at the root of the resulting package.
####Output Assemblies
NuGet pack will copy the output assemblies from the directory list obtained from $(TargetDir) (This will be a list of directories in Cross-Targeting scenario) . The output assemblies to be copied depend on the following criteria:
* Any filename that matches the $(AssemblyName) & has the extension .exe,.dll,.xml or .winmd (or .pdb if IncludeSymbols=true).
* Any filename that matches the AssemblyName of a ProjectReference that does not have ReferenceOutputAssembly=false and has the extension .exe,.dll,.xml or .winmd (or .pdb if IncludeSymbols=true).

Note that ProjectReference assemblies are only searched for in host project's output directory in the corresponding target framework folder. They are copied out to lib\\\<TargetFrameworkShortName>\\\<fileName>

####Package References
TODO: Link to the spec for package reference, which is still being designed.
####Project to Project References
Project to Project references will be, by default, be considered as nuget package references. However, this behavior can be overridden in the following manner:
    
     <ProjectReference Include="..\UwpLibrary2\UwpLibrary2.csproj">
         <Project>{25dcfe98-02b7-403a-b73d-6282d9801aa1}</Project>
         <Name>UwpLibrary2</Name>
         <TreatAsPackageReference>false</TreatAsPackageReference>
     </ProjectReference>

If a referenced project's output DLL is to be copied over to the nuget package, then **ReferenceOutputAssembly should not be set as false**. This is because the output DLL of the referenced project is copied from the output directory of the project being packed. For more details on ReferenceOutputAssembly , check out : https://blogs.msdn.microsoft.com/kirillosenkov/2015/04/04/how-to-have-a-project-reference-without-referencing-the-actual-binary/

If IsPackageReference is not specified, or is set to true, then the ProjectReference will actually be added as a Package Reference in the output nuspec, and no DLLs will be copied.

Note that this behavior is recursive - so if a ProjectReference has TreatAsProjectReference set to false, it's project to project references will also be treated in the same manner.

####Including Content in package
Currently being discussed with MSBuild team. More details coming soon.

####Cross Targeting
As per the details available right now, Target frameworks are defined in the csproj in an item list (called TargetFramework right now) where the identity maps to $(TargetFrameworkIdentity),$(TargetFrameworkVersion) - no NuGet short names.
  <ItemGroup>
    <TargetFramework Include=".NetFramework,v4.5" />
    <TargetFramework Include=".NetFramework,v4.6" />
  </ItemGroup>

The @(TargetPath) will be a list of all the output paths (path to the output assembly) with their associated TargetFramework metadata. NuGet pack will convert these full target framework names to short folder names (in above case net45 and net46) in the resulting nupkg.

Note that details on Cross Targeting are still being finalized, so this may change.

####IncludeSymbols
If msbuild /t:pack /p:IncludeSymbols=true , then the corresponding pdb files are copied along with .dll/.exe/.winmd/.xml . Note that setting IncludeSymbols= true creates a regular package AND a symbols package.

####IncludeSource
All files of type = Compile are copied over to src\\\<ProjectName>\ preserving the relative path directory structure in the resulting nupkg. The same also happens for any ProjectReference which has \<TreatAsPackageReference> set to false.

If a file of type = Compile, is outside the project folder, then it is just added to src\\\<ProjectName>\\.

IncludeSource should be used in conjunction with IncludeSymbols to have any effect.

####IsTool
If msbuild /t:pack /p:IsTool=true, all output files, as specified in the Output Assemblies scenario, are copied to the tools folder instead of the lib folder. Note that this is different from a DotNetCliTool which is specified by setting the PackageType in csproj file.

####Adding reference to other PCLs or TFMs
More details coming soon.
