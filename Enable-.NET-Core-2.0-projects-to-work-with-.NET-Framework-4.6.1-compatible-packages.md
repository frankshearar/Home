[anangaur]** Work in Progress**
## Issue
TBD: Justin

## Problem
.NET Core 2.0 projects should be able to use .NET Framework (upto version 4.6.1) packages. This is enable the new .NET Core 2.0 ecosystem make use of most of the packages on NuGet.org. (~70% of the existing packages have compatible API surface area with .NET Core 2.0)

## Who is the customer?
.NET Core 2.0/.NET Standard 2.0 customers

## Evidence
The requirement is from the .NET team(s) to make the this compatibility work (as stated in the Problem)
NuGet has a mechanism today named [PackageTargetFallback](https://docs.microsoft.com/en-us/nuget/schema/msbuild-targets#packagetargetfallback) that allows one to specify a set of compatible targets to be used when restoring packages.
[PackageTargetFallback](https://docs.microsoft.com/en-us/nuget/schema/msbuild-targets#packagetargetfallback) has a way to compute the best target match that does not work the current scenario. Details:

Let us say there is a package - PackageA, with the current structure:
```
PackageA
\build
  \netstandard2.0
    \foo.targets
  \net461
    \bar.targets 
  \uap
    \zar.targets
\lib
  \netstandard2.0
    \libfoo.dll
  \net461
    \libfoo.dll
    \libbar.dll    
  \uap
    \libfoo.dll
    \libbar.dll
\ref
  \net461
    \libfoo.dll
    \libbar.dll    
  \uap
    \libfoo.dll
    \libbar.dll
```

In the above scenario for .NET Standard 2.0 projects, if `PackageTargetFallback` is not enabled the following assets are included for the project:
```
\build\netstandard2.0\foo.targets
\lib\netstandard2.0\libfoo.dll
```

But if `PackageTargetFallback` is enabled for `net461`, the the following assets are included (**we do not want this behavior**)
```
\build\netstandard2.0\foo.targets
\lib\netstandard2.0\libfoo.dll
\ref\net461\libfoo.dll
\ref\net461\libbar.dll
```
The `\ref\net461` assets are included because in `PackageTargetFallback` mechanism gets evaluated for each of the asset-groups (i.e. build, lib, ref) **locally** and separately. 

The expected behavior is that the `PackageTargetFallback` mechanism should be evaluated **globally**. So if a matching asset is found without `PackageTargetFallback` , then `PackageTargetFallback` should not be evaluated for  

## Solution



  