Status: **Reviewing**

## Issue
The work for this feature and the discussion around the spec is tracked here:

**Change the default behavior of build dependency asset to transitive [#6091](https://github.com/NuGet/Home/issues/6091)**

## Problem
As already discussed multiple times, in the packages.config world, the packages are installed in a flat-list. Packages can contain build assets which will be imported during build. 
The build folder is a class of assets that's compatible with both packages.config and package reference worlds. 
In the packages.config to Package Reference [migration effort](https://github.com/NuGet/Home/issues/5877), we instruct package authors to use targets to replicate what they are trying to achieve in [install.ps1 scripts](https://github.com/NuGet/Home/issues/5963). 
In the transitive world (Package Reference), dependency flow behavior is controlled via the **PrivateAssets** metadata. The defaults are described [here](https://docs.microsoft.com/en-us/nuget/consume-packages/package-references-in-project-files#controlling-dependency-assets).

All of this means, that packages won't work as expected by the author, when not referenced directly by the project.

## Who is the customer?
All Package Reference customers.

## Key Scenarios

### Packages pulled in through Project-to-Project references
The out of the box experience means that these packages will not have their build assets imported. 
The package consumer can workaround that by changing the **PrivateAssets** metadata, but we believe this should be a scenario that works out of the box. 

### Packages pulled in transitively as a dependency of a package
When a package with dependencies is authored, the package dependencies PrivateAssets value will be embedded in the nuspec and later respected by restore. 
Since most packages are built with build assets as private, the transitive packages do not work as the original package author intended for them to.

## Solution

Change the default value of **PrivateAssets** to not include build. 
Make the build assets flow by default. 

### Packages pulled in through Project-to-Project references
This means that the user will not see a discrepancy in how packages work/build in parent and referenced projects. 
The consumers still retains all control here as they can still change the default behavior.

### Packages pulled in transitively as a dependency of a package
In this scenario, we would impact the package authors.
Likewise the package authors retain control and can change the default behavior.  

## Plan

Since this is a breaking change, we need to provide a switch for consumers to change back the dependency asset behavior.
It's an open question how we do [this](#open-questions). 
The implementation is straightforward 

## Open Questions
- How do we switch the dependency asset value for PrivateAssets?
-- NuGet.Config variable
-- Environment variable
-- Both? Others? Can an MSBuild property set the behavior as well?
- Is this change dependant on Package Reference support for [DevelopmentDependency metadata](https://github.com/NuGet/Home/issues/4125). 