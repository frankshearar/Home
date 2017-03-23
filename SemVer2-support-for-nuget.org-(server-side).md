Background
==========

The nuget.org gallery currently supports a package versioning mix of both **System.Version format** (source <https://msdn.microsoft.com/en-us/library/system.version(v=vs.110).aspx)>, as well as **SemVer v1.0.0 format** (source <http://semver.org/spec/v1.0.0.html>).

The introduction of the **SemVer v2.0.0 spec** (source <http://semver.org/spec/v2.0.0.html>) creates demand for this versioning format on nuget.org, as the spec adds support for adding arbitrary metadata to the version string (which does not impact version precedence), 
as well as multiple-part release labels, which may be sorted numerically when including numeric identifiers. 
This allows for better sorting of release labels, which today can be counter-intuitive and require usage of work-arounds such as leading zeros or reverse date-time stamps.

SemVer v2.0.0 version formats are currently not supported by the nuget.org gallery.

**Related design:**
SemVer2 support spec as defined by the NuGet client: <https://github.com/NuGet/Home/wiki/SemVer-2.0.0-support>

Adding support for SemVer v2.0.0 to nuget.org is important because:

-   Semantic Versioning **encourages package authors to communicate intent** to
    package consumers. The semantics of incrementing major, minor and patch
    version parts are well defined in the spec, and SemVer v2.0.0 is the latest
    standard. These version parts should be incremented *intentionally* based on changes 
    in the public API.

-   Adopting SemVer v2.0.0 support on nuget.org helps us incorporate feedback from 
    **package authors having trouble defining auto-increments in version** numbers 
    (e.g. on CI systems), as well as from **package consumers fighting tooling that did not properly handle
    version precedence intuitively**. Until now, NuGet package authors had no
    choice but to auto-increment any version part that was meant to communicate
    intent, or work around version precedence limitations using leading zeros or
    reverse date-time stamps, which is really suboptimal, and the source of
    confusion and support requests. SemVer v2.0.0 specifically adds support for
    numeric identifiers in (pre)release labels which defines version precedence.

The below table summarizes current versioning support in NuGet compared to the SemVer v2.0.0 spec, and clearly shows that the SemVer v2.0.0 versioning format is a super-set of SemVer v1.0.0.

| Versioning scheme | Format | SemVer v1 | SemVer v2 | NuGet.org |
| ------------- | ------------- | ------------- | ------------- | ------------- |
| System.Version | `major.minor.build.revision[-prerelease]` | Not supported | Not supported | Supported |
| SemVer v1.0.0 | `major.minor.patch[-prerelease]` | Supported | Supported | Supported |
| SemVer v2.0.0 | `major.minor.patch[-prerelease][+metadata]` | Not supported | Supported | Not supported |

Goals
=====

-   We must ensure full backwards-compatibility with existing, older clients who
    may break when receiving SemVer v2.0.0 package versions.

-   We must ensure SemVer v2.0.0 package versions can be pushed/uploaded to
    nuget.org.

-   We must ensure newer SemVer v2.0.0 supporting clients can consume SemVer
    v2.0.0 package versions.

Non-Goals
=========

Normalized package versions do not include the metadata part, as this is not
part of the package identity, and no version precedence can be defined on the
metadata part. As such, **only a single package version including metadata can
be uploaded to nuget.org per given normalized version.** Uploading another
package version that only differs in metadata part will cause a collision and
will be rejected as the *package version already exists*. We do not intend to
work around this restriction.

An additional edge-case is possible when using older versions of NuGet clients
that do not support SemVer v2.0.0 packages. A user could upload a package A with
version v1.0.0 to nuget.org that depends on a package B \>= v1.0.0 from another
user. If package B is actually a SemVer v2.0.0 package (e.g. v1.0.0+metadata or
v1.0.0-alpha.1), then older clients will be unable to install package A due to
not finding package B, even though it looks like everything will be SemVer
v1.0.0 from the point of view of package A's NuGet manifest (nuspec).

This is a very hard problem to solve fully recursively. Since a dependency might
not be available on nuget.org, or one restore may pull in a SemVer v2.0.0
transitive dependency while another does not, we would be hard-pressed to
correctly implement a definition of SemVer v2.0.0 packages that takes *every
possible* restore graph into account.

Solution
========

Backwards-Compatibility
-----------------------

Older clients that do not support SemVer v2.0.0 package versions may consume
both v1, v2 and v3 nuget.org endpoints. In addition, our very own v3
registration blobs are the result of the following data flow:

1.  When a package is uploaded, it goes into the nuget.org SQL database;

2.  then the *feed2catalog* job is consuming the v2 endpoint to pick it up and
    write it to the catalog;

3.  after which the *catalog2registration* job generates the registration blobs,
    and the *catalog2lucene* job writes to the Lucene search index.

This effectively means that:

-   The v1 and v2 OData endpoints should filter out SemVer v2.0.0 package
    versions **by default**.

-   The v1 OData endpoint does not need to support the `semVerLevel` parameter.

-   The v2 OData endpoints should be able to return SemVer v2.0.0 package
    versions **upon explicit request**. This is *intended* to be solely used by
    the **feed2catalog** job, however, SemVer v2.0.0 supporting clients
    targeting these v2 OData endpoints may also be using this. It is still
    convenient to have this fallback-scenario supported in case we need to point
    people to the v2 endpoint when v3 is in trouble.

Therefore, a new, optional query parameter **semVerLevel** is to be introduced
on these v2 OData endpoints. Only when **semVerLevel>=2.0.0** will these
endpoints include SemVer v2.0.0 package versions.

The `semVerLevel` query parameter is **NOT case-sensitive**.

Older clients will not send this query parameter at all, so the default value
will be null (no need to set it to any arbitrary string value as it's an
unnecessary allocation and its very absence has the same meaning).

Newer clients supporting SemVer v2.0.0 must send this query parameter
`semVerLevel=2.0.0` in order to consume these package versions from these
endpoints.

The same requirements above apply to the **search service**.

Forward-Compatibility
---------------------

However unlikely, if one day a newer version of the SemVer spec is made available, and we decide
we want to support it, we should be able to easily add support for it without
overhauling our APIs or functionality. One can easily imagine e.g.
`semVerLevel=2.1.0` being passed to these APIs one day.

The `semVerLevel` query string parameter is flexible on the surface and will not
require a change in API. Internally, server implementations can easily map these
query parameter values to whatever implementation detail is used on the
back-end. Using a string parameter on the public API does not enforce us to use
a string-based filter internally, allowing us to optimize independent from the
public API.

Identifying SemVer v2.0.0 Packages
----------------------------------

A **version** is defined as SemVer v2.0.0 if either of the following statements
is true:

-   The (pre)release label is **dot-separated**, e.g. `1.0.0-alpha.1`

-   The version has build-**metadata**, e.g. `1.0.0+githash`

A **package** is defined as a SemVer v2.0.0 package if either of the following
statements is true:

-   The package's own **version** is SemVer v2.0.0 compliant but not SemVer
    v1.0.0 compliant, as per the above definition.

-   Any of the package's **dependency version ranges** has a minimum or maximum
    version that is SemVer v2.0.0 compliant but not SemVer v1.0.0 compliant, as
    per the above definition; e.g. `[1.0.0-alpha.1, )`.

Validation
----------

### Package Versions and Version Ranges

The nuget.org gallery currently has a few validation rules that prevent
uploading of SemVer v2.0.0 package versions. Currently, the following package
versions are explicitly blocked as being invalid by the gallery
(<https://github.com/NuGet/NuGetGallery/issues/3645>):

-   Prerelease tags that only contain numerics

-   Prerelease tags that contain a dot

In addition, the following additional validation needs to happen in order to
support proper SemVer v2.0.0 package versions:

-   Numeric identifiers in prerelease tags cannot have leading zeros. **Note:
    identifiers are dot-separated! (not hyphen separated!)**

    *This means that e.g. -pre-001 would still be allowed as a valid prerelease
    tag, however, -pre.001 would not be.*
    <https://github.com/NuGet/NuGetGallery/issues/3648>

-   Dependency version ranges should not contain metadata parts.
    <https://github.com/NuGet/NuGetGallery/issues/3482>

### semVerLevel Query Parameter

The `semVerLevel` query parameter should be treated as a `NuGetVersion` (a version string). 
If it cannot be parsed, it has the same meaning as its very absence. 
If the parsed version is greater than or equal to `2.0.0` and has a major version lower than `3`, the result set must include SemVer v2.0.0 package versions. That means `2.1.0` is accepted and treated the same as SemVer v2.0.0. 
If `semVerLevel >= 3.0.0` is provided, we fallback to `semVerLevel=2.0.0`, as it leaves opportunity to the client to start supporting a hypothetical SemVer v3.0.0 before servers adopt support for it.

V1 OData Endpoints
------------------

The following v1 endpoints are impacted by this change (to filter out SemVer v2.0.0 packages):

-   /api/v1/Packages

-   /api/v1/Packages/\$count

-   /api/v1/FindPackagesById

-   /api/v1/Search

-   /api/v1/Search/\$count

### ODataV1FeedController

The v1 feed endpoint does not need to accept the `semVerLevel` query parameter, and should be modified to always filter out SemVer v2.0.0 packages from the result set.

V2 OData Endpoints
------------------

The following endpoints are impacted by this change:

-   /api/v2/Packages

-   /api/v2/Packages/\$count

-   /api/v2/FindPackagesById

-   /api/v2/Search

-   /api/v2/Search/\$count

-   /api/v2/GetUpdates

-   /api/v2/GetUpdates/\$count

Curated feed endpoints:

-   /api/v2/curated-feed/curatedFeedName/Packages

-   /api/v2/curated-feed/curatedFeedName/Packages/\$count

-   /api/v2/curated-feed/curatedFeedName/FindPackagesById

-   /api/v2/curated-feed/curatedFeedName/Search

-   /api/v2/curated-feed/curatedFeedName/Search/\$count

-   /api/v2/curated-feed/curatedFeedName/GetUpdates

-   /api/v2/curated-feed/curatedFeedName/GetUpdates/\$count

OData specifics to be aware of:

-   The **\<d:Version\>** property of the OData package must still contain the
    **original version string**! This property value is to be used by
    *feed2catalog* and other v3 components when determining the semVerLevel.

-   The **\<d:NormalizedVersion\>** property of the OData package **may not
    contain the version metadata**!

-   OData **links** typically containing the package version (e.g. self, edit,
    download link) **must not contain the version metadata**!

-   As this query parameter is a string, we should consistently use the
    [FromUri]** attribute to avoid potential parsing errors (see
    <http://blog.richschoenrock.com/2015/02/16/asp-net-odata-function-parameter-parsing/>).
    Every other parameter is treated as an OData string ('2.0.0' instead of just
    2.0.0), and we want `semVerLevel` to be consistent across all services. This
    is also how the client and NuGet.Server are implemented today.

-   The OData feed will NOT expose a `semVerLevel` (or IsSemVer2) property.
    Consumers of the feed can calculate this for themselves if needed (e.g.
    **feed2catalog**) using a shared library (NuGet.Versioning).

-   The values for the IsLatestVersion and IsAbsoluteLatestVersion properties
    will be intercepted and depend on the provided `semVerLevel` parameter.
    Suppose you have versions 1.0.0-a and 1.0.0-b.1; if `semVerLevel < 2.0.0`,
    then `1.0.0-a` is the absolute latest. If `semVerLevel=2.0.0`, then `1.0.0-b.1` is
    absolute latest. This could be based on additional *internal*
    SemVer2IsLatestVersion and SemVer2IsAbsoluteLatestVersion properties.

### ODataV2FeedController

This is the main v2 OData endpoint that needs to filter SemVer v2.0.0 package
versions by default, unless provided with the `semVerLevel=2.0.0` parameter.

#### /api/v2/Packages?semVerLevel=2.0.0

```csharp
public async Task<IHttpActionResult> Get(

    ODataQueryOptions<V2FeedPackage> options,

    [FromUri]string semVerLevel = null)
```

#### /api/v2/Packages/\$count?semVerLevel=2.0.0

```csharp
public async Task<IHttpActionResult> GetCount(
    ODataQueryOptions<V2FeedPackage> options,
    [FromUri]string semVerLevel = null)
```

#### /api/v2/FindPackagesById()?id=&semVerLevel=2.0.0

```csharp
public async Task<IHttpActionResult> FindPackagesById(
    ODataQueryOptions<V2FeedPackage> options,
    [FromODataUri]string id,
    [FromUri]string semVerLevel = null)
```

#### /api/v2/Search()?searchTerm=&targetFramework=&includePrerelease=&semVerLevel=2.0.0

```csharp
public async Task<IHttpActionResult> Search(
    ODataQueryOptions<V2FeedPackage> options,
    [FromODataUri]string searchTerm = "",
    [FromODataUri]string targetFramework = "",
    [FromODataUri]bool includePrerelease = false,
    [FromUri]string semVerLevel = null)
```

#### /api/v2/Search()/\$count?searchTerm=&targetFramework=&includePrerelease=&semVerLevel=2.0.0

```csharp
public async Task<IHttpActionResult> SearchCount(
    ODataQueryOptions<V2FeedPackage> options,
    [FromODataUri]string searchTerm = "",
    [FromODataUri]string targetFramework = "",
    [FromODataUri]bool includePrerelease = false,
    [FromUri]string semVerLevel = null)
```

#### /api/v2/GetUpdates()/?packageIds=&includePrerelease=&includeAllVersions=&targetFrameworks=&versionConstraints=&semVerLevel=2.0.0

```csharp
public async Task<IHttpActionResult> GetUpdates(
    ODataQueryOptions<V2FeedPackage> options,
    [FromODataUri]string packageIds,
    [FromODataUri]string versions,
    [FromODataUri]bool includePrerelease,
    [FromODataUri]bool includeAllVersions,
    [FromODataUri]string targetFrameworks = "",
    [FromODataUri]string versionConstraints = "",
    [FromUri]string semVerLevel = null)
```

#### /api/v2/GetUpdates()/\$count?packageIds=&includePrerelease=&includeAllVersions=&targetFrameworks=&versionConstraints=&semVerLevel=2.0.0

```csharp
public async Task<IHttpActionResult> GetUpdatesCount(
    ODataQueryOptions<V2FeedPackage> options,
    [FromODataUri]string packageIds,
    [FromODataUri]string versions,
    [FromODataUri]bool includePrerelease,
    [FromODataUri]bool includeAllVersions,
    [FromODataUri]string targetFrameworks = "",
    [FromODataUri]string versionConstraints = "",
    [FromUri]string semVerLevel = null)
```

### ODataV2CuratedFeedController

This is the v2 OData endpoint for curated feeds that needs to filter SemVer
v2.0.0 package versions by default, unless provided with the `semVerLevel=2.0.0`
parameter.

#### /api/v2/curated-feed/curatedFeedName/Packages?semVerLevel=2.0.0

```csharp
public async Task<IHttpActionResult> Get(
    ODataQueryOptions<V2FeedPackage> options,
    [FromODataUri]string curatedFeedName,
    [FromUri]string semVerLevel = null)
```

#### /api/v2/curated-feed/curatedFeedName/Packages/\$count?semVerLevel=2.0.0

```csharp
public async Task<IHttpActionResult> GetCount(
    ODataQueryOptions<V2FeedPackage> options,
    [FromODataUri]string curatedFeedName,
    [FromUri]string semVerLevel = null)
```

#### /api/v2/curated-feed/curatedFeedName/FindPackagesById()?id=&semVerLevel=2.0.0

```csharp
public async Task<IHttpActionResult> FindPackagesById(
    ODataQueryOptions<V2FeedPackage> options,
    [FromODataUri]string curatedFeedName,
    [FromODataUri]string id,
    [FromUri]string semVerLevel = null)
```

#### /api/v2/curated-feed/curatedFeedName/Search()?searchTerm=&targetFramework=&includePrerelease=&semVerLevel=2.0.0

```csharp
public async Task<IHttpActionResult> Search(
    ODataQueryOptions<V2FeedPackage> options,
    [FromODataUri]string curatedFeedName,
    [FromODataUri]string searchTerm = "",
    [FromODataUri]string targetFramework = "",
    [FromODataUri]bool includePrerelease = false,
    [FromUri]string semVerLevel = null)
```

#### /api/v2/curated-feed/curatedFeedName/Search()/\$count?searchTerm=&targetFramework=&includePrerelease=&semVerLevel=2.0.0

```csharp
public async Task<IHttpActionResult> SearchCount(
    ODataQueryOptions<V2FeedPackage> options,
    [FromODataUri]string curatedFeedName,
    [FromODataUri]string searchTerm = "",
    [FromODataUri]string targetFramework = "",
    [FromODataUri]bool includePrerelease = false,
    [FromUri]string semVerLevel = null)
```

#### /api/v2/curated-feed/curatedFeedName/GetUpdates()/?packageIds=&includePrerelease=&includeAllVersions=&targetFrameworks=&versionConstraints=&semVerLevel=2.0.0

```csharp
public async Task<IHttpActionResult> GetUpdates(
    ODataQueryOptions<V2FeedPackage> options,
    [FromODataUri]string curatedFeedName,
    [FromODataUri]string packageIds,
    [FromODataUri]string versions,
    [FromODataUri]bool includePrerelease,
    [FromODataUri]bool includeAllVersions,
    [FromODataUri]string targetFrameworks = "",
    [FromODataUri]string versionConstraints = "",
    [FromUri]string semVerLevel = null)
```

#### /api/v2/curated-feed/curatedFeedName/GetUpdates()/\$count?packageIds=&includePrerelease=&includeAllVersions=&targetFrameworks=&versionConstraints=&semVerLevel=2.0.0

```csharp
public async Task<IHttpActionResult> GetUpdatesCount(
    ODataQueryOptions<V2FeedPackage> options,
    [FromODataUri]string curatedFeedName,
    [FromODataUri]string packageIds,
    [FromODataUri]string versions,
    [FromODataUri]bool includePrerelease,
    [FromODataUri]bool includeAllVersions,
    [FromODataUri]string targetFrameworks = "",
    [FromODataUri]string versionConstraints = "",
    [FromUri]string semVerLevel = null)
```

V2 Auto-Complete Endpoints
--------------------------

The following auto-complete endpoints need to filter SemVer v2.0.0 package
versions by default, unless provided with the `semVerLevel=2.0.0` parameter.

-   /api/v2/package-ids

-   /api/v2/package-versions

### ApiController

#### /api/v2/package-ids?semVerLevel=2.0.0

```csharp
public async Task<ActionResult> GetPackageIds(
    string partialId,
    bool? includePrerelease,
    string semVerLevel = null)
```

#### /api/v2/package-versions?semVerLevel=2.0.0

```csharp
public async Task<ActionResult> GetPackageVersions(
    string id,
    bool? includePrerelease,
    string semVerLevel = null)
```

### Database Queries

The following database queries need modification to support filtering on SemVer
v2.0.0 package versions:

-   AutoCompleteDatabasePackageIdsQuery

-   AutoCompleteDatabasePackageVersionsQuery

The SQL queries will need to be expanded to filter on `semVerLevel`.

### Search Service Queries

The following search service queries need modification to support filtering on
SemVer v2.0.0 package versions:

-   AutoCompleteServicePackageIdsQuery

-   AutoCompleteServicePackageVersionsQuery

When semVerLevel parameter is provided, we will call into the v3 search service
using the same additional `semVerLevel` query parameter.

Gallery Database Changes
------------------------

For faster querying/filtering, we should store that a package version is
`semVerLevel=2.0.0`.

In addition, we need to align/validate constraints on version length
(<https://github.com/NuGet/NuGetGallery/issues/3561>).

**Design choice:** There are multiple options how to store the fact that a
package is `semVerLevel=2.0.0`:

1.  Add new column `SemVerLevel` of type `nvarchar(N)` and store `"2.0.0"` for each
    SemVer v2.0.0 package version;
;
2.  Add new column `IsSemVer2` of type `bit`, and store `0 or 1` when applicable

3.  Add new column `SemVerLevelKey` of type `int`, and add new table
    `SemVerLevels`, and join queries;

4.  Add new column `SemVerLevelKey` of type `int`, hard-code the value for now, and
    have the ability to expand with another table later on when needed.

*Option c. seems to combine the best combination of forward-compatibility and
database normalization. Short-term, option d. is preferred as it is least
intrusive, and still allows us to expand towards option c. later on.*

The nuget.org gallery, gallery database, and NuGet clients restrict package
versions to **64 characters**. There's already packages in the database with
a version-length of 58 characters. Introducing support for additional version parts 
(metadata, and numeric identifiers in release labels) to support the SemVer v2.0.0 format, 
may introduce the need to expand the maximum supported original `Version` length.

This does not affect `NormalizedVersion` (which does not include the metadata part), 
but `Version` should still be able to contain the original version string (including metadata part).
See <https://github.com/NuGet/NuGetGallery/issues/3561>.

Gallery Frontend Changes
------------------------

Currently, the gallery does not seem to properly use NuGet.Versioning in all
places that require the normalized version string. More specifically, this is
going to be important on the package details page.

We must ensure that newly uploaded SemVer v2.0.0 packages are properly
normalized both in URL and view.

Example: when uploading a SemVer v2.0.0 package version
`1.0.0-release.123+metadata`, the corresponding package details URL on nuget.org
will be: `https://www.nuget.org/packages/packageid/1.0.0-release.123`

**Note the normalized version string in the URL, which excludes metadata, but
includes multiple part release labels!**

Search Service API Changes
--------------------------

Older versions of the NuGet client (pre-3.5 & 2.14 respectively) do not
understand SemVer 2.0.0. Therefore, by default, we should always restrict the
result set to exclude package versions for which `semVerLevel == null` or
`semVerLevel >= "2.0.0"`.

If `semVerLevel=N` is provided (where `N` is a SemVer 2.0.0 version number) and
has a major version greater than or equal to `2.0.0`, SemVer 2.0.0 packages
must be included in the result set.

If `N` has a major version less than `2.0.0` or `semVerLevel` is excluded,
SemVer 2.0.0 packages must be filtered out.

### V3 Auto-Complete Endpoints

Addition of a new, optional `semVerLevel` query string parameter (default value =
`null`):

/autocomplete?q=&id=&skip=&take=&includePrerelease=&explanation=**&semVerLevel=2.0.0**

### V3 Search Endpoint

Addition of a new, optional `semVerLevel` query string parameter (default value =
`null`):

/query?q=&feed=&skip=&take=&includePrerelease=&includeExplanation=**&semVerLevel=2.0.0**

### V2 Search Endpoint

Addition of a new, optional `semVerLevel` query string parameter (default value =
`null`):

/search/query?q=&feed=&skip=&take=&includePrerelease=&ignoreFilter=&countOnly=&sortBy=**&semVerLevel=2.0.0**

### Search Service Validation

Likely similar to gallery validation and error handling. 

### Internal Implementation Changes

#### NuGet.Indexing.LatestListedHandler

An update to the constructor is required to take a new parameter `includeSemVer2`.
This will expand the filter building to include this, doubling to 8 combinations
of bits. Bits are: `includeUnlisted`, `includePrerelease`, `includeSemVer2`.

```csharp
public LatestListedHandler(
    bool includeUnlisted,
    bool includePrerelease,
    bool includeSemVer2)
```

#### NuGet.Indexing.NuGetSearcherManager

The `CreateSearcher` method needs updates to process all combinations of above
bits. We need to create a new bit set `latestSemVer2BitSet`, similar to
`latestBitSet`.

#### NuGet.Indexing.NuGetIndexSearcher

We need to store another bit set, `latestSemVer2BitSet`, during construction. The
latest parameter will become a 3-dimensional filter array.

```csharp
public NuGetIndexSearcher(
    // removed other arguments for brevity
    Filter[][][] latest,
    OpenBitSet latestSemVer2BitSet,
    OwnersResult owners)
```

In addition, we need to update the `TryGetFilter` method with a new parameter
`includeSemVer2`.

```csharp
public bool TryGetFilter(
    bool includeUnlisted,
    bool includePrerelease,
    bool includeSemVer2,
    string curatedFeed,
    out Filter filter)
```

#### NuGet.Indexing.ResponseFormatter

The `WriteData` method now needs to include a filter for the new
`latestSemVer2BitSet`.

```csharp
private static void WriteData(
    // removed other arguments for brevity
    bool includePrerelease,
    bool includeExplanation,
    bool includeSemVer2,
    Query query)
```

#### NuGet.Indexing.ServiceImpl

The `Search` method needs to be modified to accept a new parameter `includeSemVer2`,
which will be passed on to the updated `NuGetIndexSearcher.TryGetFilter` call.

```csharp
public static void Search(
    // removed other arguments for brevity
    bool includeSemVer2)
```

The `AutoComplete` method needs to be modified to accept a new parameter
`includeSemVer2`, which will be passed on to both `NuGetIndexSearcher.TryGetFilter`
calls.

```csharp
public static void AutoComplete(
    // removed other arguments for brevity
    bool includeSemVer2)
```

The `Find` method needs to be modified to accept a new parameter `includeSemVer2`,
and have its query changed to a `FilteredQuery` using
`NuGetIndexSearcher.TryGetFilter`. Registration base address also needs to be made
aware of `includeSemver2` as it will need to return responses to different hives
depending on this parameter.

```csharp
public static void Find(
    // removed other arguments for brevity
    bool includeSemVer2)
```

#### NuGet.Services.BasicSearch.ServiceEndpoint

This class needs a new method GetSemver2 which returns a Boolean after parsing
the incoming `semVerLevel` query string parameter. This method will be leveraged
in the Search Service API changes to translate the query string value to a
binary true or false filter for internal use.

### Lucene Index Changes

In order for the search service to be able to filter on `semVerLevel`, the Lucene
index will need to hold the fields containing the data to filter on. This data
will flow to the Lucene index from the gallery SQL database. This is an internal
implementation detail.

**Design choice:** internally, it is possible to either:

1.  Use a text field and index: this is more expensive to search on, but it also
    means that we will probably stay future proof when a new `semVerLevel` is
    introduced.

2.  Use a Boolean field `IsSemVer2` and a filter: this is faster, but means that
    we will need to add new bits every time a new `semVerLevel` is introduced.

*Option a. is preferred as it is a similar pattern like we use today for
unlisted/prerelease filtering and seems well suited for this scenario as well,
as the introduction of new semVerLevel's is an unlikely, rare event, and the
work to introduce support for another semVerLevel does not weigh up against
the performance we experience on day-to-day basis using NuGet search.*

For this, the query parser needs to be updated to understand the new `semVerLevel`
query parameter, and a new filter needs to be added for the `IsSemVer2` field.

#### Catalog2Lucene

The **catalog2lucene** job need to be updated to write the `IsSemVer2` field to
the index for incoming packages.

The job must independently compute if the entry is a SemVer v2.0.0 package using
a shared library call (`NuGet.Versioning`).

#### Sql2Lucene

The **sql2lucene** jobs needs to be updated to write the `IsSemVer2` field to the
index for incoming packages.

#### Feed2Catalog

The **feed2catalog** job must be changed to provide the `semVerLevel=2.0.0`
parameter when querying the feed. Additionally, catalog entry URLs must use the
normalized version, not the full version. The catalog entry JSON payload must
include the full version.

The catalog page structure would stay as it looks today.

#### Catalog2Registration

The **catalog2registration** job needs to be updated to **NOT** write SemVer
v2.0.0 packages to *old registration* blobs, and to write SemVer v2.0.0 packages
to a new registration container. These new blobs will be gzipped since there is
no such client that supports SemVer v2.0.0 registration blobs but does not
support gzipped registration blobs.

The catalog2registration job must independently compute if the entry is a SemVer
v2.0.0 package using a shared library call (`NuGet.Versioning`).

Today, there are two registration hives visible from the `api/v3/index.json`:

1.  <https://api.nuget.org/v3/registration1/>

2.  <https://api.nuget.org/v3/registration1-gz/>

We will be adding a third:

1.  <https://api.nuget.org/v3/registration1-semver2-gz/>

The job will be modified to produce all three sets of registration blobs based
on a single cursor. This the logical extension of how a single cursor is used
for producing the two existing hives, today.

#### V3/index.json

The only additional change in to the service index is the addition of the new
registration hive. It will make use of the new "clientVersion" feature so NuGet
4.0.0 can light up:

```json
{

"@id": "https://api.nuget.org/v3/registration1-semver2-gz/",

"@type": "RegistrationsBaseUrl",

"clientVersion": "4.3.0",

"comment": "Base URL of Azure storage where NuGet package registration info is
stored"

}
```

Testing
=======

For both gallery and search service, we need to add unit test coverage for all
validation logic changes, and we should expand our existing functional tests to
also cover for the `semVerLevel=2.0.0` scenarios.

For the modified jobs, we need tests to verify correctness of marking SemVer
v2.0.0 package versions. Indexing needs tests to validate that SemVer v2.0.0
package versions are understood and properly filtered.

Unit tests
----------

Today, the unit testing and integration testing of the V3 jobs is pretty
lack-luster. As a prerequisite for this feature shipping, we should commit to
having full Xunit test coverage on code that we change.

End to End tests
----------------

Additionally, we should add a simple E2E test to the Gallery functional test
suite that tests each modified endpoint:

1.  Does a SemVer 2.0.0 package show up in the new registration hive and not
    show up in the old hive?

2.  Does a SemVer 2.0.0 package show up in the search service and observer
    `semVerLevel`?

3.  Does a SemVer 2.0.0 package show up in `FindPackagesById()`, hijacked.

4.  Does a SemVer 2.0.0 package show up in `Packages` collection, not hijacked.

To be clear, this is not the very best possible place for end-to-end tests, but
this is the most convenient and allows us to move forward without spending a lot
of time building new test infrastructure for V3 end-to-end testing.

Monitoring
==========

Additional monitoring just focusing on SemVer 2.0.0 changes should be limited
given that the majority of changes are to existing endpoints and services.

However, the V3 pipeline verification should be
extended to verify that the SemVer 2.0.0 registration hive both (a) is gzipped
and (b) has all SemVer 1.0.0 and SemVer 2.0.0 packages.

We should emit telemetry for `semVerLevel` on push events to Application Insights
(add a flag to existing event).

Execution Phases
================

Db Schema + Update for Is(Absolute)LatestVersion
------------------------------------------------

We'll first need to perform the required schema and data updates to add support
for the new `SemVer2LatestVersion` and `SemVer2LatestAbsoluteVersion` properties,
and update the data already present in the gallery database.

In addition, we need to update the push logic to store the package's `SemVerLevelKey`. 
The default value will be `null`, as we don't want to parse the entire database to identify
either System.Versioning version formats, or SemVer v1.0.0 version formats. 
We don't differentiate between the two anyway, and defaulting to `null` avoids the need to
patch the entire database before enabling actual uploads of SemVer v2.0.0 versioned packages.

Generate the new V3 artifacts
-----------------------------

After updating **feed2catalog**, **catalog2registration**, and
**catalog2lucene** to properly handle SemVer 2.0.0, we should run the jobs from
the beginning of time to produce new SemVer 2.0.0-aware artifacts. In
particular:

-   **catalog2registration** or **lightning**: generate all three registration
    hives in new storage containers.  
    The SemVer 2.0.0 and gzipped hives should be exactly the same given there
    are no SemVer 2.0.0 packages.

-   **catalog2lucene** or **db2lucene**: generate the new Lucene index.

Filter by default (nuget.org)
-----------------------------

In order to be able to leverage rolling deployments (with rollback possibility),
we should first apply all proper default filtering on the v1 and v2 endpoints.

In addition, we should already fix any version normalization issues on the
gallery frontend to leverage the latest `NuGet.Versioning` APIs, to ensure any
future SemVer v2.0.0 package versions are properly displayed, and accessible
through the web site using the properly normalized version string in route
parameters.

This deployment will also contain the upgrade to `NuGet.* v4.0.0` dependencies,
which are required to leverage the new `NuGet.Versioning` support for SemVer
v2.0.0. (<https://github.com/NuGet/NuGetGallery/issues/3673>)

Filter by default (search service)
----------------------------------

As the search service can only start collecting the `semVerLevel` data once it is
available in the gallery database, this is the logical next step.

First, a **new index will need to be built and deployed** using an updated
version of the **Sql2Lucene** job. Then, an updated **catalog2lucene** job must
be deployed.

The updated search service pointing to the new index must be deployed as well.
The search service is a stand-alone component and can be deployed as soon as the
work is finished, and it should continue to work as it does today.

Unlock push/upload of SemVer v2.0.0 package versions
----------------------------------------------------

Once all required default filtering is in place, we can start accepting actual
SemVer v2.0.0 packages on nuget.org.

Unlock SemVer v2.0.0 package consumption
----------------------------------------

This step may be combined with the previous step.

**Suggestion:** Perhaps we can introduce a configuration setting that may act as
a *feature toggle*, effectively allowing us to disable the logical paths in case
of issues, and allowing us to separate deployment date from go-live date.
