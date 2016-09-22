The following features are proposed to be added at a later time, and have been taken into account in the design of the core features.

## Synthetic Reference Assemblies

A `SyntheticReferenceAssembly` MSBuild item type in package projects will allow specifying a TFM for which to generate a reference assembly automatically:

```xml
<SyntheticReferenceAssembly Include=”netstandard1.6”/>
```

These would be edited using a project options panel.

When the package project is built, it will generate a reference assembly from the intersection of all of the framework-specific libraries in the package which are compatible with the specified framework. The resulting reference assembly will be included in the virtual package.

This will mean that users no longer have to write and maintain separate API contracts for bait-and-switch packages.

## Bait-and-Switch Wizard

A project wizard for package projects would allow filling out the package ID and description upfront, and specifying which frameworks to target. It would then create individual platform-specific library projects, sharing code via a shared project. It would also create a package project that referenced all the platform-specific library projects, with a synthetic reference assembly.

## Add Platform Implementation

An _Add Platform Implementation_ context menu on package projects would help to add additional target platforms to existing projects.

When used on a package project, it would inspect the existing project references to detect the existing target platforms and their common shared project, and show a dialog allowing the user to select additional target platforms. For each platform that the user selected, it would create the library project and reference it from the package project, and reference the common shared project from the library project.

## Convert to Package Project

An _Add Platform Implementation_ context menu on library projects would enable an upgrade scenario, bridging the gap between “simple” library projects and “advanced” package projects.

The command would be available on library projects that are configured to produce NuGet package, and would show a dialog allowing the user to select additional target platforms. It would then move the package metadata from the original library project to a new package project, and move the MSBuild items from the original library project to a new shared project. It would then create the additional library projects for the platforms that the user had chosen. The original and newly added library project would all reference the shared project, and the package project would reference all of them.

## Package References

NuGet should be more tightly integrated with MSBuild across all project types by replacing packages.config and project.json with `PackageReference` items.

## Reference Unification

Project references and NuGet references should be unified so that referencing a project works the same way as referencing the NuGet produced by that project. This would work by projects calling the GetNuGetInfo on referenced projects, and consume the virtual package exactly as they would a real package created from it - content files, dependencies, libraries, etc.

This should also support referencing a packaging project from any project that's compatible with it, allowing a solution to "fuse" platforms. For example, a solution might contain a netstandard project A, and a bait and switch package project B that combines iOS and Android projects BI and BI. A should be able to reference B directly; you should not need platform-specific variants of A.

## Transitive References

NuGet dependencies should transparently flow from referenced project to referencing project as they do with xproj. This should work the same regardless whether the references are to packaging projects, library projects that output NuGets, or NuGets in external repositories.

## Content Files

Platform targets should implicitly add NuGet `contentFiles` to the virtual package from the files in the project with the appropriate build actions. This will need to be done on a platform by platform basis.

## Transitive Resources

Android/iOS projects bundle library resources and native libraries into the dll as embedded resources, then unpack them when building the app. This is inefficient and has been the cause of major performance and memory issues. The combination of Content Files and Reference Unification will allow referencing projects to consume them directly and transparently via the virtual package, and they will no longer need to be embedded.

## Tools Projects

There should be an `IsTool` project property on executable and library projects that marks them as as tools projects, so their output would go into the `tools` directory of the NuGet.

## Symbol Packages

There should be a `IncludeSymbols` project property that collects the debug symbols from beside the assemblies when building the package and places them in a symbol package. There should also be an `IncludeSource` project property that includes source files in the symbol package. By default it would include all `Compile` items.

This should be implemented via collecting `PackageSource` and `PackageSymbol` items and integrating them into the virtual package, so that they can be extended by platform targets.

## Publishing

It would be great to have IDE support for publishing the packages to nuget.org and managing release notes.
