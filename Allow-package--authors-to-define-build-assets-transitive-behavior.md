# Package's build assets flowing transitive with PackageReference

* Status: **Reviewing**
* Author(s): [Ashish Jain](https://github.com/jainaashish)

## Issue

[6091](https://github.com/NuGet/Home/issues/6091) - Change the default behavior of build dependency asset to transitive

## Problem

As of today, by default build assets (.targets and .props) from NuGet packages don't flow transitively into consuming project which sometimes breaks an application if it needs those build assets from down level package dependencies. For example Project A depends on Package B which in turn depends on Package C. Package C is available in Project A through Package B but it's build assets are not so Project A may still compile successfully but will fail at runtime. And it will hard for Project A to understand the root cause. 

So neither package author can say which build assets will flow transitively nor package consumer can change that without adding a direct reference to the corresponding package.

## Who are the Customers

All NuGet users who uses PackageReference to manage their NuGet dependencies and wants to consume or produce packages with build assets.

## Solution

There will be two parts of the solution. 1) This will focus on allowing package authors to say whether their package's build assets should flow transitively or not. 2) This will allow package consumers to override package authors behavior for build assets without making it a direct reference which will resolve this issue for existing packages as well. This Spec will focus on the first part which is more critical going forward and then there will be a separate Spec for part 2.

For part 1, NuGet will introduce a new metadata in nuspec file called `AllowBuildAssetsTrabsitive` which when set to `true` will allow flowing build assets from that package transitively to any consuming project. 

For PackageReference transitive dependencies, this will be achieved by changing the default value of `Exclude` metadata from `build,analyzers` to `analyzers` on `Dependency` element in nuspec file.

For ProjectReference transitive dependencies, this will be achieved by changing the default value of `PrivateAssets` from `contentfiles;analyzers;build` to `contentFiles;analyzers` on PackageReference element in project file.

This is about changing the default experience of these `Exclude` or `PrivateAssets` when corresponding package has set `AllowBuildAssetsTransitive` but nothing will change if this property is not set.

### Scenarios to be considered

#### 1. PackageReference with transitive dependencies to allow flowing build assets

Project A has a PackageReference to Package B </br>
Package B has a dependency on Package C </br>
Package C has a build assets (`C.targets`) and Set `AllowBuildAssetsTransitive` to `true` </br>

Project A should be able to consume C.targets from Package C

#### 2. PackageReference with transitive dependencies to deny flowing build assets

Project A has a PackageReference to Package B </br>
Package B has a dependency on Package C with `PrivateAssets=build` </br>
Package C has a build assets (`C.targets`) and Set `AllowBuildAssetsTransitive` to `true` </br>

Project A should not be able to consume C.targets from Package C

#### 3. ProjectReference with transitive dependencies to allow flowing build assets

Project A has a ProjectReference to Project B </br>
Proejct B has a dependency on Package C </br>
Package C has a build assets (`C.targets`) and Set `AllowBuildAssetsTransitive` to `true` </br>

Project A should be able to consume C.targets from Package C

#### 4. ProjectReference with transitive dependencies to deny flowing build assets

Project A has a ProjectReference to Project B </br>
Proejct B has a dependency on Package C with `PrivateAssets=build` </br>
Package C has a build assets (`C.targets`) and Set `AllowBuildAssetsTransitive` to `true` </br>

Project A should not be able to consume C.targets from Package C

#### 5. Multiple PackageReference with different transitive behavior

Package A has a build assets (`A.targets`) and Set `AllowBuildAssetsTransitive` to `true` </br>
Package B has a build assets (`B.targets`) </br>
Package C has a dependency on Package A as well as Package B </br>
Package D has a dependency on Package C </br>
Project E has a PackageReference to Package D </br>

Project E should be able to consume A.targets but not B.targets
