Status: **Incubation**

## Issue
The work for this feature and the discussion around the spec is tracked by 3 issues
* Icon - **Package Icon should be able to come from inside the package [#352](https://github.com/NuGet/Home/issues/352)**
* License - **Package trust - Licenses [#4628](https://github.com/NuGet/Home/issues/4628)**
* Documentation - **Nuspec - documentation / readme url [#6873](https://github.com/NuGet/Home/issues/6873)**

## Problem
* Icons - loading images from external URLs have security and privacy concerns.
* License -  it is possible to update the page contents at the referred license URL raising legal concerns for package consumers. 
* Documentation - it is not possible to update the documentation on the package details page without going to NuGet.org. This information is not surfaced in Visual Studio.

## Who is the customer?
* NuGet package consumers that consume packages using Visual Studio 2017 (and above) package manager UI from NuGet.org (v3 feed) and folder based feeds
* NuGet package authors that push to NuGet.org. 

## Key scenarios
* Display package icons, surface license and documentation information when:
  * Browsing packages in PMUI from NuGet.org (v3 feed) or folder based feeds
  * Viewing installed packages in VS PMUI
* nuget pack validations to limit usage of deprecated fields, ensure file paths are correct
* NuGet.org validations on package push 

## Solution

[Packaging Icon within the nupkg](https://github.com/NuGet/Engineering/wiki/Packaging-Icon-within-the-nupkg)

[Packaging License within the nupkg](https://github.com/NuGet/Engineering/wiki/Packaging-License-within-the-nupkg)

[Packaging Documentation within the nupkg](https://github.com/NuGet/Engineering/wiki/Packaging-Documentation-within-the-nupkg)