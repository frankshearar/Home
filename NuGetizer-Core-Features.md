This in the minimum set of core features required to support multi-platform and bait-and-switch packages.

## Virtual Packages

A `GetPackageContents` target is provided (via the `NuGet.Build.Packaging` nuget targets and eventually 
via MSBuild global imports to all projects that use the Common targets).

This target returns an item list containing the entire contents of a "virtual package", the exact contents 
that the project would contribute to a `.nupkg` if one were to be built. The type of item being contributed 
can be determined by inspecting the `%(Kind)` metadata, which can be:

Kind| Description
--- | --- 
Lib, Symbols, Doc|The assembly, symbols (.pdb) and xml documentation files that end up under the `lib` folder inside the package.
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
to inject information into the virtual package.

The virtual package is returned regardless of whether the project is actually configured to build a package. 
If the project is not configured to build a package, no metadata item will be included. It is not only used 
when creating nupkgs, it also allows projects to inspect package metadata and items from the projects that 
they reference.

### Metadata

Metadata is specified as [project properties](https://github.com/NuGet/Home/wiki/Adding-nuget-pack-as-a-msbuild-target#pack-target-inputs). This metadata can be retrieved from a project in two ways:

* Via the `GetPackageContents`, by inspecting the item with the `%(Kind)='Metadata'` metadata.
* Via the `GetPackageTargetPath` target, which gets just the metadata item, and is analogous to the `GetTargetPath`.

Note that project properties sometimes have the *Package* prefix to disambiguate values, such as `$(PackageRequireLicenseAcceptance)`, `$(PackageLicenseUrl), `$(PackageProjectUrl)` and $(PackageTags). 
The manifest metadata in retrieved by either of the above ways never contains the *Package* since 
there is no need to disambiguate its purpose (i.e. `PackageLicenseUrl` becomes `LicenseUrl` and so on).

#### Package Version

The version of the package to build can be specified as the `PackageVersion` project property. 

It is pretty common for this value to be more dynamic, so you can either redefine the `GetPackageVersion` 
target or add your target to the `GetPackageVersionDependsOn` to populate the `$(PackageVersion)` 
property. The NuGetizer itself [uses this mechanism](https://github.com/NuGet/NuGet.Build.Packaging/blob/dev/src/Build/NuGet.Build.Packaging.Tasks/NuGet.Build.Packaging.Tasks.targets) to version its package leveraging Git information.

The default behavior of the built-in `GetPackageVersion` target is to set `$(PackageVersion)` to 
`$(Version)` if it's empty by the time the target runs. 


### Package Files

By default, the virtual package will include .NET libraries, xml docs, satellite assemblies and framework references, 
however platform-specific targets can add additional platform-specific files.

To opt out of these defaults, set `$(IncludeOutputs)`, `$(IncludeSymbols)` and `$(IncludeFrameworkReferences)` to 
`false` as needed.


A `PackageFile` item type allows projects and targets to add arbitrary files to the virtual package at the 
package-relative location specified by the `PackagePath` item metadata. These should not generally need to be 
used directly in project files. Platform-specific targets should add targets to `GetPackageContentsDependsOn` 
that synthesizes `PackageFile` items as appropriate from platform-specific items such as `Content` and `BundleResource`. 
You can see a concrete example of this usage in the NuGetizer [build tasks project]https://github.com/NuGet/NuGet.Build.Packaging/blob/dev/src/Build/NuGet.Build.Packaging.Tasks/NuGet.Build.Packaging.Tasks.targets) itself, in the `AddBuiltOutput` target.

For example, the iOS build targets might do the following, so that the a library's `BundleResource` items would end up in any referencing projects.

```xml
<PackageFile Include="@(_BundleResourceWithOutputPath)">
    <Kind>Content</Kind>
    <TargetPath>%(OutputPath)</TargetPath>
    <BuildAction>BundleResource</BuildAction>
</PackageFile>
```

`PackageFile` items that have `Kind=Content` may also specify `BuildAction`, `CopyToOutput` and `Flatten` 
metadata, which map to the corresponding NuGet `contentFiles` [nuspec attributes](http://docs.nuget.org/ndocs/schema/nuspec#contentfiles-with-visual-studio-2015-update-1-and-later).
Additionally, a `CodeLanguage` metadata can be provided to restrict the language of the consuming projects 
to add it to, such as:

```xml
<PackageFile Include="ApiExample.cs">
    <Kind>Content</Kind>
    <CodeLanguage>cs</CodeLanguage>
    <TargetPath>Samples\%(Filename)%(Extension)</TargetPath>
</PackageFile>
```

This file will end up in `/contentFiles/cs/any/Samples/ApiExample.cs`.

By default, `CodeLanguage` and `TargetFramework` are considered `any`.

#### PackagePath vs TargetPath

When a `PackageFile` provides a `PackagePath`, that is the final relative path within the `nupkg` where the file 
will end up. 

If is quite common, however, to want the files to end up in whatever the right framework directory is for the kind 
of file. For example, if you're providing a localized XML intellisense file, you would want it to end up in the 
right `lib\[TFM]` root folder, under a relative path of (say) `es-AR`. You can achieve this with the `TargetPath` 
as follows:

```xml
<PackageFile Include="MyLibrary.es-AR.xml">
    <Kind>Doc</Kind>
    <TargetPath>es-AR\MyLibrary.xml</TargetPath>
</PackageFile>
```

If the project being built is a NETStandard 1.6 one, this file will end up in `/lib/netstandard1.6/es-ar/MyLibrary.xml`.


### Dependencies

The virtual package will implicitly depend on the project's dependencies:

* NuGet packages referenced directly via project.json or `PackageReference` items.

> NOTE: packages.config contains the closure of all dependencies. Typically not what you want 
> your top level packages to declare dependencies on. So it may be better to not support it 
> at all.

* The project's project references ("P2P").

Project references can be excluded from the virtual package using the `ReferenceOutputPackage` metadata, 
which is analogous to [ReferenceOutputAssembly](https://blogs.msdn.microsoft.com/kirillosenkov/2015/04/04/how-to-have-a-project-reference-without-referencing-the-actual-binary/):

```xml
<ProjectReference Include="..\OtherProject\OtherProject.csproj">
    <ReferenceOutputPackage>false</ReferenceOutputPackage>
```

When generating its virtual package, a referencing project will call the `GetPackageContents` target on each 
of its project references that have not been excluded from the package. If the referenced virtual package has 
metadata, it will be added to the project's own virtual package as a package dependency. If the referenced virtual 
package does not have metadata, will will be merged into the project's own virtual package.

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

**TBD:** replace this with a solution level target?

## Package Projects

A new project type will be introduced, a _Package Project_, with its own build targets.

The package project targets consumes the same targets as regular projects. Package projects will also define 
`Build`, `Clean` and `Rebuild` targets for compatibility with the solution-level targets, however these will be empty.

The only items supported by the package project targets will be `PackageReference`, `PackageFile` and `ProjectReference`.

**TBD:** can easily support project.json NuGet packages too.

`PackageReference`and `PackageFile` allow using package projects to create nupkgs with full control over content and dependencies.

### Multi-platform packages

The primary use case of packaging projects is for creating multi-platform projects. The packaging project would reference several projects that build a library with the same name, target different frameworks, and do not have package metadata.

Since the referenced packages did not have metadata, their virtual packages would be merged into the packaging project. These virtual packages would have the binaries and content in platform specific directories, and the packaging project would else up with all of them in one package.

### Metapackages

A packaging project can also be used to create metapackages. In this case, it would reference several projects in the solution that have packaging metadata, and thus would build an empty package with dependencies on the packages built by those projects.

## IDE Support

A new project flavor will be added for package projects. It will support project references, package references, and files.

Package projects and library projects will have a project options panel that allows setting the NuGet metadata.

**TBD:** the only way we can add project options to library projects is by flavoring them "on the fly" on first use of _Create NuGet Package_ context menu.


A _Create NuGet Package_ command will be available in the solution tree on solutions and all projects that have package metadata.. It will run the `Pack` target to allow building the package project.

A project template will be added for creating package projects. When run, it will show a dialog prompting the user to provide basic metadata and to choose which frameworks they would like to support.
