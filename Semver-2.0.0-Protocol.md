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
4. Accessing PackageBaseAddress/3.0.0

### Areas it doesn't affect (or at least adverse affects)

1. The WebSite - Should it highlight semver 2.0 differently (or provide a filter?)
2. The Catalog - Shouldn't have any affect

### V2 (OData) endpoints

The following endpoints need to support a `semVerLevel` query parameter which determines whether SemVer 2.0.0 packages should be visible or not. The list of endpoints considered are:

 - `Packages()`
 - `Packages(Id='<ID>',Version='<VERSION>')`
 - `FindPackagesById()`
 - `Search()`
 - `GetUpdates()`

If `semVerLevel=N` is provided (where N is an integer) and N is greater than or equal to 2, SemVer 2.0.0 packages must be visible. If N is less than 2 or `semVerLevel` is excluded, SemVer 2.0.0 package must be excluded.

The `semVerLevel` query parameter is case insensitive.

The package push and delete endpoints also available via the V2 protocol do not support the `semVerLevel` query parameter. They should always operate on the entire set of packages (not just SemVer 1.0.0 packages).

### Suggestion (rough and not detailed)

1. On the search side follow the same pattern as the OData endpoints
1. On the Registration blob we should expose a new end point RegistrationsBaseUrl-Semver2/3.0.0, this is a little more interesting pattern because servers might not expose it so clients need to opt into it if available or fallback to RegistrationsBaseUrl-Semver2/3.0.0-RC.
1. For PackageBaseAddress we have two options, I'm preferring the new end point, because it lends to less requests at the expense of more storage.
1.1 We can add a new index-Semver2.json file side by side with the existing index.json. 
1.1 We can add a new end point PackageBaseAddress-SemVer2



