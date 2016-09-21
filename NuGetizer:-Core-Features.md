This in the minimum set of core features required to support multi-platform and bait-and-switch packages.

## Virtual Packages

A `GetNuGetInfo` target will be added (via MSBuild global imports) to all projects that use the Common targets. It is a analogous to the existing `GetTargetPath` target.

This target returns the path to a "virtual package", a partial `.nuspec` file generated in the intermediate output directory that contains the following information:

* The package metadata, defined in project properties.
* The package dependencies.
* The files in the package.

The `GetNuGetInfo` target will depend on the `GetNuGetInfoDependsOn` property, allowing other targets to inject information into the virtual package.

The virtual package is returned regardless of whether the project is actually configured to build a package. If the project is not configured to build a package, the metadata in the virtual package is empty. It is not only used when creating nupkgs, it also allows projects to inspect package metadata and items from the projects that they reference.

### Metadata

Naming TBD. See also [here](https://github.com/NuGet/Home/wiki/Adding-nuget-pack-as-a-msbuild-target#solution).

### Package Files

By default, the virtual package will include .NET libraries, xml docs and satellite assemblies, however platform-specific targets can add additional platform-specific files.

A `NuGetFile` item type will allow projects and targets to add arbitrary files to the virtual package at the package-relative location specified by the `LogicalName` item metadata. These should not generally need to be used directly in project files. Platform-specific targets should add targets to `GetNuGetInfoDependsOn` that synthesizes `NuGetFile` items as appropriate from platform-specific items such as `Content` and `BundleResource`.

For example, the iOS build targets might do the following, so that the a library's `BundleResource` items would end up in any referencing projects.

```xml
<NuGetFile Include="@(_BundleResourceWithOutputPath)"
           LogicalName="%(OutputPath)"
           BuildAction="BundleResource" />
```

`NuGetFile` items that are in the `contentFiles` package path may also specify `BuildAction`, `CopyToOutput` and `Flatten` metadata, which map to the corresponding NuGet `contentFiles` nuspec attributes.

### Dependencies

The virtual package will implicitly depend on the project's dependencies:

* NuGet packages referenced directly via packages.config, project.json or `PackageReference` items.
* The project's project references ("P2P").

Project references can be excluded from the virtual package using the `ExcludeFromPackage` metadata

```xml
<ProjectReference Include="..\OtherProject\OtherProject.csproj"
                  ExcludeFromPackage="true" />
```

When generating its virtual package, a referencing project will call the `GetNuGetInfo` target on each of its project references that have not been excluded from the package. If the referenced virtual package has metadata, it will be added to the project's own virtual package as a package dependency. If the referenced virtual package does not have metadata, will will be merged into the project's own virtual package.

### Package Merging

When merging a referenced virtual package into a referencing virtual package, the package's files will be added directly to the merged package, with local paths adjusted appropriately.

The referenced package's dependencies will merged into the referencing package. They will be treated as specific to that package's supported frameworks, however they will be re-unified into non-framework-specific dependencies where possible.

## Package Creation

A `BuildNuGet` build target will be added (via MSBuild global imports) to all projects that use the Common targets. This will use the virtual package to build an actual nupkg file in the project's output directory.

**TBD:** call this `Package` instead?

A `BuildNuGet` property will include the `BuildNuGet` target as a `BuildDependsOn` post-build step to allow building all NuGets in a solution via `msbuild Foo.sln /p:BuildNuGet=true`.

**TBD:** replace this with a solution level target?

## Package Projects

A new project type will be introduced, a _Package Project_, with its own build targets.

The package project targets will support the `BuildNuGet` and `GetNuGetInfo` targets. Package projects will also define `Build`, `Clean` and `Rebuild` targets for compatibility with the solution-level targets, however these will be empty.

The only items supported by the package project targets will be `PackageReference`, `NuGetFile` and `ProjectReference`.

`PackageReference`and `NuGetFile` allow using package projects to create nupkgs with full control over content and dependencies.

### Multi-platform packages

The primary use case of packaging projects is for creating multi-platform projects. The packaging project would reference several projects that build a library with the same name, target different frameworks, and do not have package metadata.

Since the referenced packages did not have metadata, their virtual packages would be merged into the packaging project. These virtual packages would have the binaries and content in platform specific directories, and the packaging project would else up with all of them in one package.

### Metapackages

A packaging project can also be used to create metapackages. In this case, it would reference several projects in the solution that have packaging metadata, and thus would build an empty package with dependencies on the packages built by those projects.

## IDE Support

A new project flavor will be added for package projects. It will support project references, package references, and files.

Package projects and library projects will have a project options panel that allows setting the NuGet metadata.

A _Create NuGet Package_ command will be available in the solution tree on solutions and all projects that have package metadata.. It will run the `BuildNuGet` target to allow building the package project.

A project template will be added for creating package projects. When run, it will show a dialog prompting the user to provide basic metadata and to choose which frameworks they would like to support.
