## Issue
https://github.com/NuGet/Home/issues/4538
## Problem
Today, NuGet has no consistent story about the number of parallel HTTP requests allowed.

In our Visual Studio extension or NuGet.exe/Dotnet CLI on non-mac OS, there is no http request throttling for package dependency resolving. We end up sending hundreds requests at the same time, restore times out due to default connection limit is 2.


## Who is the customer?
Any customer who do restore with limited network resource like on docker, mono..etc.

## Current Code

We have a [`NetworkProtocolUtility.SetConnectionLimit`](https://github.com/NuGet/NuGet.Client/blob/a192c0d2c0a627642211e04ba9808f840e205d94/src/NuGet.Core/NuGet.Common/NetworkProtocolUtility.cs#L28-L42) API, but this only works on .NET Framework and sets the value to 64 on non-Mono environments (Mono is set to 1). This API is not called in VS (and should not until we know how it would effect other components in the VS process and if it's even the right solution).

[`HttpSource` has an `IThrottle` parameter](https://github.com/NuGet/NuGet.Client/blob/a192c0d2c0a627642211e04ba9808f840e205d94/src/NuGet.Core/NuGet.Protocol.Core.v3/HttpSource/HttpSource.cs#L38) which allows controlling the total number of pending HTTP requests. This is used for `--disable-parallel` by implementing `IThrottle` with a semaphore of size 1.

Package.config use maxDegreeOfParallelism  which is set to Environment.ProcessorCount by default for [dependency fetching](https://github.com/NuGet/NuGet.Client/blob/f82a7ac6b54719225cda5f3317e3c31d5843279d/src/NuGet.Core/NuGet.PackageManagement/Resolution/ResolverGather.cs#L42) and [package download & extraction](https://github.com/NuGet/NuGet.Client/blob/9e1e8b52818b4bc5dbd7e8b6049fcdf15cad04b2/src/NuGet.Core/NuGet.PackageManagement/IDE/PackageRestoreManager.cs#L393)

PackageReference throttling in [SourceRepositoryDependencyProvider](https://github.com/NuGet/NuGet.Client/blob/98fefadc885bacac0a061315580b47fa24eca53c/src/NuGet.Core/NuGet.Commands/RestoreCommand/SourceRepositoryDependencyProvider.cs#L45) only on mac for dependency resolving and use MaxDegreeOfConcurrency for [package download and extraction](https://github.com/NuGet/NuGet.Client/blob/b59110138902555ef2a8d4a8bb8913d14ff28d27/src/NuGet.Core/NuGet.Commands/RestoreCommand/ProjectRestoreCommand.cs#L236). The default value for both are 16

## Solution

We should provide a way to control the max http request from NuGet.Config.

```
<config>
    <add key="maxHttpRequest" value="16" />
</config>
```

This setting can control all http request number sending from nuget for package dependency resolving and package downloading.


## Open question

1. Do we need a default value for this setting on non-mac scenario? We don't have any throttling on that now, since throttling hurts performance. 

2. Do we still keep high level throttling like throttling in NuGetPackageManager/SourceReposiotryDependencyProvider after we have this low level throttling on httpClient?



