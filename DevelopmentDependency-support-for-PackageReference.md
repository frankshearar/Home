This spec is about how are we going to support developmentDependency for projects which use PackageReference to manage their NuGet dependencies. This developmentDependency flag is already supported for packages.config based projects.

## Issue
https://github.com/NuGet/Home/issues/4125

## Problem
There are couple of issues because PackageReference doesn't support DevelopmentDependency flag defined in package's nuspec file, as follow
* Package assets are available at runtime even though it's only meant for build time.
* Package is flowing transitively to parent projects.
* Packing a project which has a developmentDependency package, also include this package as dependency.

## Who is the customer?
Every Visual Studio customer not using packages.config based projects. We want to get users to move away from the dependency management hell some of them are finding themselves in. Future investments will be primarily on top of new standards like PackageReference and we want to bring all our customers in for a ride. Currently, consumers have to explicitly set some metadata on PackageReference to achieve the right behavior. Even discovering what metadata to set isn't easy so consumers and authors both will be hugely benefited by this change.

## Solution
We need to respect developmentDependency flag set in package's nuspec file, similar to how we respect it for packages.config based projects. But there will be slight difference how to handle this flag in different scenario which is described below:
* when package's developmentDependency is true, then set PrivateAssets to All and ExcludeAssets to Compile on PackageReference item in project, while
  * Installing package through VS NuGet UI or PMC
  * Executing dotnet add package command
  * Migrating existing project from packages.config to PackageReference (future scenario)
* Even though developmentDependency is true, don't do anything treat it as normal PackageReference, while
  * Manually adding PackageReference item in project to add a package
  * Upgrading existing VS instance from previous version to latest (which started supporting developmentDependency)

By having the right metadata on PackageReference, it will make sure that package with developmentDependency flag are treated in the right way and provide the correct behavior to consumers. Having that said, in any case consumers will always be able to override this default behavior by explicitly setting these metadata on PackageReference.

## Example
When installed a package with developmentDependency set to true with `PackageReference` then it will look something like:
```
<PackageReference Include="Foo" Version="1.0.0">
   <PrivateAssets>all</PrivateAssets>
   <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
</PackageReference>
```