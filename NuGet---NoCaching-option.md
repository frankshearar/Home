
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
Splitting nuget downloads into batches of ten to limit growth of the Global Packages Folder before deleting it.

## Who is the customer?
The primary customer for this problem is Microsoft and specifically its use of the internal tool CoreXT.
Within the VS team over 30 GB of packages are delivered to developers desktops using CoreXT which manages its own package cache. The Global Packages folder adds an additional 30GB for package content and 8GB for nupkgs to the amount of free disk space necessary to successfully complete an initial download.

## Evidence
This behavior was first noticed and reported by CoreXT users in November 2015 and has been tracked under issue [#1406](https://github.com/NuGet/Home/issues/1406). The VS team is anxious for this fix so they can completely move to v3 feeds without impacting download performance or causing developers to run out of disk space.

## Solution
Fix the bug where Global Packages Folder is used as first source when -NoCache is specified.
Add a new option that bypasses the Global Packages Folder when the -NoCache option is passed to nuget restore.
Currently the -NoCache option is totally ignored by nuget restore when working against a v3 feed, it looks in the Global Packages Folder first and caches any downloaded packages in the Global Packages Folder.
So we propose that the -NoCache option should:
* Not look in the Global Packages Folder for a cached package
* Continue to cache download packages in the Global Packages Folder
We propose that a new -NoCaching option should:
* Download packages directly to the PackagesDirectory to avoid unnecessary disk activity
Specifying both the -NoCache option and the -NoCaching option would:
* No look in the Global Packages Folder for a cached package
* Download packages directly to the PacakgesDirectory 
The primary requirement is to implement these changes for v3 feeds. However for a completely consistent solution we should also implement the same behavior for the Machine Cache for v2 feeds.

We should change the help text of the -NoCache option as it refers to the Machine Cache which is only used in v2 feeds and makes no mention of the Global Package Folder.
```
	-NoCache  Disable using the machine cache as the first package source.
```
This could be replaced with:
```
	-NoCache  Disable using the machine cache or global package folder as the first package source.
```

The help text of the new -NoCaching option can read:
```
  -NoCaching  Do not populate the machine cache or global package cache.
```
