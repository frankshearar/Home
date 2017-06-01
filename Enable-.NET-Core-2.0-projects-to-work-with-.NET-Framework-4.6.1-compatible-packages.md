[anangaur]** Work in Progress**
## Issue
[#5192](https://github.com/NuGet/Home/issues/5192) Enable .NET Core 2.0 projects to work with .NET Framework 4.6.1 compatible packages 

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
**Current behavior**: The `\ref\net461` assets are included because in `PackageTargetFallback` mechanism gets evaluated for each of the asset-groups (i.e. build, lib, ref) **locally** and separately. 

**Expected behavior** The `PackageTargetFallback` mechanism should be evaluated **globally** i.e. if a matching asset is found without `PackageTargetFallback` , then `PackageTargetFallback` should not be evaluated at all. In other words, `PackageTargetFallback` should only kick in when no matching asset if found in the default TFM match mechanism.

## Solution

The proposed solution is to enable fallback mechanism that is evaluated **globally** as explained above. This mechanism will be called "AssetTargetFallback". The existing `PackageTargetFallback` should be left as is lest we break users that are relying on this mechanism. 

**How does it work**
Terms:
`default-tfm-match`: Mechanism to find the best match based on the target framework of the project. 
`AssetTargetFallback`: Mechanism to find matching assets based on the targets specified using `AssetTargetFallback` 
`No-ref-and-lib-rule`: If a package does not have a ref and lib asset groups (folders), then the package is deemed compatible even when there is no assets pulled in to the project by NuGet. This is to support meta-packages that have only dependencies and no assets.

Here is how `AssetTargetFallback` mechanism will work:

------------

Step 1. Evaluate if any assets match using default-tfm-match mechanism

Step 2. If #1 returns one or more matching assets, do not evaluate the package for `AssetTargetFallback` assets. Go to **Step S**

Step 3. Else If #1 returns 0 matching assets (including package install failure), Go to **Step 4**

Step 4. Evaluate if any assets match using `AssetTargetFallback` mechanism.

Step 5. If #4 returns one or more matching assets, Go to **Step S**

Step 6. Else If #4 returns 0 matching assets, Go to **Step N**\

...

Step N. Evaluate if `No-ref-and-lib-rule` applies. If **Yes** Go to **Step S**, Else Go to **Step F** 

Step S. Package install ```success```

Step F. Package install ```failed```

------------

**How does it behave for different package structures?**

For a .NET Standard 2.0 project, following is a table that explains which assets are pulled in, based on different approaches i.e. `default-tfm-match only` without PTF/ATF, with `AssetTargetFallback` ( for net461) and with `PackageTargetFallback` (for net461)

| Package structure  | `default-tfm-match only` (netstandard2.0) | `AssetTargetFallback`(net461)  | `PackageTargetFallback`(net461)  | 
|---|---|---|---|
| `build\foo.targets`  | `build\foo.targets`  | **NE** `build\foo.targets`  | **NE** `build\foo.targets`  |
| `build\netstandard1.0\foo.targets`  | `build\netstandard1.0\foo.targets`  | **NE** `build\netstandard1.0\foo.targets`   | **NE** `build\netstandard1.0\foo.targets`  |
| `build\net461\foo.targets`  | The package gets installed because of `No-ref-and-lib-rule` but no assets gets pulled in | `build\net461\foo.targets` | `build\net461\foo.targets`  |
| `build\netstandard2.0\foo.targets` `build\net461\bar.targets` `lib\netstandard2.0\libfoo.dll` `ref\net461\libbar.dll` | `build\netstandard2.0\foo.targets` `lib\netstandard2.0\libfoo.dll`  | **NE** `build\netstandard2.0\foo.targets` `lib\netstandard2.0\libfoo.dll`  | `build\netstandard2.0\foo.targets` `lib\netstandard2.0\libfoo.dll`  `ref\net461\libbar.dll` |
| `build\net461\bar.targets` `ref\net461\libbar.dll` | No matching asset; Package fails to install | `build\net461\bar.targets` `ref\net461\libbar.dll` | `build\net461\bar.targets` `ref\net461\libbar.dll` |
| `build\bar.targets` `ref\net461\libbar.dll` | `build\bar.targets` | **NE** `build\bar.targets` | `build\net461\bar.targets` `ref\net461\libbar.dll` |

**What happens when both `AssetTargetFallback` and `PackageTargetFallback` are used in a project?**

NuGet detects this and outputs an error (**NU1003**):
```
The project uses 'AssetTargetFallback' along with deprecated 'PackageTargetFallback' to specify compatible targets to be used when restoring packages. This is not supported. Please remove 'PackageTargetFallback' references from the project environment and re-run restore.
``` 