
# Package restore cache bypass option

## Problem
In large software projects with thousands of projects NuGet is used to deliver hundreds of packages containing gigabytes of content to developers desktops. In this environment download performance, download size and management of the downloaded packages is important.
As a result engineering systems have been built on top NuGet in order to manage the download and storage of the required packages.
In Nuget v3 a package restore downloads the package to the Global Packages Folder (%userprofile%.nuget\packages by default) and then copies the package to the specified PackagesDirectory.
This has some negative side effects:
* The package content is stored twice, once in the Global Packages Folder and once in the PackagesDirectory.
* The original nupkg is always retained in the Global Packages Folder
* Downloading to the cache and copying to the destination is leads to increased disk activity for the subsequent copy

Workarounds involve:
* Using file shares for feeds, where there is no caching behavior 
* Using v2 feeds, where only the nupkg is cached under Machine Cache (%localappdata%\NuGet\Cache)
* Redirecting the Global Packages Folder to a temporary location and then deleting the content after nuget returns.
* Splitting nuget downloads into batches of ten to limit growth of the Global Packages Folder before deleting it.

## Who is the customer?
The primary customer for this problem is Microsoft and specifically its use of the internal tool CoreXT.
Within the VS team over 30 GB of packages are delivered to developers desktops using CoreXT which manages its own package cache. The Global Packages folder adds an additional 30GB for package content and 8GB for nupkgs to the amount of free disk space necessary to successfully complete an initial download.

## Evidence
This behavior was first noticed and reported by CoreXT users in November 2015 and has been tracked under issue [#1406](https://github.com/NuGet/Home/issues/1406). The VS team is anxious for this fix so they can completely move to v3 feeds without impacting download performance or causing developers to run out of disk space.

## Solution
Add a new option that bypasses the Global Packages Folder when a new -DirectDownload option is passed to nuget restore.
We propose that a new -DirectDownload option should:
* Download packages directly to the PackagesDirectory to avoid unnecessary disk activity

The primary requirement is to implement these changes for package resore using packages.config against v3 feeds. However for consistency we should also implement the same behavior for the following scenarions:
* v2 feeds should not populate a cache (note: the machine cache is not populated in the latest version of nuget)
* nuget install should also honor the -DirectDownload option
* no cache should be populated when using packages.config, package identity or project.json 
* the v2 machine cache, the Global Packages Folder and the v3 http cache should not be populated when -DirectDownload is specified

The help text of the new -DirectDownload option should read:
```
  -DirectDownload  Download directly without populating the global package cache.
```
