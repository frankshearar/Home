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

For part 1, NuGet will introduce a new folder structure called `/buildTransitive` similar to existing `/build` folder except by default this new folder will be transitive in nature. So any build assets inside this folder will transitively flow to any consuming project. There will also be a new [IncludeType](https://docs.microsoft.com/en-us/nuget/consume-packages/package-references-in-project-files) flag called `buildTransitive` which will control the flow of this new assets group just like existing `build` flag does for existing `build` assets group. So if any package author wants to restrict `/buildTransitive` assets from it's dependencies then he can exclude `buildTransitive` from that PackageReference. `buildTransitive` flag will internally also exclude `build` flag to maintain the consistency across both folders.

To construct a package which allows build assets to flow transitively, package author will need to put all these build assets in `/buildTransitive` as well as `/build` folder to make the package compatible with `packages.config`. 

### Scenarios to be considered

#### 1. PackageReference with transitive dependencies to allow flowing build assets

Project A has a PackageReference to Package B  
Package B has a dependency on Package C  
Package C has a build assets (`C.targets`) which is added under `/build` as well as `/buildTransitive` folders  

Project A should be able to consume C.targets from Package C  

#### 2. PackageReference with transitive dependencies to deny flowing build assets

Project A has a PackageReference to Package B  
Package B has a dependency on Package C with `PrivateAssets=buildTransitive`  
Package C has a build assets (`C.targets`) which is added under `/build` as well as `/buildTransitive` folders  

Project A should not be able to consume C.targets from Package C  

#### 3. ProjectReference with transitive dependencies to allow flowing build assets

Project A has a ProjectReference to Project B  
Proejct B has a dependency on Package C  
Package C has a build assets (`C.targets`) which is added under `/build` as well as `/buildTransitive` folders  

Project A should be able to consume C.targets from Package C  

#### 4. ProjectReference with transitive dependencies to deny flowing build assets

Project A has a ProjectReference to Project B  
Proejct B has a dependency on Package C with `PrivateAssets=buildTransitive`  
Package C has a build assets (`C.targets`) which is added under `/build` as well as `/buildTransitive` folders  

Project A should not be able to consume C.targets from Package C  

#### 5. Multiple PackageReference with different transitive behavior

Package A has a build assets (`A.targets`) which is added under `/build` as well as `/buildTransitive` folders  
Package B has a build assets (`B.targets`) which is only added under `/build` folder  
Package C has a dependency on Package A as well as Package B  
Package D has a dependency on Package C  
Project E has a PackageReference to Package D  

Project E should be able to consume A.targets but not B.targets
