This in the minimum set of core features required to support multi-platform and bait-and-switch packages.

## Virtual Packages

A `GetPackageContents` target is provided (via the `NuGet.Build.Packaging` nuget targets and eventually 
via MSBuild global imports to all projects that use the Common targets).

This target returns an item list containing the entire contents of a "virtual package", the exact contents 
that the project would contribute to a `.nupkg` if one were to be built. The type of item being contributed 
can be determined by inspecting the `%(Kind)` metadata, which can be:

Kind| Description
--- | --- 
Lib |The assembly, symbols (.pdb) and xml documentation files that end up under the `lib` folder inside the package.
Metadata|The package metadata, defined in project properties.
Dependency|The package dependencies.
Build, Tools, ContentFiles, Native, Runtimes, Ref, Analyzers, Source|The files in the various [known folders](https://github.com/NuGet/NuGet.Client/blob/dev/src/NuGet.Core/NuGet.Packaging/PackagingConstants.cs#L19-L27) in a package. <br/>TBD: maybe we should remove these from the doc for now? Not clear how some are used currently?

Common metadata for these items that can be used to analyze how they contribute to a package:

Common Metadata| Description
--- | --- 
`%(PackageId)`|The project's `$(PackageId)`, if any.
`%(PackagePath)`|If the item represents actual file in the package, its relative path inside it (i.e. `lib\net45\foo.dll`). 
`%(TargetFrameworkMoniker)`|The TFM of the project contributing the file (i.e. `.NETFramework,Version=v4.5`).
`%(TargetFramework)`|The short framework name, such as `net45`


The `GetPackageContents` target depends on the `GetPackageContentsDependsOn` property, allowing other targets 
to inject information into the virtual package. It's also possible to just create a new target and specify 
`BeforeTargets="GetPackageContents"` instead, to contribute additional `PackageFile` items or even remove 
inferred items.

The default package contents inference behavior will automatically discover some default `PackageFile` items 
too (see [Package Files](#package-files)) before `GetPackageContents` runs. The inference is done in the `InferPackageContents`, which you can also extend with `BeforeTargets` or `AfterTargets` to affect the inference.

The virtual package is returned regardless of whether the project is actually configured to build a package. 
If the project is not configured to build a package, no metadata item will be included. It is not only used 
when creating nupkgs, it also allows projects to inspect package metadata and items from the projects that 
they reference.

> NOTE: The `$(IsPackable)` property can be used in conditions to determine if a project generates an actual
> nupkg. This property is 'true' if the project defines a 'PackageId'. The built-in 
> [GetTargetPathWithTargetPlatformMoniker](https://github.com/Microsoft/msbuild/blob/30c9fbc/src/XMakeTasks/Microsoft.Common.CurrentVersion.targets#L1756-L1759) target is [extended](https://github.com/NuGet/NuGet.Build.Packaging/blob/4fbc5fe/src/Build/NuGet.Build.Packaging.Tasks/NuGet.Build.Packaging.targets#L47-L53) to include this metadata as well as an `IsNuGetized=true` metadata value, so it's easy for MSBuild targets to 
> determine if a project reference produces a nuget package or not, and if it has been 'nugetized' or not, without 
> risking invoking non-existing targets (i.e. 'GetPackageContents').

### Metadata

Metadata is specified as [project properties](https://github.com/NuGet/Home/wiki/Adding-nuget-pack-as-a-msbuild-target#pack-target-inputs). This metadata can be retrieved from a project in two ways:

* Via the `GetPackageTargetPath` target, which gets just the metadata item, and is analogous to the `GetTargetPath`.
* Alternatively, via the `GetPackageContents`, by inspecting the item with the `%(Kind)='Metadata'` metadata.

Note that project properties sometimes have the *Package* prefix to disambiguate values, such as `$(PackageRequireLicenseAcceptance)`, `$(PackageLicenseUrl), `$(PackageProjectUrl)` and $(PackageTags). 
The manifest metadata in retrieved by either of the above ways never contains the *Package* since 
there is no need to disambiguate its purpose (i.e. `PackageLicenseUrl` becomes `LicenseUrl` and so on).

#### Package Version

The version of the package to build can be specified as the `PackageVersion` project property. 

It is pretty common for this value to be more dynamic, so you can either redefine the `GetPackageVersion` 
target or add your target to the `GetPackageVersionDependsOn` to populate the `$(PackageVersion)` 
property. The NuGetizer itself uses this mechanism to version its package leveraging Git information.

The default behavior of the built-in `GetPackageVersion` target is to set `$(PackageVersion)` to 
`$(Version)` if it's empty by the time the target runs, which is the same property used by [.NET Core]
(https://github.com/dotnet/sdk/blob/master/src/Tasks/Microsoft.NET.Build.Tasks/build/Microsoft.NET.GenerateAssemblyInfo.targets#L120-L122).


### Package Files
 
A `PackageFile` item type allows projects and targets to add arbitrary files to the virtual package at the 
package-relative location specified by the `PackagePath` item metadata. These should not generally need to be 
used directly in project files unless you want full control of what goes into the package (see #package-file-inference below). Platform-specific targets should add targets to `GetPackageContentsDependsOn` (or are just declared with `BeforeTargets="GetPackageContents"`) that synthesize `PackageFile` items as appropriate from platform-specific items such as `Content` and `BundleResource`. 

You can see a concrete example of this usage in the NuGetizer [build tasks project](https://github.com/NuGet/NuGet.Build.Packaging/blob/bc03917/src/Build/NuGet.Build.Packaging.Tasks/NuGet.Build.Packaging.Tasks.targets#L29) itself, in the `AddBuiltOutput` target.

For example, the iOS build targets might do the following, so that a library's `BundleResource` items would end up in any referencing projects.

```xml
<PackageFile Include="@(_BundleResourceWithOutputPath)">
    <Kind>Content</Kind>
    <TargetPath>%(OutputPath)</TargetPath>
    <BuildAction>BundleResource</BuildAction>
</PackageFile>
```

`PackageFile` items that have `Kind=Content` may also specify `BuildAction`, `CopyToOutput` and `Flatten` 
metadata, which map to the corresponding NuGet `contentFiles` [nuspec attributes](http://docs.nuget.org/ndocs/schema/nuspec#contentfiles-with-visual-studio-2015-update-1-and-later).
Additionally, a `CodeLanguage` and `TargetFramework` metadata can be provided to restrict the language and 
target framework of the consuming projects to add it to, such as:

```xml
<PackageFile Include="Samples\ApiExample.cs">
    <Kind>Content</Kind>
    <CodeLanguage>cs</CodeLanguage>
    <TargetFramework>any</TargetFramework>
</PackageFile>
```

This file will end up in `/contentFiles/cs/any/Samples/ApiExample.cs`.

By default, `CodeLanguage` is set to `any`, and `TargetFramework` is set to the containing project's 
`TargetFramework`.

### Package File Inference

Obviously, having to specify manually everything that should go into a package would be cumbersome, 
so NuGetizer provides some default inference behavior that synthesize `PackageFile` items using the 
following heuristics: 

- If no `$(PrimaryOutputKind)` is specified, it is assumed to be `Lib` (therefore, current project 
  target framework-specific)
- If `$(IncludeOutputsInPackage)` is not set to `false`, the virtual package will include .NET libraries 
  (from `BuiltProjectOutputGroup`) and their symbols (from `DebugSymbolsProjectOutputGroup`), xml docs 
  (from `DocumentationProjectOutputGroup`) and satellite assemblies (from `SatelliteDllsProjectOutputGroup`) 
  in the package, with the same `%(Kind)` as `$(PrimaryOutputKind)` 
- If `$(IncludeFrameworkReferencesInPackage)` is not `false` (defaults to `true` for `$(PrimaryOutputKind)="Lib"`), 
  framework references (from `'%(ReferencePath.ResolvedFrom)' == '{TargetFrameworkDirectory}'`) are added as 
  framework dependencies.
- `@(None)` and `@(Content)` with a `CopyToOutputDirectory` value (both `PreserveNewest` as well as `Always`) are 
  included relative to the primary output too (i.e. if `$(PrimaryOutputKind)="Lib"` they will be relative to the 
  framework-specific `lib` folder inside the package).
- `@(Content)` items without a `CopyToOutputDirectory` value are included as `%(Kind)="Content"` and placed relative to 
  the `contentFiles` special folder. More details on `contentFiles` below.
- `@(None)` items without a `CopyToOutputDirectory` metadata are included as `%(Kind)="None"` and placed relative to the package root directory only if the `$(IncludeNoneInPackage)` property is set to 'true'. The preferred way to achieve this 
  same behavior is to simply use `PackageFile` instead, however.

`Content` inference can be turned off with `$(IncludeContentInPackage)`. As mentioned, `None` inference when 
via the `$(IncludeNoneInPackage)` is `false` by default.

You can opt out of inference for specific items by adding `%(Pack)="false"` item metadata, such as:

```
<Content Include="...">
  <Pack>false</Pack>
</Content>
```

Ultimately, all these built-in supported item types are turned into `PackageFile` items, so for `Content` items, the 
`CodeLanguage`, `TargetFramework`, `BuildAction`, `CopyToOutput` and `Flatten` metadata values behave as explained in the previous section.


#### PackagePath vs TargetPath

When a `PackageFile` provides a `PackagePath`, that is the final relative path within the `nupkg` where the file 
will end up and no further processing is performed for them by the `AssignPackagePath` task. 

If is quite common, however, to want the files to end up in whatever the right framework directory is depending on 
the current project's TFM. For example, if you're providing a localized XML intellisense file, you would want it to 
end up in the right `lib\[TFM]` root folder, under a relative path of (say) `es-AR`. You can achieve this with the `TargetPath` 
as follows:

```xml
<PackageFile Include="MyLibrary.es-AR.xml">
    <Kind>Lib</Kind>
    <TargetPath>es-AR\MyLibrary.xml</TargetPath>
</PackageFile>
```

If the project being built is a NETStandard 1.6 one, this file will end up in `/lib/netstandard1.6/es-ar/MyLibrary.xml`.
If you later decide to retarget the project to NETStandard 2.0, no changes will be necessary to the `PackageFile` 
declaration above.


### Dependencies

The virtual package will implicitly depend on the project's dependencies:

* NuGet packages referenced directly via project.json or `PackageReference` items.

* The project's project references ("P2P").

Project references can be excluded from the virtual package by setting `Pack=false` metadata, 
which is analogous to [ReferenceOutputAssembly](https://blogs.msdn.microsoft.com/kirillosenkov/2015/04/04/how-to-have-a-project-reference-without-referencing-the-actual-binary/):

```xml
<ProjectReference Include="..\OtherProject\OtherProject.csproj">
    <Pack>false</Pack>
```

When generating its virtual package, a referencing project will call the `GetPackageContents` target on each 
of its project references that have not been excluded from the package. If the referenced project reference as 
package metadata, it will be added to the project's own virtual package as a package dependency. Otherwise, its
outputs are merged into the referencing project's own virtual package.

### Package Merging

When merging a referenced virtual package into a referencing virtual package, the package's files will be added 
directly to the merged package, with local paths adjusted appropriately.

The referenced package's dependencies will merged into the referencing package. They will be treated as specific 
to that package's supported frameworks, however they will be re-unified into non-framework-specific dependencies 
where possible.

## Package Creation

A `Pack` build target is provided, which will use the virtual package to build an actual nupkg file in the 
project's output directory. The `$(PackageOutputPath)` property can be specified to override this default.

A `PackOnBuild=true` property causes the `Pack` target to be added as a `BuildDependsOn` post-build step to 
allow building all NuGets in a solution via `msbuild Foo.sln /p:PackOnBuild=true`.


## Package Projects

A new project type will be introduced, a _Package Project_, with its own build targets.

The package project targets consumes the same targets as regular projects. Package projects will also define 
`Build`, `Clean` and `Rebuild` targets for compatibility with the solution-level targets, however these will be empty.

The only items supported by the package project targets are `PackageReference`, `PackageFile` and `ProjectReference`.

> NOTE: `Content` files inference is also support like in regular libraries.

`PackageReference`and `PackageFile` allow using package projects to create nupkgs with full control over content and dependencies.

### Multi-platform packages

The primary use case of packaging projects is for creating multi-platform projects. The packaging project would reference several projects that build a library with the same name, target different frameworks, and do not have package metadata.

Since the referenced packages did not have metadata, their virtual packages would be merged into the packaging project. These virtual packages would have the binaries and content in platform specific directories, and the packaging project would else up with all of them in one package.

### Metapackages

A packaging project can also be used to create metapackages. In this case, it would reference several projects in the solution that have packaging metadata, and thus would build an empty package with dependencies on the packages built by those projects.

## Reference Assembly Generation

Reference assemblies can be generated for one or more Portable Class Library profiles. This is done by adding a ReferenceAssemblyFramework item into the NuGet packaging project.

```xml
  <ItemGroup>
    <ReferenceAssemblyFramework Include=".NETPortable,Version=v4.5,Profile=Profile259" />
  </ItemGroup>
```

On building the NuGet packaging project a reference assembly will be generated based on the intersection of the assemblies that the NuGet packaging project references.

The reference assembly generation process can be further configured.

### Specifying assembly reference paths

If the reference assembly generation process since it cannot resolve an assembly the path to the assembly can be specified as a property in the NuGet packaging project.

```xml
  <PropertyGroup>
      <IntersectionAssemblyReferencePath>/Library/Frameworks/Xamarin.Android.framework/Versions/Current/lib/xbuild-frameworks/MonoAndroid/v7.0;/Some/Other/Path</IntersectionAssemblyReferencePath>
  </PropertyGroup>
```

The IntersectionAssemblyReferencePath property accepts a semi-colon delimited list of paths for assemblies to be resolved.

By default the output directory of the projects referenced by the NuGet packaging project will be used as reference paths. To disable this the EnableDefaultIntersectionAssemblyReferencePath property can be set to false.

```xml
<PropertyGroup>
  <EnableDefaultIntersectionAssemblyReferencePath>false</EnableDefaultIntersectionAssemblyReferencePath>
</PropertyGroup>
```

### Excluding Types

A type can be excluded from the generated reference assembly by specifying the type name in the IntersectionExcludeType property.

```xml
<PropertyGroup>
  <IntersectionExcludeType>FullTypeName1;FullTypeName2</IntersectionExcludeType>
</PropertyGroup>
```

### Removing Abstract Type Members

The IntersectionRemoveAbstractTypeMembers property can be used to remove abstract members from a type.

```xml
<PropertyGroup>
  <IntersectionRemoveAbstractTypeMembers>Type1;Type2</IntersectionRemoveAbstractTypeMembers>
</PropertyGroup>
```

### Excluding Assemblies

To exclude an assembly when generating a reference assembly the IntersectionExcludeAssembly item can be used.

```xml
<ItemGroup>
  <IntersectionExcludeAssembly Include="path/to/assembly.dll" />
</ItemGroup>
```

## IDE Support

A new project flavor will be added for package projects. It will support project references, package references, and files. Building this project type causes a package to be created.

A _Create NuGet Package_ command is available in the solution tree on solutions and all projects. When it's run for 
a solution, it will set the `PackOnBuild=true` property and build the solution normally, causing all configured 
packages to be created. When run for a project, it will first show a dialog with the package metadata (to allow modifications prior to generating the package).

A project template will be added for creating package projects. When run, it will show a dialog prompting the user to provide basic metadata and to choose which frameworks they would like to support.

A project options page will be available where the PackOnBuild property can be enabled or disabled for a project. This will update the PackOnBuild setting in the project file.