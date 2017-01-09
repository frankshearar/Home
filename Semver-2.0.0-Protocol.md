## Issue
https://github.com/NuGet/Home/issues/3610

## Problem
With the introduction of SemVer 2.0.0 support in the client we are now facing the next challenge of making it widely available on servers that serve a large or diverse community of users.

Older versions of the NuGet client (pre 3.5 & 2.14 respectively) do not understand SemVer 2.0.0. That means that when trying to parse a list of available version from a feed, these client will error out badly and will not be able to get to a point where they can download a package.

For example imagine a package on nuget.org that releases pre-release versions with build numbers. Older clients are no longer able to parse through the version metadata and pick a version even if the one they are looking for is a SemVer 1.0.0 compliant.

For example the following available versions for a package will break older clients trying to read it.
1.0.0
2.0.0-Beta
2.0.0-RC.1

Given that we have no time machine available, we can't go back in time and fix older clients to be compliant, but we still want servers to be able to adopt SemVer 2.0 without forcing all their clients to update.

## Who is the customer?
We have two type of customers here
1. Server owners (including us as the owners of nuget.org) that want to serve a heterogeneous set of clients
2. Customers that consume packages from public or private feeds that do not wish to upgrade their visual studio/NuGet extension/nuget.exe on their build server for any reason.

## Solution

### Lets start with the idea

The suggestion is to make versions that are not SemVer 1.0.0 compliant be invisible to older client. These clients will simply not see the new pre-release versions that will break them.

### Areas it affects

1. OData requests
2. V3 Search
3. Accessing Registration blobs

### Areas it doesn't affect (or at least adverse affects)

1. The WebSite - Should it highlight semver 2.0 differently (or provide a filter?)
2. The Catalog - Shouldn't have any affect
3. Accessing PackageBaseAddress/3.0.0 - Only new clients that support SemVer 2.0.0 use this resource

### V2 (OData) endpoints

The following endpoints need to support a `semVerLevel` query parameter which determines whether SemVer 2.0.0 packages should be visible or not. The list of endpoints considered are:

 - `Packages()`
 - `FindPackagesById()`
 - `Search()`
 - `GetUpdates()`

If `semVerLevel=N` is provided (where N is a SemVer 2.0.0 version number) has a major version greater than or equal to 2, SemVer 2.0.0 packages must not be filtered out. If N has a major version is less than 2 or `semVerLevel` is excluded, SemVer 2.0.0 packages must be filtered out.

The following endpoint refers to a specific version of a package and is discovered via one of the previous endpoints. Therefore, SemVer 2.0.0 packages are **not** filtered out. The `semVerLevel` query parameter has no effect on this endpoint.

 - `Packages(Id='<ID>',Version='<VERSION>')`

The `semVerLevel` query parameter is case insensitive.

The package push and delete endpoints also available via the V2 protocol do not support the `semVerLevel` query parameter. They should always operate on the entire set of packages (not just SemVer 1.0.0 packages).

The `<d:Version>` property for the OData package should still contain the original version string, which may include version metadata (e.g. `1.0.0+git`). The `<d:NormalizedVersion>` property should not contain the version metadata. OData links typically containing the package version (e.g. self, edit, download link) must not contain the version metadata.

### Suggestion (rough and not detailed)

1. On the search side follow the same pattern as the OData endpoints
1. On the registration blob we should expose a new endpoint RegistrationsBaseUrl/Versioned, with a client version greater than or equal to 3.5.0.
