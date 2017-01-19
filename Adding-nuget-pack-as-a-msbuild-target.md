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
PackageType|`<PackageType>DotNetCliTool, 1.0.0.0;Dependency, 2.0.0.0</PackageType>`||


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
* IncludeSource
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
NuGet pack will copy the output files (which are of extension .exe, .dll, .xml, .winmd). The output files that are copied depend on what MSBuild provides from BuiltOutputProjectGroup target. 

There are two msbuild properties that you can use in your project file or command line to control where output assemblies go:

1) IncludeBuildOuput : This is a boolean, which decided whether the build output assemblies should be packed into the nupkg or not.

2) BuildOutputTargetFolder : Specify the folder in which the output assemblies should go to. The output assemblies (and other output files) are copied into their respective framework folders.

####Package References
Spec to Package References : https://github.com/NuGet/Home/wiki/PackageReference-Specification

####Project to Project References
Project to Project references will be, by default, be considered as nuget package references.
    
     <ProjectReference Include="..\UwpLibrary2\UwpLibrary2.csproj">
         <Project>{25dcfe98-02b7-403a-b73d-6282d9801aa1}</Project>
         <Name>UwpLibrary2</Name>
     </ProjectReference>

If a referenced project is not to be added as a package reference, then **ReferenceOutputAssembly should not be set as false**. This is because the output DLL of the referenced project is copied from the output directory of the project being packed. For more details on ReferenceOutputAssembly , check out : https://blogs.msdn.microsoft.com/kirillosenkov/2015/04/04/how-to-have-a-project-reference-without-referencing-the-actual-binary/

You can also add the following metadata to your project reference:

\<IncludeAssets>

\<ExcludeAssets>

\<PrivateAssets>

####Including Content in package

Add extra metadata to existing \<Content> item . By default everything of type "Content" gets included for pack, unless you override by specifying something like:

     <Content Include="..\win7-x64\libuv.txt">
         <Pack>false</Pack>
     </Content>

Everything gets added to the root of the **content** and **contentFiles\any\ \<TFM>** folder within a package and preserves the relative directory structure, unless you specify a package path: 

     <Content Include="..\win7-x64\libuv.txt">
         <Pack>true</Pack>
         <PackagePath>content\myfiles\</PackagePath>
     </Content>

If you want to copy all your content only to a specific root folder(s) (instead of **content** and **contentFiles** both), you can use the msbuild property **ContentTargetFolders** , which defaults to "content;contentFiles", but can be set to any other folder names. Note that just specifying "contentFiles" puts files under "contentFiles\any\ \<TFM>" or "contentFiles\ \<language>\ \<TFM>" based on buildAction.

PackagePath can be a semicolon delimited set of target paths.
Specifying an empty PackagePath string would add the file to the root of the package.
     
     <Content Include="..\win7-x64\libuv.txt">
         <Pack>true</Pack>
         <PackagePath>content\myfiles\;content\sample\;\;</PackagePath>
     </Content>

The above will add libuv.txt to content\myfiles , content\sample and the root of the package.

Packing of content files is recursive too. Content files from any project to project reference, which has TreatAsPackageReference set to false, are also copied in the similar manner and the same rules apply.

If you wish to prevent copying of a content from another project into your nuget package, you can do something like:

     <Content Include="..\..\project2\readme.txt">
         <Pack>false</Pack>
     </Content>

Similarly, you can override the behavior in the referenced project and include a file to be packed which would have otherwise been excluded: 
     
     <Content Include="..\..\project2\readme.txt">
         <Pack>true</Pack>
         <PackagePath>content\myfiles\</PackagePath>
         <Visible>false</Visible>
     </Content>

Setting visible to false prevents VS from showing the file in the Solution Explorer.

There is also a new MSBuild property $(IncludeContentInPack), which defaults to true. If this is set to false on any project, then the content from that project or it's project to project dependencies are not included in the nuget package.

**Apart from Content items, the \<Pack> and \<PackagePath> metadata can also be set on files with Build Action = Compile, EmbeddedResource, ApplicationDefinition, Page, Resource, SplashScreen, DesignData, DesignDataWithDesignTimeCreatableTypes, CodeAnalysisDictionary, AndroidAsset, AndroidResource, BundleResource or None.**

**Note that for pack to append the filename to your package path, your package path must end with the directory separator character, otherwise the package path is treated as the full path including the file name.**

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
Same as IncludeSymbols, except that it copies source files along with pdbs as well. All files of type = Compile are copied over to src\\\<ProjectName>\ preserving the relative path directory structure in the resulting nupkg. The same also happens for source files of any ProjectReference which has \<TreatAsPackageReference> set to false.

If a file of type = Compile, is outside the project folder, then it is just added to src\\\<ProjectName>\\.

####IsTool
If msbuild /t:pack /p:IsTool=true, all output files, as specified in the Output Assemblies scenario, are copied to the tools folder instead of the lib folder. Note that this is different from a DotNetCliTool which is specified by setting the PackageType in csproj file.

####Packing using a nuspec
You can use a nuspec file to pack your project, however, you still need to have a project file to import NuGet.Build.Tasks.Pack.targets so that the pack task can be executed.
The following three msbuild properties are relevant to packing using a nuspec :

1. **NuspecFile** -> relative or absolute path to the nuspec file being used for packing

2. **NuspecProperties** -> semicolon separated list of key=value pairs. Due to the way msbuild command line parsing works, if there is more than one property, you need to specify something like this ```/p:NuspecProperties=\"key1=value1;key2=value2\" ``` 

3. **NuspecBasePath** -> BasePath for the nuspec file.

If using dotnet.exe to pack your project, use a command-line like :

``` dotnet pack <path to csproj file> /p:NuspecFile=<path to nuspec file> /p:NuspecProperties=<> /p:NuspecBasePath=<Base path>```

If using msbuild to pack your project, use a command-line like :

``` msbuild /t:pack <path to csproj file> /p:NuspecFile=<path to nuspec file> /p:NuspecProperties=<> /p:NuspecBasePath=<Base path>```