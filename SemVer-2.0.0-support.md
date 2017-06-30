
# NuGet SemVer 2.0.0 support 

## What is SemVer 2.0.0
NuGet currently supports SemVer 1.0.0 with additional support for four part version numbers to work with System.Version. SemVer 2.0.0 adds the concept of multiple part release labels that may be sorted numerically and metadata which is an additional string contained in the version that does not impact sorting.

http://semver.org/

### Related specs
The latest specs for SemVer 2.0.0 are here:

[SemVer 2.0.0 - client-side (nuget.exe 4.3)](https://github.com/NuGet/Home/wiki/SemVer-2.0.0-support)

[SemVer 2.0.0 - server-side](https://github.com/NuGet/Home/wiki/SemVer2-support-for-nuget.org-%28server-side%29)

This spec covers the basic client work to support SemVer 2.0.0, there is a significant drawback in this spec because it doesn't provide a backward compatible way for the older client to deal with servers that support SemVer 2.0.0 with the current set of protocols. https://github.com/NuGet/Home/wiki/Semver-2.0.0-Protocol is a suggestion for an expanded V2 & V3 protocol to allow for backward compatibility.

### Benefits
SemVer 2.0.0 allows for better sorting of release labels. NuGet users are commonly forced to append extra zeros (NuGet.Client included) when building to avoid running out of room and having the labels sorted incorrectly. For example 1.0.0-beta-09 will come before 1.0.0-beta-10 according to the SemVer 1.0.0 lexicographical sorting.  With SemVer 2.0.0 versions may be written as 1.0.0-beta.9 and 1.0.0-beta.10. These labels are read individually and numerical only labels will be sorted as integers.

Floating to a specific set of versions becomes easier with multiple release labels. Project.json can now define a dependency to float on a certain part of the label. Ex: 1.0.0-beta.5.*

Metadata can be displayed in the NuGet UI version drop down list to make it easier to identity which version of a package to use when looking for a specific change.

### Examples of SemVer 2.0.0 versions
```
1.0.0-beta.20.5
2.4.1-pre.0.134+git.hash.5aa7fa8324af609bcdb43a90e54ee076d1a6b067
2.4.1-pre.0.134+buildagent.12.date.2016.3.31.config.debug
1.0.1+security.patch.2349
```

## Spec

### Package identity
NuGet defines the unique identity of a package to be the package id and version. The version of a package is normalized to get a string that is unique for the version. This allows user input of the version to be turned into a string that can then be used in an exact url for v3 servers.

Within NuGet the metadata section of a version will be largely ignored. This information is passed along and could be displayed in the UI, but it has no impact on version equality, comparisons or ranges since it is not part of the package identity.

Examples of the normalized version for the unique identity string

| Non-normalized | Normalized |
| ----- | ----- |
| 1.0.01 | 1.0.1 |
| 1.0.0.0 | 1.0.0 |
| 1.0 | 1.0.0 |
| 1.0.0+BuildAgent1 | 1.0.0 |
| 1.0.0-alpha.1.2.30+BuildAgent1 | 1.0.0-alpha.1.2.30 |

### Pack

NuGet.exe pack should use the normalized version and drop metadata for the nupkg name. This will help avoid duplicate packages of the same unique version within a flat folder of nupkgs.

Version: PackageA 1.0.0-alpha.1.2.3+Fixes
Output: PackageA.1.0.0-alpha.1.2.3.nupkg

### Nuspec
The id field of a nuspec should contain the full version string which includes the metadata labels. This will allow the full information to flow throught the client to the UI.

#### Version ranges
Version ranges are used to describe dependency constraints, these will use the normalized versions. There is no need for metadata in ranges since it is not used for version comparisons.

### Local folders
Flat folders of nupkgs rely on the nupkg file names. These should contain the normalized version.

V3 folder structures create a folder for the version. This should also use the normalized version and not contain metadata. This ensures that the client can predict and construct the exact path even if the user has not included the optional metadata in the version field when installing it, and when retrieving a package by finding the minimum version of a range.

### HTTP sources
V2 sources need to support SemVer 2.0.0 on the /Packages method. They will return the full version in the OData XML result.

V3 sources will return the full versions in search, package base address (flatcontainer) index files, registration blobs, and other places where the client consumes version information. 

V3 sources will use normalized versions for urls, nupkg names, and other places where the client constructs urls and passes in versions. Ranges in registration blobs will not contain metadata as specified above.

## Server compatibility
| Source | SemVer 2.0.0 Support |
| ------------- | -----:|
| NuGet.Server | No |
| NuGet.org | No |
| MyGet.org | Yes |
| VisualStudio.com | Yes |

## Client compatibility

### NuGet Client 2.11.0
NuGet Client 2.12.0 and below do not support SemVer 2.0.0

### NuGet Client 3.0.0-3.4.4
3.0.0 and up support SemVer 2.0.0 with a limited capacity. Local folders and shares are unable to parse versions with multiple release labels or metadata due to the use of NuGet.Core.dll from the older 2.11.0 client.

Project.json restore supports SemVer 2.0.0 and is able to read local folders correctly since it does not depend on NuGet.Core.dll. In some scenarios metadata is written to the disk folder path which causes. Multiple release labels are supported.

### NuGet Client 3.5.0-rc1

Starting in NuGet Client 3.5.0-rc1 has added SemVer 2.0.0 for supported for local folders and UNC shares.

NuGet.exe supports SemVer 2.0.0 for the majority of scenarios but still needs work in the following areas
* List command
* Pack - SemVer 2.0.0 is blocked to avoid conflicts for NuGet.org

## Entry points

### Where versions can appear
Versions appear can appear in the following locations and enter into the NuGet client.

* Nuspec package version
* Nuspec dependency version range
* Project.json project version
* Project.json dependency version entry
* AssemblyInformationalVersionAttribute property on a dll, used by pack
* Packages.config files in both the version and allowedVersions properties.
* HTTP feeds and search
* Visual Studio .vstemplate files
* Third party extensions that install packages using the IVs interfaces

### Source types
These are sources which may allow SemVer 2.0.0 packages to enter.

 * Local folder
 * Network share
 * Local http server (ex: NuGet.Server, ProGet, Klondike)
 * Online gallery (ex: MyGet.org, NuGet.org, VisualStudio.com)
 * Install-Package with a nupkg url
 * Unzipped package repository or a 3rd party VSIX for templates

## Scenarios

#### IVsPackageInstaller / Templates
IVsPackageInstaller takes versions as strings - No changes will be required for the Visual Studio API, however callers may need to be aware that SemVer 2.0.0 strings will not work on older versions of NuGet. Currently there is no supported way of finding out which version of NuGet is installed to check this.

#### SxS between different NuGet client versions
A github project may add SemVer 2.0.0 package to a project using a newer version of the client. When consumed by a user on Visual Studio 2013 using NuGet 2.8.x the user will hit errors since the older client is unable to parse versions containing the additional SemVer 2.0.0 characters.

#### SxS with online sources
Users on non-SemVer 2.0.0 enabled clients will be unable to install packages from the gallery if there is a SemVer 2.0.0 version in the list. The older client will attempt to find the latest version by comparing each package and encounter parsing errors. The package version is checked before the minClientVersion property which makes this difficult to avoid.

This could potentially be fixed by the gallery checking the user agent and excluding SemVer 2.0.0 versions if an older client is detected. It could also be fixed by limiting SemVer 2.0.0 to the v3 feed to reduce the number of conflicts.

## Required work

| Task | Area | Days |
| ------------- |-------------| -----:|
| Unblock SemVer 2.0.0 in pack and add tests | client | 0.5 |
| Restore tests for project.json | client | done |
| install/update/restore tests for packages.config | client | 1.0 |
| Add local folder resources for v2 and v3 formats (port from nuget.core.dll or project.json restore) | client | done |
| Add unzipped repository resource for templates (port from nuget.core.dll) | client | done |
| Perf testing for file new project scenarios | client | 2.0 |
| Update NuGet.Server to use v3 libraries | nuget.server | 6.0 |
| nuget.org gallery support | gallery | ? |